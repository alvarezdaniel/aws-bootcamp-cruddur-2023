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

### Watched Ashish's Week 4 - Security Considerations

[video](https://www.youtube.com/watch?v=UourWxz7iQg&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=45)

### Create RDS Postgres Instance

[video](https://www.youtube.com/watch?v=EtD7Kv5YCUs&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=46)

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

Database instance should be stopped temporarily, in order not to consume credits, because it is a VM in AWS

![](assets/week-4/02-db-stop-temp.png)

![](assets/week-4/03-db-stop-temp.png)

In this case, the database will be first in `Stopping` status, and then after some minutes, when the process completed, in `Stopped temporarily` status

![](assets/week-4/04-db-stopped.png)


### Bash scripting for common database actions

[video](https://www.youtube.com/watch?v=EtD7Kv5YCUs&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=46)

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

#### Shell script for connecting to DB

`bin/db-connect`: 

```sh
#! /usr/bin/bash

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-connect"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

if [ "$1" = "prod" ]; then
  echo "Running in production mode"
  URL=$PROD_CONNECTION_URL
else
  URL=$CONNECTION_URL
fi

NO_DB_URL=$(sed 's/\/cruddur//g' <<<"$URL")
psql $NO_DB_URL
```

> This script is used for testing database connection. It uses the defined connection url in the environment variable CONNECTION_URL

> As this variable contains database name, it must be trimmed out from its value, by using `sed` command (https://askubuntu.com/questions/595269/use-sed-on-a-string-variable-rather-than-a-file)

> Shell script output color is changed by using this code (https://stackoverflow.com/questions/5947742/how-to-change-the-output-color-of-echo-in-linux):

```sh
CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-schema-load"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"
```

![](./assets/week-4/05-db-connect.png)


#### Shell script for creating DB

`bin/db-create`: 

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

> psql or createdb AWS cli commands can be used

![](./assets/week-4/06-db-create.png)


#### Shell script for dropping DB

`bin/db-drop`: 

```sh
#! /usr/bin/bash

CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-drop"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
psql $NO_DB_CONNECTION_URL -c "drop database cruddur;"
```

![](./assets/week-4/07-db-drop.png)


#### Shell script for loading DB schema 

`bin/db-schema-load`: 

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

![](./assets/week-4/08-db-schema-load.png)



#### Shell script for loading seed data into DB 

`bin/db-seed`: 

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
  );
```

![](./assets/week-4/09-db-seed.png)


#### Shell script for checking used connections to DB 

`bin/db-sessions`: 

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

![](./assets/week-4/10-db-sessions.png)


#### Shell script for resetting everything on DB 

`bin/db-setup`: 

```sh
#! /usr/bin/bash -e # stop if it fails at any point

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

![](./assets/week-4/11-db-setup.png)


### Install Postgres Driver in Backend Application

[video](https://www.youtube.com/watch?v=Sa2iB33sKFo&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=47)

We are going to install some python modules in `requirements.txt` file:

```txt
psycopg[binary]
psycopg[pool]
```

> https://www.psycopg.org/psycopg3/

and then execute

```sh
pip install -r requirements.txt
```

```
Collecting flask
  Downloading Flask-2.2.3-py3-none-any.whl (101 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 101.8/101.8 kB 5.3 MB/s eta 0:00:00
Collecting flask-cors
  Downloading Flask_Cors-3.0.10-py2.py3-none-any.whl (14 kB)
Collecting opentelemetry-api
  Downloading opentelemetry_api-1.16.0-py3-none-any.whl (57 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 57.3/57.3 kB 14.4 MB/s eta 0:00:00
Collecting opentelemetry-sdk
  Downloading opentelemetry_sdk-1.16.0-py3-none-any.whl (94 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 94.6/94.6 kB 13.9 MB/s eta 0:00:00
Collecting opentelemetry-exporter-otlp-proto-http
  Downloading opentelemetry_exporter_otlp_proto_http-1.16.0-py3-none-any.whl (21 kB)
Collecting opentelemetry-instrumentation-flask
  Downloading opentelemetry_instrumentation_flask-0.37b0-py3-none-any.whl (13 kB)
Collecting opentelemetry-instrumentation-requests
  Downloading opentelemetry_instrumentation_requests-0.37b0-py3-none-any.whl (11 kB)
Collecting aws-xray-sdk
  Downloading aws_xray_sdk-2.11.0-py2.py3-none-any.whl (102 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 102.0/102.0 kB 25.2 MB/s eta 0:00:00
Collecting watchtower
  Downloading watchtower-3.0.1-py3-none-any.whl (17 kB)
Collecting blinker
  Downloading blinker-1.5-py2.py3-none-any.whl (12 kB)
Collecting rollbar
  Downloading rollbar-0.16.3-py3-none-any.whl (98 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 98.1/98.1 kB 24.8 MB/s eta 0:00:00
Collecting Flask-AWSCognito
  Downloading Flask_AWSCognito-1.3-py3-none-any.whl (12 kB)
Collecting click>=8.0
  Downloading click-8.1.3-py3-none-any.whl (96 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 96.6/96.6 kB 18.7 MB/s eta 0:00:00
Requirement already satisfied: importlib-metadata>=3.6.0 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from flask->-r requirements.txt (line 1)) (6.0.0)
Collecting Werkzeug>=2.2.2
  Downloading Werkzeug-2.2.3-py3-none-any.whl (233 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 233.6/233.6 kB 18.8 MB/s eta 0:00:00
Collecting itsdangerous>=2.0
  Downloading itsdangerous-2.1.2-py3-none-any.whl (15 kB)
Requirement already satisfied: Jinja2>=3.0 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from flask->-r requirements.txt (line 1)) (3.1.2)
Requirement already satisfied: Six in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from flask-cors->-r requirements.txt (line 2)) (1.16.0)
Requirement already satisfied: setuptools>=16.0 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from opentelemetry-api->-r requirements.txt (line 4)) (65.6.3)
Collecting deprecated>=1.2.6
  Downloading Deprecated-1.2.13-py2.py3-none-any.whl (9.6 kB)
Requirement already satisfied: typing-extensions>=3.7.4 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from opentelemetry-sdk->-r requirements.txt (line 5)) (4.4.0)
Collecting opentelemetry-semantic-conventions==0.37b0
  Downloading opentelemetry_semantic_conventions-0.37b0-py3-none-any.whl (26 kB)
Collecting opentelemetry-proto==1.16.0
  Downloading opentelemetry_proto-1.16.0-py3-none-any.whl (52 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 52.6/52.6 kB 13.2 MB/s eta 0:00:00
Collecting googleapis-common-protos~=1.52
  Downloading googleapis_common_protos-1.58.0-py2.py3-none-any.whl (223 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 223.0/223.0 kB 32.8 MB/s eta 0:00:00
Requirement already satisfied: requests~=2.7 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from opentelemetry-exporter-otlp-proto-http->-r requirements.txt (line 6)) (2.28.1)
Collecting backoff<3.0.0,>=1.10.0
  Downloading backoff-2.2.1-py3-none-any.whl (15 kB)
Collecting protobuf<5.0,>=3.19
  Downloading protobuf-4.22.1-cp37-abi3-manylinux2014_x86_64.whl (302 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 302.4/302.4 kB 26.3 MB/s eta 0:00:00
Collecting opentelemetry-instrumentation==0.37b0
  Downloading opentelemetry_instrumentation-0.37b0-py3-none-any.whl (24 kB)
Collecting opentelemetry-util-http==0.37b0
  Downloading opentelemetry_util_http-0.37b0-py3-none-any.whl (6.7 kB)
Collecting opentelemetry-instrumentation-wsgi==0.37b0
  Downloading opentelemetry_instrumentation_wsgi-0.37b0-py3-none-any.whl (12 kB)
Requirement already satisfied: wrapt<2.0.0,>=1.0.0 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from opentelemetry-instrumentation==0.37b0->opentelemetry-instrumentation-flask->-r requirements.txt (line 7)) (1.14.1)
Collecting botocore>=1.11.3
  Downloading botocore-1.29.92-py3-none-any.whl (10.5 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 10.5/10.5 MB 93.5 MB/s eta 0:00:00
Collecting boto3<2,>=1.9.253
  Downloading boto3-1.26.92-py3-none-any.whl (134 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 134.7/134.7 kB 36.4 MB/s eta 0:00:00
Collecting python-jose
  Downloading python_jose-3.3.0-py2.py3-none-any.whl (33 kB)
Collecting s3transfer<0.7.0,>=0.6.0
  Downloading s3transfer-0.6.0-py3-none-any.whl (79 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 79.6/79.6 kB 16.3 MB/s eta 0:00:00
Collecting jmespath<2.0.0,>=0.7.1
  Downloading jmespath-1.0.1-py3-none-any.whl (20 kB)
Requirement already satisfied: urllib3<1.27,>=1.25.4 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from botocore>=1.11.3->aws-xray-sdk->-r requirements.txt (line 10)) (1.26.13)
Requirement already satisfied: python-dateutil<3.0.0,>=2.1 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from botocore>=1.11.3->aws-xray-sdk->-r requirements.txt (line 10)) (2.8.2)
Requirement already satisfied: zipp>=0.5 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from importlib-metadata>=3.6.0->flask->-r requirements.txt (line 1)) (3.11.0)
Requirement already satisfied: MarkupSafe>=2.0 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from Jinja2>=3.0->flask->-r requirements.txt (line 1)) (2.1.1)
Requirement already satisfied: charset-normalizer<3,>=2 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from requests~=2.7->opentelemetry-exporter-otlp-proto-http->-r requirements.txt (line 6)) (2.1.1)
Requirement already satisfied: certifi>=2017.4.17 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from requests~=2.7->opentelemetry-exporter-otlp-proto-http->-r requirements.txt (line 6)) (2022.12.7)
Requirement already satisfied: idna<4,>=2.5 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from requests~=2.7->opentelemetry-exporter-otlp-proto-http->-r requirements.txt (line 6)) (3.4)
Collecting ecdsa!=0.15
  Downloading ecdsa-0.18.0-py2.py3-none-any.whl (142 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 142.9/142.9 kB 38.7 MB/s eta 0:00:00
Collecting pyasn1
  Downloading pyasn1-0.4.8-py2.py3-none-any.whl (77 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 77.1/77.1 kB 17.4 MB/s eta 0:00:00
Collecting rsa
  Downloading rsa-4.9-py3-none-any.whl (34 kB)
Installing collected packages: pyasn1, Werkzeug, rsa, protobuf, opentelemetry-util-http, opentelemetry-semantic-conventions, jmespath, itsdangerous, ecdsa, deprecated, click, blinker, backoff, rollbar, python-jose, opentelemetry-proto, opentelemetry-api, googleapis-common-protos, flask, botocore, s3transfer, opentelemetry-sdk, opentelemetry-instrumentation, flask-cors, Flask-AWSCognito, aws-xray-sdk, opentelemetry-instrumentation-wsgi, opentelemetry-instrumentation-requests, opentelemetry-exporter-otlp-proto-http, boto3, watchtower, opentelemetry-instrumentation-flask
Successfully installed Flask-AWSCognito-1.3 Werkzeug-2.2.3 aws-xray-sdk-2.11.0 backoff-2.2.1 blinker-1.5 boto3-1.26.92 botocore-1.29.92 click-8.1.3 deprecated-1.2.13 ecdsa-0.18.0 flask-2.2.3 flask-cors-3.0.10 googleapis-common-protos-1.58.0 itsdangerous-2.1.2 jmespath-1.0.1 opentelemetry-api-1.16.0 opentelemetry-exporter-otlp-proto-http-1.16.0 opentelemetry-instrumentation-0.37b0 opentelemetry-instrumentation-flask-0.37b0 opentelemetry-instrumentation-requests-0.37b0 opentelemetry-instrumentation-wsgi-0.37b0 opentelemetry-proto-1.16.0 opentelemetry-sdk-1.16.0 opentelemetry-semantic-conventions-0.37b0 opentelemetry-util-http-0.37b0 protobuf-4.22.1 pyasn1-0.4.8 python-jose-3.3.0 rollbar-0.16.3 rsa-4.9 s3transfer-0.6.0 watchtower-3.0.1

[notice] A new release of pip available: 22.3.1 -> 23.0.1
[notice] To update, run: pip install --upgrade pip
```

We are going to add a db library to our backend folder in `lib/db.py`

```py
from psycopg_pool import ConnectionPool
import os

def query_wrap_object(template):
  sql = f"""
  (SELECT COALESCE(row_to_json(object_row),'{{}}'::json) FROM (
  {template}
  ) object_row);
  """
  return sql

def query_wrap_array(template):
  sql = f"""
  (SELECT COALESCE(array_to_json(array_agg(row_to_json(array_row))),'[]'::json) FROM (
  {template}
  ) array_row);
  """
  return sql

connection_url = os.getenv("CONNECTION_URL")
pool = ConnectionPool(connection_url)
```

> This library will be useful for obtaining the connection and the pool for connecting to Postgres. 

> Also, this will provide some functions for converting data returned from db to be shown in json format

A new environment variable is required to be added to backend service in compose file

```yml
version: "3.8"
services:
  backend-flask:
    environment:
      CONNECTION_URL: "${COMPOSE_CONNECTION_URL}"
```

And this references this new env var to be created as well (We cannot use the same CONNECTION_URL already created, because that one references localhost, and in here we need to use db, as the service name defined in compose file)

```sh
export COMPOSE_CONNECTION_URL="postgresql://postgres:password@db:5432/cruddur"
gp env COMPOSE_CONNECTION_URL="postgresql://postgres:password@db:5432/cruddur"
```

Finally, in `home_activities.py`, we need to change the returned information in there and use a call to postgress to retrieve the data (of course, we need to import first from lib.db the classes we need to use in here)

```py
from datetime import datetime, timedelta, timezone
from opentelemetry import trace
from lib.db import pool, query_wrap_array

tracer = trace.get_tracer("home.activities")

# Authentication
class HomeActivities:
  #def run(logger):
  def run(logger, cognito_user_id=None):
    #logger.info('Hello Cloudwatch! from /api/activities/home')
    
    with tracer.start_as_current_span("home-activites-mock-data"):
      span = trace.get_current_span()
      now = datetime.now(timezone.utc).astimezone()
      span.set_attribute("app.now", now.isoformat())
      
      """ no more hardcoded data
      results = [{
        'uuid': '68f126b0-1ceb-4a33-88be-d90fa7109eee',
        'handle':  'Andrew Brown',
        'message': 'Cloud is very fun!',
        'created_at': (now - timedelta(days=2)).isoformat(),
        'expires_at': (now + timedelta(days=5)).isoformat(),
        'likes_count': 5,
        'replies_count': 1,
        'reposts_count': 0,
        'replies': [{
          'uuid': '26e12864-1c26-5c3a-9658-97a10f8fea67',
          'reply_to_activity_uuid': '68f126b0-1ceb-4a33-88be-d90fa7109eee',
          'handle':  'Worf',
          'message': 'This post has no honor!',
          'likes_count': 0,
          'replies_count': 0,
          'reposts_count': 0,
          'created_at': (now - timedelta(days=2)).isoformat()
        }],
      },
      {
        'uuid': '66e12864-8c26-4c3a-9658-95a10f8fea67',
        'handle':  'Worf',
        'message': 'I am out of prune juice',
        'created_at': (now - timedelta(days=7)).isoformat(),
        'expires_at': (now + timedelta(days=9)).isoformat(),
        'likes': 0,
        'replies': []
      },
      {
        'uuid': '248959df-3079-4947-b847-9e0892d1bab4',
        'handle':  'Garek',
        'message': 'My dear doctor, I am just simple tailor',
        'created_at': (now - timedelta(hours=1)).isoformat(),
        'expires_at': (now + timedelta(hours=12)).isoformat(),
        'likes': 0,
        'replies': []
      }
      ]
      
      # Authentication
      if cognito_user_id != None:
        extra_crud = {
          'uuid': '248959df-3079-4947-b847-9e0892d1bab4',
          'handle':  'Lore',
          'message': 'My dear brother, it the humans that are the problem',
          'created_at': (now - timedelta(hours=1)).isoformat(),
          'expires_at': (now + timedelta(hours=12)).isoformat(),
          'likes': 1042,
          'replies': []
        }
        
        results.insert(0,extra_crud)
      
      span.set_attribute("app.result_length", len(results))
      
      return results      
      """

      sql = query_wrap_array("""
        SELECT
          activities.uuid,
          users.display_name,
          users.handle,
          activities.message,
          activities.replies_count,
          activities.reposts_count,
          activities.likes_count,
          activities.reply_to_activity_uuid,
          activities.expires_at,
          activities.created_at
        FROM public.activities
        LEFT JOIN public.users ON users.uuid = activities.user_uuid
        ORDER BY activities.created_at DESC
      """)
      print("SQL--------------")
      print(sql)
      with pool.connection() as conn:
        with conn.cursor() as cur:
          cur.execute(sql)
          # this will return a tuple
          # the first field being the data
          json = cur.fetchone()
      return json[0]
```

We can test the changes by starting the compose file and browsing cruddur home page

![](./assets/week-4/12-home-from-db.png)

> We can see that the returned activities are the ones defined in seed.sql file, and now extracted from postgres db

