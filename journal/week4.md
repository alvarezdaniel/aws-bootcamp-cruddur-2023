# Week 4 — Postgres and RDS

## Week 4 instructors

- Andrew Brown [@andrewbrown](https://twitter.com/andrewbrown)
- Shala Warner [@GiftedLane](https://twitter.com/GiftedLane)

## Class Summary

- Have a lecture about data modelling in (3rd Normal Form) 3NF for SQL
- Launch Postgres locally via a container
- Seed our Postgres Database table with data
- Write a Postgres adapter
- Write a DDL (for creating schema)
- Write an SQL read query
- Write an SQL write query
- Provision an RDS Postgres instance
- Configure VPC Security Groups
- Configure local backend application to use production connection URL
- Add a caching layer using Momento Serverless Cache
- Propagate metrics from DDB to an RDS metrics table 

## Todo Checklist

### Launch Postgres locally via a container

First of all we need to ensure that postgres container is configured in `docker-compose.yml` file. I've disabled before, so in this case I'm enabling it again.

```yml
version: "3.8"
services:
  db:
    image: postgres:13-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=password
    ports:
      - '5432:5432'
    volumes: 
      - db:/var/lib/postgresql/data      
```

> The image we are using is postgres:13-alpine: https://hub.docker.com/_/postgres

Also, we need postgres client installed in workspace, so I've checked it in `.gitpod.yml` file

```yml
tasks:

  - name: postgres
    init: |
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev      
```

> Installed debian packages:

> front-end programs for PostgreSQL 13: https://packages.debian.org/bullseye/postgresql-client-13

> header files for libpq5 (PostgreSQL library): https://packages.debian.org/bullseye/libpq-dev

For testing client and postgres server installed locally, we can use psql cli

```sh
psql -Upostgres --host localhost
```

> Remember to use the host flag to specific localhost (only in docker)

Common PSQL commands:

```sql
\x on -- expanded display when looking at data
\q -- Quit PSQL
\l -- List all databases
\c database_name -- Connect to a specific database
\dt -- List all tables in the current database
\d table_name -- Describe a specific table
\du -- List all users and their roles
\dn -- List all schemas in the current database
CREATE DATABASE database_name; -- Create a new database
DROP DATABASE database_name; -- Delete a database
CREATE TABLE table_name (column1 datatype1, column2 datatype2, ...); -- Create a new table
DROP TABLE table_name; -- Delete a table
SELECT column1, column2, ... FROM table_name WHERE condition; -- Select data from a table
INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...); -- Insert data into a table
UPDATE table_name SET column1 = value1, column2 = value2, ... WHERE condition; -- Update data in a table
DELETE FROM table_name WHERE condition; -- Delete data from a table
```

### Provision an RDS Postgres instance

This step is done before everything else because we will be needing a cloud Postgres RDS instance in AWS for connecting later.

We will be creating the RDS instance using CLI, using this script

```sh
aws rds create-db-instance \
  --db-instance-identifier cruddur-db-instance \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --engine-version  14.6 \
  --master-username $MASTER_USERNAME \
  --master-user-password $MASTER_USER_PASSWORD \
  --allocated-storage 20 \
  --availability-zone ca-central-1a \
  --backup-retention-period 0 \
  --port 5432 \
  --no-multi-az \
  --db-name cruddur \
  --storage-type gp2 \
  --publicly-accessible \
  --storage-encrypted \
  --enable-performance-insights \
  --performance-insights-retention-period 7 \
  --no-deletion-protection
```

> AWS CLI command reference for create db instance: https://docs.aws.amazon.com/cli/latest/reference/rds/create-db-instance.html

> This will take about 10-15 mins

> This is a lot easier than using the console

> Andrew showed all the options to create a RDS instance using the console

Some links

- https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_GettingStarted.html
- https://aws.amazon.com/rds
- https://aws.amazon.com/documentation/rds
- https://aws.amazon.com/articles/Amazon-RDS
- https://aws.amazon.com/rds/pricing
- https://forums.aws.amazon.com/

We should set the corresponding environment variables for not exposing master-username and master-user-password

```sh
export MASTER_USERNAME="XXXX"
gp env MASTER_USERNAME="XXXX"

export MASTER_USER_PASSWORD="XXXX"
gp env MASTER_USER_PASSWORD="XXXX"
```

After setting the variables we can run the script to create the postgres RDS instance

```sh
aws rds create-db-instance \
>   --db-instance-identifier cruddur-db-instance \
>   --db-instance-class db.t3.micro \
>   --engine postgres \
>   --engine-version  14.6 \
>   --master-username $MASTER_USERNAME \
>   --master-user-password $MASTER_USER_PASSWORD \
>   --allocated-storage 20 \
>   --availability-zone ca-central-1a \
>   --backup-retention-period 0 \
>   --port 5432 \
>   --no-multi-az \
>   --db-name cruddur \
>   --storage-type gp2 \
>   --publicly-accessible \
>   --storage-encrypted \
>   --enable-performance-insights \
>   --performance-insights-retention-period 7 \
>   --no-deletion-protection
```

Result

```json
{
    "DBInstance": {
        "DBInstanceIdentifier": "cruddur-db-instance",
        "DBInstanceClass": "db.t3.micro",
        "Engine": "postgres",
        "DBInstanceStatus": "creating",
        "MasterUsername": "****",
        "DBName": "cruddur",
        "AllocatedStorage": 20,
        "PreferredBackupWindow": "07:26-07:56",
        "BackupRetentionPeriod": 0,
        "DBSecurityGroups": [],
        "VpcSecurityGroups": [
            {
                "VpcSecurityGroupId": "sg-0a539dae17942e3b3",
                "Status": "active"
            }
        ],
        "DBParameterGroups": [
            {
                "DBParameterGroupName": "default.postgres14",
                "ParameterApplyStatus": "in-sync"
            }
        ],
        "AvailabilityZone": "ca-central-1a",
        "DBSubnetGroup": {
            "DBSubnetGroupName": "default",
            "DBSubnetGroupDescription": "default",
            "VpcId": "vpc-0a387705bb7ef150c",
            "SubnetGroupStatus": "Complete",
            "Subnets": [
                {
                    "SubnetIdentifier": "subnet-02427f9148364f1d6",
                    "SubnetAvailabilityZone": {
                        "Name": "ca-central-1b"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-0003de2399abffe43",
                    "SubnetAvailabilityZone": {
                        "Name": "ca-central-1d"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                },
                {
                    "SubnetIdentifier": "subnet-0c960b9a5b03f1803",
                    "SubnetAvailabilityZone": {
                        "Name": "ca-central-1a"
                    },
                    "SubnetOutpost": {},
                    "SubnetStatus": "Active"
                }
            ]
        },
        "PreferredMaintenanceWindow": "mon:08:16-mon:08:46",
        "PendingModifiedValues": {
            "MasterUserPassword": "****"
        },
        "MultiAZ": false,
        "EngineVersion": "14.6",
        "AutoMinorVersionUpgrade": true,
        "ReadReplicaDBInstanceIdentifiers": [],
        "LicenseModel": "postgresql-license",
        "OptionGroupMemberships": [
            {
                "OptionGroupName": "default:postgres-14",
                "Status": "in-sync"
            }
        ],
        "PubliclyAccessible": true,
        "StorageType": "gp2",
        "DbInstancePort": 0,
        "StorageEncrypted": true,
        "KmsKeyId": "arn:aws:kms:ca-central-1:052985194353:key/45903320-bfda-4a41-9046-d05a49db4ab7",
        "DbiResourceId": "db-MKM2SG6VRXGS4YTYYSCFCN2Q3A",
        "CACertificateIdentifier": "rds-ca-2019",
        "DomainMemberships": [],
        "CopyTagsToSnapshot": false,
        "MonitoringInterval": 0,
        "DBInstanceArn": "arn:aws:rds:ca-central-1:052985194353:db:cruddur-db-instance",
        "IAMDatabaseAuthenticationEnabled": false,
        "PerformanceInsightsEnabled": true,
        "PerformanceInsightsKMSKeyId": "arn:aws:kms:ca-central-1:052985194353:key/45903320-bfda-4a41-9046-d05a49db4ab7",
        "PerformanceInsightsRetentionPeriod": 7,
        "DeletionProtection": false,
        "AssociatedRoles": [],
        "TagList": [],
        "CustomerOwnedIpEnabled": false,
        "BackupTarget": "region",
        "NetworkType": "IPV4",
        "StorageThroughput": 0,
        "CertificateDetails": {
            "CAIdentifier": "rds-ca-2019"
        }
    }
}
```

After executing the script, the database appears as `Creating`

![](assets/week-4/01-db-creating.png)

After some minutes (in my case the process took about 5 minutes), database instance should be stopped temporally, in order not to consume credits, because it is a VM in AWS

![](assets/week-4/02-db-stop-temp.png)

![](assets/week-4/03-db-stop-temp.png)

In this case, the database will be first in `Stopping` status, and then after some minutes, when the process completed, in `Stopped temporarily` status

![](assets/week-4/04-db-stopped.png)





### Seed our Postgres Database table with data


First of all we've been trying manually executing some commands to create and drop a database



For creating the database, tables and seed them with initial data we will be writting several scripts, that will be located in a bin folder inside backend-flask. These scripts will be using some sql files that will be located in a sql folder in the same root location

`/backend-flask/bin`
`/backend-flask/sql`





