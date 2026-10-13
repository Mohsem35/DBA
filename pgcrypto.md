# Column-Level Encryption in PostgreSQL with pgcrypto

How to encrypt sensitive columns (names, national ID, PIN, date of birth, addresses, photos) with the `pgcrypto` extension, keep them searchable with a blind index, migrate an existing plain-text table, and manage keys safely.

The running example is a voter-details table (`nda.voter_details`), but the method applies to any table holding personal data.

> **Version:** written for **PostgreSQL 18**. Everything here also works on PostgreSQL 13–17 unless noted. The SQL in sections 3, 6, 7 and 11 was tested on PostgreSQL 16 with psql 16.

---

## Table of contents

1. [What this protects against (and what it does not)](#1-what-this-protects-against-and-what-it-does-not)
2. [Design decisions](#2-design-decisions)
3. [Install and verify pgcrypto](#3-install-and-verify-pgcrypto)
4. [Handle the keys safely](#4-handle-the-keys-safely)
5. [Create the encrypted table](#5-create-the-encrypted-table)
6. [Migrate the existing plain-text data](#6-migrate-the-existing-plain-text-data)
7. [Verify the migration](#7-verify-the-migration)
8. [Cut over and remove the plain text](#8-cut-over-and-remove-the-plain-text)
9. [Use the encrypted data](#9-use-the-encrypted-data)
10. [Access control](#10-access-control)
11. [Key rotation](#11-key-rotation)
12. [Performance and storage](#12-performance-and-storage)
13. [Backups, replicas and logs](#13-backups-replicas-and-logs)
14. [pgcrypto function reference](#14-pgcrypto-function-reference)
15. [Common mistakes](#15-common-mistakes)
16. [Version notes](#16-version-notes)
17. [References](#17-references)

---

## 1. What this protects against (and what it does not)

**Protects against:**

- Stolen or leaked database dumps, backups and disk images. The sensitive columns are unreadable without the key.
- Database roles that can read the table but don't have the key.
- Accidental exposure in ad-hoc queries, exports and support screenshots.

**Does not protect against:**

- Anyone who has both database access **and** the key.
- A compromised application server that holds the key.
- Keys leaking into server logs, `psql` history or monitoring tools (see [section 4](#4-handle-the-keys-safely)).
- Data in transit, unless connections use TLS. pgcrypto runs inside the server, so plain text and keys travel between the client and the server.

If the database administrators must not be able to see the data at all, encrypt in the **application** instead and store only ciphertext in PostgreSQL.

---

## 2. Design decisions

| Decision | Why |
|---|---|
| **Encrypted columns are `bytea`** | `pgp_sym_encrypt()` returns `bytea`. If the column is `text`, PostgreSQL silently stores the hex text form (`\xc30d04…`): twice the size, and every decrypt then needs a `::bytea` cast. |
| **`pgp_sym_encrypt` with `cipher-algo=aes256`** | The default cipher is `aes128`. AES-256 is the common requirement for personal data. |
| **Searchable fields get a blind index** (`HMAC-SHA256` with a separate key) | `pgp_sym_encrypt` output is random: the same input produces different ciphertext each time (random salt and IV). You can't search or enforce uniqueness on ciphertext. |
| **HMAC, not plain `digest(..., 'sha256')`** | National IDs, PINs and birth dates have small value ranges. A plain SHA-256 of a birth date can be reversed by hashing all ~36,500 dates in 100 years in under a second. HMAC needs the secret key, so the hashes can't be brute-forced from a dump. |
| **Two separate keys** (encryption key, HMAC key) | Leaking one doesn't compromise the other. |
| **Keys live in a vault, never in SQL files, the database or Git** | Losing the key means losing the data. Leaking the key means losing the protection. |
| **A `key_version` column** | Lets you rotate keys row by row without downtime. |
| **Uniqueness on `national_id_hash`, not `national_id`** | A `UNIQUE` constraint on ciphertext never detects duplicates. |

---

## 3. Install and verify pgcrypto

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
```

- `pgcrypto` is a **trusted** extension (PostgreSQL 13+). A non-superuser with `CREATE` privilege on the database can install it.
- Install it **in each database** that uses it. Extensions are per database.
- On RHEL/Rocky with PGDG packages it comes from `postgresql18-contrib`. On Debian/Ubuntu it's included in the `postgresql-18` package.

### Verify it's installed (not just available)

`pg_available_extensions` lists extensions that **can** be installed. Check the `installed_version` column, or query `pg_extension`:

```sql
-- installed_version is NULL if the extension is available but not installed
SELECT name, default_version, installed_version
FROM pg_available_extensions
WHERE name = 'pgcrypto';

-- returns a row only if it is installed in the current database
SELECT extname, extversion, extnamespace::regnamespace AS schema
FROM pg_extension
WHERE extname = 'pgcrypto';
```

In `psql`: `\dx pgcrypto`.

### Quick test

```sql
SELECT pgp_sym_decrypt(
         pgp_sym_encrypt('hello', 'test-key', 'cipher-algo=aes256'),
         'test-key') AS round_trip;      -- returns: hello
```

---

## 4. Handle the keys safely

Never type the real key into an SQL file or command. If a key has ever appeared in a file, a ticket, a chat or a Git commit, treat it as **compromised and rotate it**.

### Generate strong keys

Use long random keys, not human passwords:

```bash
openssl rand -base64 32     # run twice: one encryption key, one HMAC key
```

Store both in your vault (for example HashiCorp Vault, AWS Secrets Manager or Azure Key Vault) with a version number.

### Pass keys to psql without writing them down

Load the keys from the vault into environment variables just for the session, then read them with `\getenv` (psql 15+):

```bash
# example with HashiCorp Vault; adapt to your vault
export PG_ENC_KEY="$(vault kv get -field=enc_key  secret/nda/pgcrypto)"
export PG_HMAC_KEY="$(vault kv get -field=hmac_key secret/nda/pgcrypto)"
psql -d nda_db
```

```sql
\getenv enc_key  PG_ENC_KEY
\getenv hmac_key PG_HMAC_KEY
-- use them as :'enc_key' and :'hmac_key' (psql quotes them as string literals)
```

`unset PG_ENC_KEY PG_HMAC_KEY` when you're done.

### Keep keys out of the server log

psql substitutes the variables **before** sending the query, so the server receives the key as a literal. Statement or error logging would write it to the log. For the migration session (needs superuser):

```sql
SET log_statement = 'none';
SET log_min_duration_statement = -1;
SET log_min_error_statement = 'panic';   -- a failing statement would otherwise be logged with the key
```

`pg_stat_statements` replaces constants with `$1`, `$2`, …, so it doesn't store the key. Server logs and `auto_explain` can.

In applications, always pass keys as **bind parameters** (`$1`), never by concatenating them into SQL strings.

---

## 5. Create the encrypted table

Starting point: the existing plain-text table, renamed (or restored from a dump) as `nda.voter_details_plain`:

```sql
ALTER TABLE nda.voter_details RENAME TO voter_details_plain;
-- or restore the dump into nda.voter_details_plain
```

The target table: every sensitive column is `bytea`, plus HMAC blind-index columns for lookups:

```sql
CREATE TABLE nda.voter_details (
    id                     varchar(128) NOT NULL,
    name_bn                bytea NOT NULL,
    name_en                bytea,
    date_of_birth          bytea NOT NULL,
    father_name_bn         bytea,
    father_name_en         bytea,
    mother_name_bn         bytea,
    mother_name_en         bytea,
    spouse_name_bn         bytea,
    spouse_name_en         bytea,
    occupation             bytea,
    blood_group            bytea,
    national_id            bytea NOT NULL,
    pin                    bytea,
    permanent_address      bytea,
    present_address        bytea,
    permanent_address_str  bytea,
    present_address_str    bytea,
    photo                  bytea,
    fetched_on             timestamp NOT NULL,
    created_on             timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,

    -- blind indexes: HMAC-SHA256 with a separate key, used for exact-match search
    national_id_hash       bytea NOT NULL,
    pin_hash               bytea,
    dob_hash               bytea,

    key_version            smallint NOT NULL DEFAULT 1,

    CONSTRAINT pk_voter_details PRIMARY KEY (id),
    CONSTRAINT uk_voter_details_national_id_hash UNIQUE (national_id_hash)
);

CREATE INDEX idx_voter_details_pin_hash ON nda.voter_details (pin_hash);
CREATE INDEX idx_voter_details_dob_hash ON nda.voter_details (dob_hash);
```

Notes:

- `fetched_on` and `created_on` stay plain so you can still filter and sort by date. Encrypt them too if the dates themselves are sensitive.
- `id` stays plain because it's the primary key. Make sure it isn't derived from the national ID. If it is, it leaks the value you're encrypting.
- Avoid timestamps without a time zone for new designs (`timestamptz` is usually better). The original types are kept here for compatibility.

---

## 6. Migrate the existing plain-text data

### Normalize before hashing

The blind index only matches if the value is hashed in **exactly** the same form at insert time and at search time. Decide the canonical form once and use it everywhere:

- National ID / PIN: `btrim()`, remove spaces and dashes.
- Date of birth: one format, for example `YYYY-MM-DD`.

### Single-statement migration (small or medium tables)

Run in psql after loading the keys ([section 4](#4-handle-the-keys-safely)):

```sql
BEGIN;

INSERT INTO nda.voter_details (
    id, name_bn, name_en, date_of_birth,
    father_name_bn, father_name_en, mother_name_bn, mother_name_en,
    spouse_name_bn, spouse_name_en, occupation, blood_group,
    national_id, pin,
    permanent_address, present_address, permanent_address_str, present_address_str,
    photo, fetched_on, created_on,
    national_id_hash, pin_hash, dob_hash, key_version
)
SELECT
    src.id,
    pgp_sym_encrypt(src.name_bn,                 :'enc_key', 'cipher-algo=aes256'),
    pgp_sym_encrypt(src.name_en,                 :'enc_key', 'cipher-algo=aes256'),
    pgp_sym_encrypt(src.date_of_birth,           :'enc_key', 'cipher-algo=aes256'),
    pgp_sym_encrypt(src.father_name_bn,          :'enc_key', 'cipher-algo=aes256'),
    pgp_sym_encrypt(src.father_name_en,          :'enc_key', 'cipher-algo=aes256'),
    pgp_sym_encrypt(src.mother_name_bn,          :'enc_key', 'cipher-algo=aes256'),
    pgp_sym_encrypt(src.mother_name_en,          :'enc_key', 'cipher-algo=aes256'),
    pgp_sym_encrypt(src.spouse_name_bn,          :'enc_key', 'cipher-algo=aes256'),
    pgp_sym_encrypt(src.spouse_name_en,          :'enc_key', 'cipher-algo=aes256'),
    pgp_sym_encrypt(src.occupation,              :'enc_key', 'cipher-algo=aes256'),
    pgp_sym_encrypt(src.blood_group,             :'enc_key', 'cipher-algo=aes256'),
    pgp_sym_encrypt(src.national_id,             :'enc_key', 'cipher-algo=aes256'),
    pgp_sym_encrypt(src.pin,                     :'enc_key', 'cipher-algo=aes256'),
    pgp_sym_encrypt(src.permanent_address::text, :'enc_key', 'cipher-algo=aes256'),
    pgp_sym_encrypt(src.present_address::text,   :'enc_key', 'cipher-algo=aes256'),
    pgp_sym_encrypt(src.permanent_address_str,   :'enc_key', 'cipher-algo=aes256'),
    pgp_sym_encrypt(src.present_address_str,     :'enc_key', 'cipher-algo=aes256'),
    pgp_sym_encrypt(src.photo,                   :'enc_key', 'cipher-algo=aes256'),
    src.fetched_on,
    src.created_on,
    hmac(btrim(src.national_id),   :'hmac_key', 'sha256'),
    hmac(btrim(src.pin),           :'hmac_key', 'sha256'),
    hmac(btrim(src.date_of_birth), :'hmac_key', 'sha256'),
    1
FROM nda.voter_details_plain AS src;

COMMIT;
```

- **Source and target are different tables.** Reading from and inserting into the same `nda.voter_details` would only duplicate rows and fail on the primary key.
- `::text` casts columns such as `json`/`jsonb` addresses to text before encryption. Decrypting returns text, which you cast back (`::jsonb`) when reading.
- `pgp_sym_encrypt(NULL, ...)` and `hmac(NULL, ...)` return `NULL`, so optional columns stay `NULL`.
- If `photo` is binary (`bytea`) in the source, use `pgp_sym_encrypt_bytea(src.photo, ...)` instead.

### Batched migration (large tables)

Each `pgp_sym_encrypt` call runs a key-derivation step, so encrypting millions of rows × 19 columns takes time. Migrate in batches so each transaction stays short and progress survives interruptions. Run this statement repeatedly until it reports `INSERT 0 0`:

```sql
INSERT INTO nda.voter_details ( /* same column list as above */ )
SELECT /* same expressions as above */
FROM nda.voter_details_plain AS src
WHERE NOT EXISTS (SELECT 1 FROM nda.voter_details t WHERE t.id = src.id)
ORDER BY src.id
LIMIT 50000;
```

Time one batch first and estimate the total before starting on production.

---

## 7. Verify the migration

```sql
-- 1. Row counts match
SELECT (SELECT count(*) FROM nda.voter_details_plain) AS plain_rows,
       (SELECT count(*) FROM nda.voter_details)       AS encrypted_rows;

-- 2. Decrypted values match the source (expect 0)
SELECT count(*) AS mismatches
FROM nda.voter_details_plain p
JOIN nda.voter_details e USING (id)
WHERE pgp_sym_decrypt(e.national_id,   :'enc_key') IS DISTINCT FROM p.national_id
   OR pgp_sym_decrypt(e.name_bn,       :'enc_key') IS DISTINCT FROM p.name_bn
   OR pgp_sym_decrypt(e.date_of_birth, :'enc_key') IS DISTINCT FROM p.date_of_birth;

-- 3. Blind index works
SELECT count(*) AS found
FROM nda.voter_details
WHERE national_id_hash = hmac(btrim(:'test_nid'), :'hmac_key', 'sha256');
```

For very large tables, run check 2 on a random sample (`TABLESAMPLE SYSTEM (1)`) instead of every row.

---

## 8. Cut over and remove the plain text

1. Deploy the application changes ([section 9](#9-use-the-encrypted-data)) and test them against the new table.
2. Stop writes to the plain table during the switch.
3. Drop the plain-text table once you've verified the migration and the business owner has signed off:

   ```sql
   DROP TABLE nda.voter_details_plain;
   ```

4. Plain text still exists in places `DROP TABLE` doesn't reach:
   - **Old backups and dumps:** expire or securely destroy them according to your retention policy.
   - **WAL archives and replicas:** archived WAL keeps the old data until it ages out of retention.
   - **Exported CSVs, test copies and developer laptops:** find and delete them.

---

## 9. Use the encrypted data

### Read (decrypt) a record

```sql
SELECT id,
       pgp_sym_decrypt(name_en,       $1) AS name_en,
       pgp_sym_decrypt(national_id,   $1) AS national_id,
       pgp_sym_decrypt(date_of_birth, $1) AS date_of_birth,
       pgp_sym_decrypt(present_address, $1)::jsonb AS present_address
FROM nda.voter_details
WHERE id = $2;
```

`$1` is the encryption key, passed by the application as a bind parameter.

### Search by national ID (uses the index)

```sql
SELECT id, pgp_sym_decrypt(name_en, $1) AS name_en
FROM nda.voter_details
WHERE national_id_hash = hmac(btrim($2), $3, 'sha256');   -- $2 = NID, $3 = HMAC key
```

**Never** search with `WHERE pgp_sym_decrypt(national_id, $1) = $2`. That decrypts every row in the table on every query, can't use an index, and is extremely slow.

### Insert a new record

```sql
INSERT INTO nda.voter_details (
    id, name_bn, date_of_birth, national_id, fetched_on,
    national_id_hash, dob_hash, key_version)
VALUES (
    $1,
    pgp_sym_encrypt($2, $10, 'cipher-algo=aes256'),
    pgp_sym_encrypt($3, $10, 'cipher-algo=aes256'),
    pgp_sym_encrypt($4, $10, 'cipher-algo=aes256'),
    $5,
    hmac(btrim($4), $11, 'sha256'),
    hmac(btrim($3), $11, 'sha256'),
    1);
-- $10 = encryption key, $11 = HMAC key
```

### What you can no longer do in SQL

- Range queries, sorting, `LIKE` and partial matches on encrypted columns.
- Aggregations on encrypted values (counting by blood group, for example) without decrypting every row.

If a feature needs one of these, keep that particular field unencrypted (if allowed) or do the work in the application after decrypting.

---

## 10. Access control

```sql
-- Nobody gets access by default
REVOKE ALL ON nda.voter_details FROM PUBLIC;

-- Application role: read and write ciphertext; still needs the key to decrypt
GRANT SELECT, INSERT, UPDATE ON nda.voter_details TO nda_app;

-- Reporting role: only non-sensitive columns
GRANT SELECT (id, fetched_on, created_on) ON nda.voter_details TO nda_reporting;
```

- Only the application (and an emergency break-glass procedure) should be able to get the keys from the vault.
- Enable `pgaudit` (or equivalent) to record who queries the table.
- Require TLS for all connections (`hostssl` in `pg_hba.conf`), because keys and decrypted values cross the network.

---

## 11. Key rotation

The `key_version` column records which key encrypted each row. To rotate:

1. Create a new key version in the vault (for example version 2). Keep version 1 available until rotation finishes.
2. Re-encrypt in batches:

   ```sql
   UPDATE nda.voter_details
   SET name_bn     = pgp_sym_encrypt(pgp_sym_decrypt(name_bn,     :'old_key'), :'new_key', 'cipher-algo=aes256'),
       national_id = pgp_sym_encrypt(pgp_sym_decrypt(national_id, :'old_key'), :'new_key', 'cipher-algo=aes256'),
       -- ... every encrypted column ...
       key_version = 2
   WHERE id IN (
       SELECT id FROM nda.voter_details
       WHERE key_version = 1
       ORDER BY id
       LIMIT 10000
   );
   ```

3. Repeat until no rows have `key_version = 1`. Meanwhile the application decrypts each row with the key matching its `key_version`.
4. Retire the old key.
5. If you rotate the **HMAC** key, recompute every blind-index column at the same time. Old hashes stop matching.

Each rotation rewrites every row, which creates dead tuples. Run `VACUUM (ANALYZE) nda.voter_details;` afterwards and watch autovacuum.

---

## 12. Performance and storage

- **CPU:** each `pgp_sym_encrypt` call derives a key from the password (iterated S2K, 65,536–253,952 iterations by default). Encrypting 19 columns per row makes bulk migration CPU-bound. Measure on a sample first.
- **S2K cost:** with a long random key from a vault (not a human password), you can lower the derivation cost, for example `'cipher-algo=aes256, s2k-count=1024'`. The iterations exist to slow down guessing of weak passwords and add little for a 32-byte random key. This is a trade-off: get your security team's agreement before changing it.
- **Storage:** each encrypted value adds PGP packet overhead (tens of bytes), and `bytea` ciphertext can't be compressed effectively. Expect the table to be noticeably larger than the plain version, especially with many short columns. The `photo` column dominates if it holds images.
- **Indexes:** only the blind-index columns and plain columns can be indexed usefully.
- **Compression:** `compress-algo=1` or `2` compresses before encrypting. It helps for large text such as base64 photos, but compressing before encrypting can leak information about the content through the ciphertext length. Leave it off unless you have a reason.

---

## 13. Backups, replicas and logs

- Backups (`pg_dump`, pgBackRest, Barman) contain only ciphertext for the encrypted columns. **Back up the keys separately** in the vault. A restore without the keys is useless.
- Physical replicas receive the same ciphertext. They need no extra configuration.
- Review `log_statement`, `log_min_duration_statement`, `log_min_error_statement` and `auto_explain` settings so keys passed as literals never reach the logs (see [section 4](#4-handle-the-keys-safely)).

---

## 14. pgcrypto function reference

| Function | Returns | Use |
|---|---|---|
| `pgp_sym_encrypt(data text, key text [, options text])` | `bytea` | Encrypt text |
| `pgp_sym_decrypt(msg bytea, key text [, options text])` | `text` | Decrypt to text |
| `pgp_sym_encrypt_bytea(data bytea, key text [, options])` | `bytea` | Encrypt binary data |
| `pgp_sym_decrypt_bytea(msg bytea, key text [, options])` | `bytea` | Decrypt binary data |
| `hmac(data text, key text, type text)` | `bytea` | Keyed hash, for blind indexes |
| `digest(data text, type text)` | `bytea` | Unkeyed hash (checksums; **not** for personal data) |
| `crypt(password text, salt text)` | `text` | Password hashing (logins) |
| `gen_salt(type text [, iter_count int])` | `text` | Salt for `crypt()`: `bf`, `md5`, `sha256crypt`, `sha512crypt`, `xdes`, `des` |
| `gen_random_bytes(count int)` | `bytea` | Random bytes (max 1024 per call) |

Useful PGP options (comma-separated third argument):

| Option | Values | Default |
|---|---|---|
| `cipher-algo` | `bf`, `aes128`, `aes192`, `aes256`, `3des`, `cast5` | `aes128` |
| `compress-algo` | `0` none, `1` ZIP, `2` ZLIB | `0` |
| `s2k-mode` | `0` (no salt; dangerous), `1`, `3` | `3` |
| `s2k-count` | `1024`–`65011712` | random 65536–253952 |
| `s2k-digest-algo` | `md5`, `sha1` | `sha1` |

Hex output for display: `encode(hmac(...), 'hex')`.

For user **passwords** (logins), don't use `pgp_sym_encrypt` or `digest`. Use `crypt()` with a slow salt: `crypt('secret', gen_salt('bf', 10))`, or better, hash passwords in the application with a dedicated library.

---

## 15. Common mistakes

| Mistake | Correct |
|---|---|
| Checking installation with `pg_available_extensions` alone | That lists **available** extensions. Check `installed_version`, `pg_extension`, or `\dx`. |
| Encrypted columns declared as `text` | It doesn't error: the `bytea` is silently stored as hex text, doubling the size and forcing `::bytea` casts to decrypt. Declare the columns `bytea`. |
| Hash columns declared as `text` while storing `digest()` or `hmac()` output | Both return `bytea`, which gets stored as `\x…` text. Store `bytea`, or `encode(..., 'hex')` deliberately into `text`, and compare in the same form when searching. |
| `INSERT INTO t SELECT ... FROM t` (same table as source and target) | Rename the original to `*_plain` and migrate from there into a new table. |
| `UNIQUE` constraint on an encrypted column | Ciphertext is random each time. Enforce uniqueness on the HMAC column. |
| Plain `digest(value, 'sha256')` for national ID, PIN or date of birth | Easily brute-forced for small value ranges. Use `hmac(value, hmac_key, 'sha256')`. |
| Hashing without normalizing | `' 123'` and `'123'` produce different hashes. Normalize (trim, one format) at insert and at search time. |
| Hard-coding the key in SQL scripts or documentation | Load it from the vault at runtime (`\getenv` or bind parameters). Rotate any key that was ever written down. |
| Default cipher (`aes128`) without deciding | Pass `'cipher-algo=aes256'` if your policy requires AES-256. |
| Searching with `WHERE pgp_sym_decrypt(col, key) = value` | Full table scan, decrypting every row. Search the blind-index column instead. |
| Dropping the plain table and assuming the data is gone | Old backups, WAL archives, replicas and exports still contain it. |
| Losing the key | The data can't be recovered. Back up keys in the vault with access controls. |

---

## 16. Version notes

- **PostgreSQL 13+:** `pgcrypto` is a trusted extension (non-superusers with `CREATE` privilege can install it).
- **psql 15+:** `\getenv` reads environment variables into psql variables.
- **PostgreSQL 18:** `crypt()`/`gen_salt()` support `sha256crypt` and `sha512crypt`, and the `pgcrypto.builtin_crypto_enabled` setting can disable the built-in (non-OpenSSL) `crypt()`/`gen_salt()` code, for example in FIPS mode. `pgp_sym_encrypt()` isn't affected.
- **OpenSSL 3:** older ciphers (`bf`, `cast5`, DES) need the OpenSSL legacy provider. AES works without it.

---

## 17. References

- [pgcrypto documentation (PostgreSQL 18)](https://www.postgresql.org/docs/current/pgcrypto.html)
- [psql `\getenv` and variables](https://www.postgresql.org/docs/current/app-psql.html)
- [Error reporting and logging settings](https://www.postgresql.org/docs/current/runtime-config-logging.html)
- [Privileges and column-level GRANT](https://www.postgresql.org/docs/current/ddl-priv.html)
