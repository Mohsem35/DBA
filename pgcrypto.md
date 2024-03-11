_**1st step**: Install `pgcrypto` extension to the specific database_

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;
```

Now confirm if this extension is installed or not

```sql
select * from pg_available_extensions where "name"='pgcrypto';
```

_**2nd Step**: Now create the table on whose you want to apply encryption using DDL_

```sql
CREATE TABLE nda.voter_details (
	id varchar(128) NOT NULL,
	name_bn text NOT NULL,
	name_en text NULL,
	date_of_birth text NOT NULL,
	father_name_bn text NULL,
	father_name_en text NULL,
	mother_name_bn text NULL,
	mother_name_en text NULL,
	spouse_name_bn text NULL,
	spouse_name_en text NULL,
	occupation text NULL,
	blood_group text NULL,
	national_id text NOT NULL,
	pin text NULL,
	permanent_address text NULL,
	present_address text NULL,
	permanent_address_str text NULL,
	present_address_str text NULL,
	photo text NULL,
	fetched_on timestamp NOT NULL,
	created_on timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
	national_id_hash text NULL,
	pin_hash text NULL,
	dob_hash text NULL,
	CONSTRAINT pk_id_voter_detail PRIMARY KEY (id),
	CONSTRAINT uk_national_id_voter_detail UNIQUE (national_id)
);
```
_**3rd Step**: Now populate the `nda.voter_details` table with dump_

_**4th Step**: Migrate existing NDA database table into newly created table_

```sql
INSERT INTO nda.voter_details (
  id,
  name_bn,
  name_en,
  date_of_birth,
  father_name_bn,
  father_name_en,
  mother_name_bn,
  mother_name_en,
  spouse_name_bn,
  spouse_name_en,
  occupation,
  blood_group,
  national_id,
  pin,
  permanent_address,
  present_address,
  permanent_address_str,
  present_address_str,
  photo,
  fetched_on,
  created_on,
  national_id_hash,
  pin_hash,
  dob_hash
)
SELECT 
  src.id,
  pgp_sym_encrypt(src.name_bn,'p#@yuSuF'),
  pgp_sym_encrypt(src.name_en,'p#@yuSuF'),
  pgp_sym_encrypt(src.date_of_birth,'p#@yuSuF'),
  pgp_sym_encrypt(src.father_name_bn,'p#@yuSuF'),
  pgp_sym_encrypt(src.father_name_en,'p#@yuSuF'),
  pgp_sym_encrypt(src.mother_name_bn,'p#@yuSuF'),
  pgp_sym_encrypt(src.mother_name_en,'p#@yuSuF'),
  pgp_sym_encrypt(src.spouse_name_bn,'p#@yuSuF'),
  pgp_sym_encrypt(src.spouse_name_en,'p#@yuSuF'),
  pgp_sym_encrypt(src.occupation,'p#@yuSuF'),
  pgp_sym_encrypt(src.blood_group,'p#@yuSuF'),
  pgp_sym_encrypt(src.national_id,'p#@yuSuF'),
  pgp_sym_encrypt(src.pin,'p#@yuSuF'),
  pgp_sym_encrypt(src.permanent_address::text,'p#@yuSuF'),
  pgp_sym_encrypt(src.present_address::text,'p#@yuSuF'),
  pgp_sym_encrypt(src.permanent_address_str,'p#@yuSuF'),
  pgp_sym_encrypt(src.present_address_str,'p#@yuSuF'),
  pgp_sym_encrypt(src.photo,'p#@yuSuF'),
  src.fetched_on,
  src.created_on,
  digest(src.national_id, 'sha256'),
  digest(src.pin, 'sha256'),
  digest(src.date_of_birth, 'sha256')
FROM nda.voter_details AS src;
```
- `nda.voter_details` টেবিল হল এখানে src
- বাকি সব column encrypted হবে, যার encryption password হচ্ছে **`p#@yuSuF`**
- কিন্তু শেষ ৩ টা column হবে hash format কারণ `sha256` format use করা হইছে
  
In this query replace **`p#@yuSuF`** with secret key which is stored in the vault. এই secret key ছাড়া data decrypt করা সম্ভব হবে না 


