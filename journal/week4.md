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

### Seed our Postgres Database table with data

First of all we've been trying manually executing some commands to create and drop a database



For creating the database, tables and seed them with initial data we will be writting several scripts, that will be located in a bin folder inside backend-flask. These scripts will be using some sql files that will be located in a sql folder in the same root location

`/backend-flask/bin`
`/backend-flask/sql`





