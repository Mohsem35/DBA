
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