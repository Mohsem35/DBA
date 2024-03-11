
## AWS RDS

**Why should you run DBS on EC2?**
- _Access to the DB instance OS._
- _Advanced DB Option tuning (DBROOT)._
- _Vendor demands._
- DB or DB version that AWS doesn't provide.
- You might need a _specific version of an OS and DB that AWS doesn't provide._


**Why shouldn't you run DBs on EC2?**
- Admin overhead.
- Backup and DR.
- _EC2 is running in a single AZ._
- Will miss out on features from AWS DB products.
- Skills and setup time to monitor.
- Performance will be slower than AWS options.

#### RDS Database Instance

Some key features and characteristics of Amazon Web Services (AWS) Relational Database Service (RDS).

1. **Database Connection with CNAME**: You can configure your RDS database to be accessed using a _CNAME (Canonical Name)_ for ease of connection and management.

2. **Standard Database Engines**: AWS RDS supports various standard database engines like MySQL, PostgreSQL, MariaDB, Oracle, SQL Server, and Amazon Aurora.

3. **Instance Optimization Options**: RDS offers different instance types optimized for various workloads:
        
        
    - **`db.m5`**: General-purpose instances for balanced compute, memory, and network resources.
    - **`db.r5`**: Memory-optimized instances with higher memory-to-CPU ratio, suitable for memory-intensive workloads.
    - **`db.t3`**: Burstable performance instances, ideal for workloads with intermittent usage patterns.

4. **Instance Provisioning and Storage**: When you provision an RDS instance, you allocate dedicated storage for that instance. _This storage is Amazon Elastic Block Store (EBS) located in the same Availability Zone (AZ) as the RDS instance_. However, it's important to note that while the storage is in the same AZ, RDS itself can be configured for multi-AZ deployments for high availability.

5. **Storage Types**: You can choose between SSD (Solid State Drive) and magnetic storage options for your allocated EBS volumes. SSD offers better performance and is recommended for most workloads, while magnetic storage might be suitable for less performance-critical applications where cost savings are a priority.

6. **Billing Model**: _AWS bills RDS instances based on hourly usage_. You're charged for the compute resources (instance type) provisioned and the storage allocated to the instance. The hourly rate depends on the chosen instance type.

Overall, these features provide flexibility and scalability for managing relational databases on AWS RDS, catering to various performance and cost requirements.



#### How to initialize RDS instance

_1st step: Create subnet group for RDS_

Amazon RDS -> click on `Subnet groups` form left side bar -> click on `Create DB subnet group` -> give specifice `Name`, `Description`, `VPC` ->  select `Availability Zones` according to the following pictures.


<img width="750" alt="Screenshot 2024-03-11 at 3 17 55 PM" src="https://github.com/Mohsem35/DBA/assets/58659448/f1e03c4d-29ae-4bfe-b48b-e560d1adffd3">

<img width="750" alt="Screenshot 2024-03-11 at 3 18 29 PM" src="https://github.com/Mohsem35/DBA/assets/58659448/38101f74-5892-4d61-a6fb-9f14366cd6d0">

2nd Step: Create database

#### Migrating DB(MySQL) from EC2 to RDS

1. Get the dump of your existing DB on EC2.
2. Connect to your RDS DB Instance.
3. Migrate the DB Dump that you have taken in Step 1 to RDS 
4. Verify if the data is available.

```shell
# acces to mysql database(ec2)
mysql -u root --password="${DBRootPassword}"
```
```sql
SHOW Databases;
USE ec2db;
exit
```

```shell
# dump from ec2
mysqldump -u root -p <ec2-db-name> > dump.sql

# restore to rds
# for restoration, run the following command in ec2 instance
mysql -h <replace-rds-end-point-here> -P 3306 -u rdsuser -p <rds-db-name> < dump.sql

# access to rds db(rds)
mysql -h <replace-rds-end-point-here> -P 3306 -u rdsuser -p
```
```sql
USE rdsdb
SELECT * FROM table1;
```

#### RDS Multi AZ (High-Availability)

1. **RDS Access via Database CNAME**: In Amazon RDS, access to the database is typically done using a CNAME (Canonical Name), _which points to the primary instance of the database_. This CNAME provides a stable endpoint for connecting to the database.

2. **No Access to Standby Replica**: The standby replica, which is a secondary instance maintained for high availability and failover purposes, cannot be directly accessed via RDS. **Access is only available through the primary instance**, and the _standby replica cannot be used for any purpose other than failover and maintaining high availability_.

    
3. **Standby Replica Not for Extra Capacity**: While the standby replica exists to provide redundancy and failover capabilities, it cannot be utilized to increase database capacity or serve additional workload. It remains in a standby state and is synchronized with the primary instance for failover scenarios.

4. **Synchronous Replication**: Amazon RDS supports synchronous replication between the primary instance and its standby replica in certain configurations. Synchronous replication ensures that data changes made on the primary instance are immediately replicated to the standby replica, ensuring data consistency and minimizing the risk of data loss in case of a failure.

These restrictions and features are designed to ensure data consistency, high availability, and reliability in Amazon RDS environments, particularly in scenarios where synchronous replication and failover capabilities are crucial requirements.



**Points to Remember**

- Multi-AZ feature incurs additional cost and provides standby infrastructure.
- Standby replica inaccessible unless during a failure event.
- Failover ensures high availability, not fault tolerance.
- Failover occurs within the same region.
- Backups are taken from the standby to minimize performance impacts.
- Failover scenarios include AZ outage, primary instance failure, manual failover, instance type change, and software patching


### RDS Backup and Restores

**RPO - Recovery Point Objective**


- Time between the last backup and when the failure occurred.
- Amount of maximum data loss
- Influences technical solution and cost
- Business usually provides an RPO value

**RTO - Recovery Time Objective**

- Time between the DR event and full recovery
- Influenced by process, staff, tech and documentation
