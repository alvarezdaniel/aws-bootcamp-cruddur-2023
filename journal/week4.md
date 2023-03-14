# Week 4 — Postgres and RDS

## Week 4 instructors

- Andrew Brown [@andrewbrown](https://twitter.com/andrewbrown)
- Shala Warner [@GiftedLane](https://twitter.com/GiftedLane)

## Class Summary

- Provision an RDS instance
- Temporarily stop an RDS instance
- Remotely connect to RDS instance
- Programmatically update a security group rule
- Write several bash scripts for database operations
- Operate common SQL commands
- Create a schema SQL file by hand
- Work with UUIDs and PSQL extensions
- Implement a postgres client for python using a connection pool
- Troubleshoot common SQL errors
- Implement a Lambda that runs in a VPC and commits code to RDS
- Work with PSQL json functions to directly return json from the database
- Correctly sanitize parameters passed to SQL to execute

## Todo Checklist

### Provision an RDS instance

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

After some minutes (in my case the process took about 5 minutes)

### Temporarily stop an RDS instance

Database instance should be stopped temporarily, in order not to consume credits, because it is a VM in AWS

![](assets/week-4/02-db-stop-temp.png)

![](assets/week-4/03-db-stop-temp.png)

In this case, the database will be first in `Stopping` status, and then after some minutes, when the process completed, in `Stopped temporarily` status

![](assets/week-4/04-db-stopped.png)

### Remotely connect to RDS instance

Pending

### Programmatically update a security group rule

Pending

### Write several bash scripts for database operations

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

After successfully checking connection to postgres localhost container, we are going to implement some scripts for executing operations against database

We will need variables for determining connection url to our postgres instance. In this case we are going to generate a connection url to localhost and another to our hosted RDS instance. We need to extact host name by checking endpoint in AWS console for the created RDS instance.

> https://stackoverflow.com/questions/3582552/what-is-the-format-for-the-postgresql-connection-string-url

> https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-CONNSTRING

```sh
export CONNECTION_URL="postgresql://postgres:password@localhost:5432/cruddur"
gp env CONNECTION_URL="postgresql://postgres:password@localhost:5432/cruddur"

export PROD_CONNECTION_URL="postgresql://***:***@$cruddur-db-instance.***.ca-central-1.rds.amazonaws.com:5432/cruddur"
gp env PROD_CONNECTION_URL="postgresql://***:***@$cruddur-db-instance.***.ca-central-1.rds.amazonaws.com:5432/cruddur"
```

These scripts will be located in `/backend-flask/bin` folder (also, some of them requires sql files that will be located in `/backend-flask/sql`)

> All these scripts must be made executable by executing:

```sh
chmod u+x bin/*
```

> And for executing them:

```sh
./bin/db-connect
```


`bin/db-connect`: shell script for connecting to DB

```sh
#! /usr/bin/bash

psql $CONNECTION_URL
```

> This script is used for testing database connection. It uses the defined connection url in the environment variable CONNECTION_URL


`bin/db-create`: shell script for creating DB

```sh
#! /usr/bin/bash

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-create"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
psql $NO_DB_CONNECTION_URL -c "create database cruddur;"
#createdb cruddur $NO_DB_CONNECTION_URL
```

> This script creates cruddur database in the instance passed in CONNECTION_URL environment variable

> As this variable contains database name, it must be trimmed out from its value, by using `sed` command (https://askubuntu.com/questions/595269/use-sed-on-a-string-variable-rather-than-a-file)

> psql or createdb AWS cli commands can be used

> Shell script output color is changed by using this code (https://stackoverflow.com/questions/5947742/how-to-change-the-output-color-of-echo-in-linux):

```sh
CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-schema-load"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"
```


`bin/db-drop`: shell script for dropping DB

```sh
#! /usr/bin/bash

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-drop"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
psql $NO_DB_CONNECTION_URL -c "drop database cruddur;"
```


`bin/db-schema-load`: shell script for loading DB schema

```sh
#! /usr/bin/bash

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-schema-load"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

schema_path="$(realpath .)/db/schema.sql"
echo $schema_path

if [ "$1" = "prod" ]; then
  echo "Running in production mode"
  URL=$PROD_CONNECTION_URL
else
  URL=$CONNECTION_URL
fi

psql $URL cruddur < $schema_path
```


`db/schema.sql`: sql schema file

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
DROP TABLE IF EXISTS public.users;
DROP TABLE IF EXISTS public.activities;


CREATE TABLE public.users (
  uuid UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  display_name text,
  handle text,
  cognito_user_id text,
  created_at TIMESTAMP default current_timestamp NOT NULL
);

CREATE TABLE public.activities (
  uuid UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  user_uuid UUID NOT NULL,
  message text NOT NULL,
  replies_count integer DEFAULT 0,
  reposts_count integer DEFAULT 0,
  likes_count integer DEFAULT 0,
  reply_to_activity_uuid integer,
  expires_at TIMESTAMP,
  created_at TIMESTAMP default current_timestamp NOT NULL
);
```


`bin/db-seed`: shell script for loading seed data into DB

```sh
#! /usr/bin/bash

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-seed"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

seed_path="$(realpath .)/db/seed.sql"
echo $seed_path

if [ "$1" = "prod" ]; then
  echo "Running in production mode"
  URL=$PROD_CONNECTION_URL
else
  URL=$CONNECTION_URL
fi

psql $URL cruddur < $seed_path
```

`db/seed.sql`: sql seed file

```sql
-- this file was manually created
INSERT INTO public.users (display_name, handle, cognito_user_id)
VALUES
  ('Andrew Brown', 'andrewbrown' ,'MOCK'),
  ('Andrew Bayko', 'bayko' ,'MOCK');

INSERT INTO public.activities (user_uuid, message, expires_at)
VALUES
  (
    (SELECT uuid from public.users WHERE users.handle = 'andrewbrown' LIMIT 1),
    'This was imported as seed data!',
    current_timestamp + interval '10 day'
  )
```


`bin/db-sessions`: shell script for checking used connections to DB

```sh
#! /usr/bin/bash
CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-sessions"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

if [ "$1" = "prod" ]; then
  echo "Running in production mode"
  URL=$PROD_CONNECTION_URL
else
  URL=$CONNECTION_URL
fi

NO_DB_URL=$(sed 's/\/cruddur//g' <<<"$URL")
psql $NO_DB_URL -c "select pid as process_id, \
       usename as user,  \
       datname as db, \
       client_addr, \
       application_name as app,\
       state \
from pg_stat_activity;"
```

> We could have idle connections left open by our Database Explorer extension, try disconnecting and checking again the sessions 


`bin/db-setup`: shell script for resetting everything on DB

```sh
#! /usr/bin/bash
-e # stop if it fails at any point

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-setup"
printf "${CYAN}==== ${LABEL}${NO_COLOR}\n"

bin_path="$(realpath .)/bin"

source "$bin_path/db-drop"
source "$bin_path/db-create"
source "$bin_path/db-schema-load"
source "$bin_path/db-seed"
```


