# Week 5 — DynamoDB and Serverless Caching

## Week 5 instructors

- Andrew Brown [@andrewbrown](https://twitter.com/andrewbrown)
- Shala Warner [@GiftedLane](https://twitter.com/GiftedLane) (Student Advocate)
- Kirk Kirkconnell [@NoSQLKnowHow](https://twitter.com/NoSQLKnowHow) (DynamoDB and Serverless caching)

## Class Summary

- Data Modelling a Direct Messaging System using Single Table Design
- Implementing DynamoDB query using Single Table Design
- Provisioning DynamoDB tables with Provisioned Capacity
- Utilizing a Global Secondary Index (GSI) with DynamoDB
- Rapid data modelling and implementation of DynamoDB with DynamoDB Local
- Writing utility scripts to easily setup and teardown and debug DynamoDB data

## Required tasks

- Learn about Data Modelling (Single Table Design) for NoSQL.
- Implement Schema Load Script
- Implement Seed Script
- Implement Scan Script
- Implement Pattern Scripts for Read and List Conversations
- Implement Update Cognito ID Script for Postgres Database
- Implement (Pattern A) Listing Messages in Message Group into Application
- Implement (Pattern B) Listing Messages Group into Application
- Implement (Pattern C) Creating a Message for an existing Message Group into Application
- Implement (Pattern D) Creating a Message for a new Message Group into Application
- Implement (Pattern E) Updating a Message Group using DynamoDB Streams

## Todo Checklist

### Watched Week 5 - Data Modelling (Live Stream)

https://www.youtube.com/watch?v=5oZHNOaL8Og&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=50

DynamoDB Modeling
https://lucid.app/lucidchart/8f58a19d-3821-4529-920f-5bb802d6c6a3/edit?invitationId=inv_e47bc316-9caa-4aee-940f-161e01e22715#

Notes

> No joins in DynamoDB (in NoSQL)
> The quantity of tables depends on the design (in this case is a single table design)
> Design = what data do we need, when do we need it, what's the velocity that we need it
> Message groups = conversations
> Identify use patterns
> PartiQL: https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/ql-reference.html
> DynamoDB primary keys = simple primary key o composite primary key

Livestream DynamoDB Data Modelling Excel File
https://docs.google.com/spreadsheets/d/1LrTC_y2X_YBEthFNlnwbo8TgxGlpHV3TCpM6XeVyiGg/edit#gid=0

![](./assets/week-5/02.png)

> DynamoDB — Introduction (AWS Developer Associate 2020) https://www.youtube.com/watch?v=Ih8Bxtt5Ekw&list=PLBfufR7vyJJ5WuCNg2em7SgdAfjduqnNq&index=75

> GSI = Global Secondary Index

The Boundaries of DynamoDB

> When you write a query you have provide a Primary Key (equality) eg. pk = 'andrew'
> Are you allowed to "update" the Hash and Range?
> No, whenever you change a key (simple or composite) eg. pk or sk you have to create a new item.
> you have to delete the old one
> Key condition expressions for query only for RANGE, HASH is only equality
> Don't create UUID for entity if you don't have an access pattern for it

### Watched Ashish's Week 5 - DynamoDB Considerations

https://www.youtube.com/watch?v=gFPljPNnK2Q&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=52


### Implement Schema Load Script

https://www.youtube.com/watch?v=pIGi_9E_GwA&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=52

#### Enable local DynamoDB container

This week we will be working on DynamoDB, so we need to reenable DynamoDB container in `docker-compose.yml` file

```yml
version: "3.8"
services:
  dynamodb-local:
      # https://stackoverflow.com/questions/67533058/persist-local-dynamodb-data-in-volumes-lack-permission-unable-to-open-databa
      # We needed to add user:root to get this working.
      user: root
      command: "-jar DynamoDBLocal.jar -sharedDb -dbPath ./data"
      image: "amazon/dynamodb-local:latest"
      container_name: dynamodb-local
      ports:
        - "8000:8000"
      volumes:
        - "./docker/dynamodb:/home/dynamodblocal/data"
      working_dir: /home/dynamodblocal
```

#### Restructure db bash scripts

We will be adding scripts for DynamoDB, so for separating them from the ones already built for postgres, we will be adding a new folder. So we will restructure current scripts, moving the ones starting with db to a `db` folder, and the one starting with rds to a `rds` folder

> In `db/setup` script we need to change the referenced scripts making the same change

![](./assets/week-5/01.png)

> We also need to change `.gitpod.yml` file, for changing the postgres initial task when starting the workspace, fixing the right script folder and name

```
  - name: postgres
    init: |
      curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc|sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
      echo "deb http://apt.postgresql.org/pub/repos/apt/ `lsb_release -cs`-pgdg main" |sudo tee  /etc/apt/sources.list.d/pgdg.list
      sudo apt update
      sudo apt install -y postgresql-client-13 libpq-dev      
    command: |
      export GITPOD_IP=$(curl ifconfig.me)
      source  "$THEIA_WORKSPACE_ROOT/backend-flask/bin/rds/update-sg-rule"
```

#### Add DynamoDB bash scripts

We will add a new folder as a container for DynamoDB bash scripts

`./backend-flask/bin/ddb`

For interfacing with DynamoDB, in some scripts we will be using AWS SDK for python (Boto3), so we will need to add it to `requirements.txt` and install it

https://boto3.amazonaws.com/v1/documentation/api/latest/index.html

```txt
boto3
```

```sh
pip install -r requirements.txt
```

Also, we will add a init task for Gitpod to execute pip install on workspace initialization

`.gitpod.yml`

```yml
  - name: pip-install
    command: |
      cd backend-flask
      pip install -r requirements.txt
      exit
```

We need to run DynamoDB container and Postgres container for using these scripts

```sh
docker compose  -f "docker-compose.yml" up -d --build db dynamodb-local 
```

```
 ✔ Network aws-bootcamp-cruddur-2023_default  Created                                                                                    0.0s 
 ✔ Volume "aws-bootcamp-cruddur-2023_db"      Created                                                                                    0.0s 
 ✔ Container aws-bootcamp-cruddur-2023-db-1   Started                                                                                    0.5s 
 ✔ Container dynamodb-local                   Started 
```

Also, we need to initialize postgres database, running these scripts

```sh
./bin/db/setup
```

Result
```
==== db-setup
== db-drop
ERROR:  database "cruddur" does not exist
== db-create
CREATE DATABASE
== db-schema-load
/workspace/aws-bootcamp-cruddur-2023/backend-flask/db/schema.sql
CREATE EXTENSION
NOTICE:  table "users" does not exist, skipping
DROP TABLE
NOTICE:  table "activities" does not exist, skipping
DROP TABLE
CREATE TABLE
CREATE TABLE
== db-seed
/workspace/aws-bootcamp-cruddur-2023/backend-flask/db/seed.sql
INSERT 0 2
INSERT 0 1
```

Check that data is inserted in users table

```
== db-connect
psql (13.10 (Ubuntu 13.10-1.pgdg20.04+1))
Type "help" for help.

postgres=# \c cruddur
You are now connected to database "cruddur" as user "postgres".
cruddur=# select * from public.users;
                 uuid                 | display_name |   handle    |         email          | cognito_user_id |         created_at         
--------------------------------------+--------------+-------------+------------------------+-----------------+----------------------------
 3a28a20f-964f-42fe-8356-79804e4f753d | Andrew Brown | andrewbrown | andrewbrown@exampro.co | MOCK            | 2023-03-22 22:50:05.723748
 59e8e5e9-288b-4422-80d1-66eadd0fa162 | Andrew Bayko | bayko       | bayko@exampro.co       | MOCK            | 2023-03-22 22:50:05.723748
(2 rows)

cruddur=# \q
```


#### schema-load

The first script we will be adding is the one for creating the table with the corresponding schema, for storing cruddur messages

`./backend-flask/bin/ddb/schema-load`

```py
#!/usr/bin/env python3

import boto3
import sys

attrs = {
  'endpoint_url': 'http://localhost:8000'
}

if len(sys.argv) == 2:
  if "prod" in sys.argv[1]:
    attrs = {}

ddb = boto3.client('dynamodb',**attrs)

table_name = 'cruddur-messages'

response = ddb.create_table(
  TableName=table_name,
  AttributeDefinitions=[
    {
      'AttributeName': 'pk',
      'AttributeType': 'S'
    },
    {
      'AttributeName': 'sk',
      'AttributeType': 'S'
    },
  ],
  KeySchema=[
    {
      'AttributeName': 'pk',
      'KeyType': 'HASH'
    },
    {
      'AttributeName': 'sk',
      'KeyType': 'RANGE'
    },
  ],
  #GlobalSecondaryIndexes=[
  #],
  BillingMode='PROVISIONED',
  ProvisionedThroughput={
      'ReadCapacityUnits': 5,
      'WriteCapacityUnits': 5
  }
)

print(response)
```

> Execute permissions should be added to the new script by executing `chmod u+x ./bin/ddb/schema_load` 

> The script will be executed by python, so it should start with `#!/usr/bin/env python3`

> The default endpoint will be a localhost service

> A new table will be created in DynamoDB, using the name `cruddur-message` and with the required characteristics (attributes definitions, key schema, billing mode and provisioned throughput)

> https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GettingStarted.CreateTable.html

When executing the script, it will return information about the created table

```sh
./bin/ddb/schema-load
```

Result
```
{'TableDescription': {'AttributeDefinitions': [{'AttributeName': 'pk', 'AttributeType': 'S'}, {'AttributeName': 'sk', 'AttributeType': 'S'}], 'TableName': 'cruddur-message', 'KeySchema': [{'AttributeName': 'pk', 'KeyType': 'HASH'}, {'AttributeName': 'sk', 'KeyType': 'RANGE'}], 'TableStatus': 'ACTIVE', 'CreationDateTime': datetime.datetime(2023, 3, 21, 23, 36, 34, 261000, tzinfo=tzlocal()), 'ProvisionedThroughput': {'LastIncreaseDateTime': datetime.datetime(1970, 1, 1, 0, 0, tzinfo=tzlocal()), 'LastDecreaseDateTime': datetime.datetime(1970, 1, 1, 0, 0, tzinfo=tzlocal()), 'NumberOfDecreasesToday': 0, 'ReadCapacityUnits': 5, 'WriteCapacityUnits': 5}, 'TableSizeBytes': 0, 'ItemCount': 0, 'TableArn': 'arn:aws:dynamodb:ddblocal:000000000000:table/cruddur-message'}, 'ResponseMetadata': {'RequestId': '84448b1c-7840-4998-911e-4347f2b0c3c8', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Tue, 21 Mar 2023 23:36:33 GMT', 'x-amzn-requestid': '84448b1c-7840-4998-911e-4347f2b0c3c8', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2280689091', 'content-length': '578', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
```


#### list-tables

Now that we have the table created in DynamoDB, we will be building a script for listing the current tables

For this script we will be using AWS CLI directly, using bash interpreter instead of python SDK

`./backend-flask/bin/ddb/list-tables`

```sh
#! /usr/bin/bash
set -e # stop if it fails at any point

if [ "$1" = "prod" ]; then
  ENDPOINT_URL=""
else
  ENDPOINT_URL="--endpoint-url=http://localhost:8000"
fi

aws dynamodb list-tables $ENDPOINT_URL \
--query TableNames \
--output table
```

> https://docs.aws.amazon.com/cli/latest/reference/dynamodb/list-tables.html

> Execute permissions should be added to the new script by executing `chmod u+x ./bin/ddb/list-tables`

```sh
./bin/ddb/list-tables
```

Result
```
----------------------
|     ListTables     |
+--------------------+
|  cruddur-messages  |
+--------------------+
```


#### drop

Another script we will be adding is the one for dropping the created table, so we can recreate it again

`./backend-flask/bin/ddb/drop`

```sh
#! /usr/bin/bash

set -e # stop if it fails at any point

if [ -z "$1" ]; then
  echo "No TABLE_NAME argument supplied eg ./bin/ddb/drop cruddur-messages prod "
  exit 1
fi
TABLE_NAME=$1

if [ "$2" = "prod" ]; then
  ENDPOINT_URL=""
else
  ENDPOINT_URL="--endpoint-url=http://localhost:8000"
fi

echo "deleting table: $TABLE_NAME"

aws dynamodb delete-table $ENDPOINT_URL \
  --table-name $TABLE_NAME
```

> In this case we are using a bash script using AWS CLI commands to delete the DynamoDB table

> https://docs.aws.amazon.com/cli/latest/reference/dynamodb/delete-table.html

> set -e allows to stop executing the script if it fails at any point

> We need to give execute permissions also to this script by executing `chmod u+x ./bin/ddb/drop`

```sh
./bin/ddb/drop cruddur-messages
```

Result
```
deleting table: cruddur-messages
{
    "TableDescription": {
        "AttributeDefinitions": [
            {
                "AttributeName": "pk",
                "AttributeType": "S"
            },
            {
                "AttributeName": "sk",
                "AttributeType": "S"
            }
        ],
        "TableName": "cruddur-messages",
        "KeySchema": [
            {
                "AttributeName": "pk",
                "KeyType": "HASH"
            },
            {
                "AttributeName": "sk",
                "KeyType": "RANGE"
            }
        ],
        "TableStatus": "ACTIVE",
        "CreationDateTime": "2023-03-22T17:14:33.241000+00:00",
        "ProvisionedThroughput": {
            "LastIncreaseDateTime": "1970-01-01T00:00:00+00:00",
            "LastDecreaseDateTime": "1970-01-01T00:00:00+00:00",
            "NumberOfDecreasesToday": 0,
            "ReadCapacityUnits": 5,
            "WriteCapacityUnits": 5
        },
        "TableSizeBytes": 0,
        "ItemCount": 0,
        "TableArn": "arn:aws:dynamodb:ddblocal:000000000000:table/cruddur-messages"
    }
}
```


#### seed

This script will be used for inserting initial data into DynamoDB table

`./backend-flask/bin/ddb/seed`

```py
#!/usr/bin/env python3

import boto3
import os
import sys
from datetime import datetime, timedelta, timezone
import uuid

current_path = os.path.dirname(os.path.abspath(__file__))
parent_path = os.path.abspath(os.path.join(current_path, '..', '..'))
sys.path.append(parent_path)
from lib.db import db

attrs = {
  'endpoint_url': 'http://localhost:8000'
}

# unset endpoint url for use with production database
if len(sys.argv) == 2:
  if "prod" in sys.argv[1]:
    attrs = {}
ddb = boto3.client('dynamodb',**attrs)

def get_user_uuids():
  sql = """
    SELECT 
      users.uuid,
      users.display_name,
      users.handle
    FROM users
    WHERE
      users.handle IN(
        %(my_handle)s,
        %(other_handle)s
        )
  """
  users = db.query_array_json(sql,{
    'my_handle':  'andrewbrown',
    'other_handle': 'bayko'
  })
  my_user    = next((item for item in users if item["handle"] == 'andrewbrown'), None)
  other_user = next((item for item in users if item["handle"] == 'bayko'), None)
  results = {
    'my_user': my_user,
    'other_user': other_user
  }
  print('get_user_uuids')
  print(results)
  return results

def create_message_group(client,message_group_uuid, my_user_uuid, last_message_at=None, message=None, other_user_uuid=None, other_user_display_name=None, other_user_handle=None):
  table_name = 'cruddur-messages'
  record = {
    'pk':   {'S': f"GRP#{my_user_uuid}"},
    'sk':   {'S': last_message_at},
    'message_group_uuid': {'S': message_group_uuid},
    'message':  {'S': message},
    'user_uuid': {'S': other_user_uuid},
    'user_display_name': {'S': other_user_display_name},
    'user_handle': {'S': other_user_handle}
  }

  response = client.put_item(
    TableName=table_name,
    Item=record
  )
  print(response)

def create_message(client,message_group_uuid, created_at, message, my_user_uuid, my_user_display_name, my_user_handle):
  table_name = 'cruddur-messages'
  record = {
    'pk':   {'S': f"MSG#{message_group_uuid}"},
    'sk':   {'S': created_at },
    'message_uuid': { 'S': str(uuid.uuid4()) },
    'message': {'S': message},
    'user_uuid': {'S': my_user_uuid},
    'user_display_name': {'S': my_user_display_name},
    'user_handle': {'S': my_user_handle}
  }
  # insert the record into the table
  response = client.put_item(
    TableName=table_name,
    Item=record
  )
  # print the response
  print(response)

message_group_uuid = "5ae290ed-55d1-47a0-bc6d-fe2bc2700399" 
now = datetime.now(timezone.utc).astimezone()
users = get_user_uuids()

create_message_group(
  client=ddb,
  message_group_uuid=message_group_uuid,
  my_user_uuid=users['my_user']['uuid'],
  other_user_uuid=users['other_user']['uuid'],
  other_user_handle=users['other_user']['handle'],
  other_user_display_name=users['other_user']['display_name'],
  last_message_at=now.isoformat(),
  message="this is a filler message"
)

create_message_group(
  client=ddb,
  message_group_uuid=message_group_uuid,
  my_user_uuid=users['other_user']['uuid'],
  other_user_uuid=users['my_user']['uuid'],
  other_user_handle=users['my_user']['handle'],
  other_user_display_name=users['my_user']['display_name'],
  last_message_at=now.isoformat(),
  message="this is a filler message"
)

conversation = """
Person 1: Have you ever watched Babylon 5? It's one of my favorite TV shows!
Person 2: Yes, I have! I love it too. What's your favorite season?
Person 1: I think my favorite season has to be season 3. So many great episodes, like "Severed Dreams" and "War Without End."
Person 2: Yeah, season 3 was amazing! I also loved season 4, especially with the Shadow War heating up and the introduction of the White Star.
Person 1: Agreed, season 4 was really great as well. I was so glad they got to wrap up the storylines with the Shadows and the Vorlons in that season.
Person 2: Definitely. What about your favorite character? Mine is probably Londo Mollari.
Person 1: Londo is great! My favorite character is probably G'Kar. I loved his character development throughout the series.
Person 2: G'Kar was definitely a standout character. I also really liked Delenn's character arc and how she grew throughout the series.
Person 1: Delenn was amazing too, especially with her role in the Minbari Civil War and her relationship with Sheridan. Speaking of which, what did you think of the Sheridan character?
Person 2: I thought Sheridan was a great protagonist. He was a strong leader and had a lot of integrity. And his relationship with Delenn was so well-done.
Person 1: I totally agree! I also really liked the dynamic between Garibaldi and Bester. Those two had some great scenes together.
Person 2: Yes! Their interactions were always so intense and intriguing. And speaking of intense scenes, what did you think of the episode "Intersections in Real Time"?
Person 1: Oh man, that episode was intense. It was so well-done, but I could barely watch it. It was just too much.
Person 2: Yeah, it was definitely hard to watch. But it was also one of the best episodes of the series in my opinion.
Person 1: Absolutely. Babylon 5 had so many great episodes like that. Do you have a favorite standalone episode?
Person 2: Hmm, that's a tough one. I really loved "The Coming of Shadows" in season 2, but "A Voice in the Wilderness" in season 1 was also great. What about you?
Person 1: I think my favorite standalone episode might be "The Long Twilight Struggle" in season 2. It had some great moments with G'Kar and Londo.
Person 2: Yes, "The Long Twilight Struggle" was definitely a standout episode. Babylon 5 really had so many great episodes and moments throughout its run.
Person 1: Definitely. It's a shame it ended after only five seasons, but I'm glad we got the closure we did with the series finale.
Person 2: Yeah, the series finale was really well-done. It tied up a lot of loose ends and left us with a great sense of closure.
Person 1: It really did. Overall, Babylon 5 is just such a great show with fantastic characters, writing, and world-building.
Person 2: Agreed. It's one of my favorite sci-fi shows of all time and I'm always happy to revisit it.
Person 1: Same here. I think one of the things that makes Babylon 5 so special is its emphasis on politics and diplomacy. It's not just a show about space battles and aliens, but about the complex relationships between different species and their political maneuvering.
Person 2: Yes, that's definitely one of the show's strengths. And it's not just about big-picture politics, but also about personal relationships and the choices characters make.
Person 1: Exactly. I love how Babylon 5 explores themes of redemption, forgiveness, and sacrifice. Characters like G'Kar and Londo have such compelling arcs that are driven by their choices and actions.
Person 2: Yes, the character development in Babylon 5 is really top-notch. Even minor characters like Vir and Franklin get their moments to shine and grow over the course of the series.
Person 1: I couldn't agree more. And the way the show handles its themes is so nuanced and thought-provoking. For example, the idea of "the one" and how it's used by different characters in different ways.
Person 2: Yes, that's a really interesting theme to explore. And it's not just a one-dimensional concept, but something that's explored in different contexts and with different characters.
Person 1: And the show also does a great job of balancing humor and drama. There are so many funny moments in the show, but it never detracts from the serious themes and the high stakes.
Person 2: Absolutely. The humor is always organic and never feels forced. And the show isn't afraid to go dark when it needs to, like in "Intersections in Real Time" or the episode "Sleeping in Light."
Person 1: Yeah, those episodes are definitely tough to watch, but they're also some of the most powerful and memorable episodes of the series. And it's not just the writing that's great, but also the acting and the production values.
Person 2: Yes, the acting is fantastic across the board. From Bruce Boxleitner's performance as Sheridan to Peter Jurasik's portrayal of Londo, every actor brings their A-game. And the production design and special effects are really impressive for a TV show from the 90s.
Person 1: Definitely. Babylon 5 was really ahead of its time in terms of its visuals and special effects. And the fact that it was all done on a TV budget makes it even more impressive.
Person 2: Yeah, it's amazing what they were able to accomplish with the limited resources they had. It just goes to show how talented the people behind the show were.
Person 1: Agreed. It's no wonder that Babylon 5 has such a devoted fanbase, even all these years later. It's just such a well-crafted and timeless show.
Person 2: Absolutely. I'm glad we can still appreciate it and talk about it all these years later. It really is a show that stands the test of time.
Person 1: One thing I really appreciate about Babylon 5 is how it handles diversity and representation. It has a really diverse cast of characters from different species and backgrounds, and it doesn't shy away from exploring issues of prejudice and discrimination.
Person 2: Yes, that's a great point. The show was really ahead of its time in terms of its diverse cast and the way it tackled issues of race, gender, and sexuality. And it did so in a way that felt natural and integrated into the story.
Person 1: Definitely. It's great to see a show that's not afraid to tackle these issues head-on and address them in a thoughtful and nuanced way. And it's not just about representation, but also about exploring different cultures and ways of life.
Person 2: Yes, the show does a great job of world-building and creating distinct cultures for each of the species. And it's not just about their physical appearance, but also about their customs, beliefs, and values.
Person 1: Absolutely. It's one of the things that sets Babylon 5 apart from other sci-fi shows. The attention to detail and the thought that went into creating this universe is really impressive.
Person 2: And it's not just the aliens that are well-developed, but also the human characters. The show explores the different factions and political ideologies within EarthGov, as well as the different cultures and traditions on Earth.
Person 1: Yes, that's another great aspect of the show. It's not just about the conflicts between different species, but also about the internal struggles within humanity. And it's all tied together by the overarching plot of the Shadow War and the fate of the galaxy.
Person 2: Definitely. The show does a great job of balancing the episodic stories with the larger arc, so that every episode feels important and contributes to the overall narrative.
Person 1: And the show is also great at building up tension and suspense. The slow burn of the Shadow War and the mystery of the Vorlons and the Shadows kept me on the edge of my seat throughout the series.
Person 2: Yes, the show is really good at building up anticipation and delivering satisfying payoffs. Whether it's the resolution of a character arc or the climax of a season-long plotline, Babylon 5 always delivers.
Person 1: Agreed. It's just such a well-crafted and satisfying show, with so many memorable moments and characters. I'm really glad we got to talk about it today.
Person 2: Me too. It's always great to geek out about Babylon 5 with someone who appreciates it as much as I do!
Person 1: Yeah, it's always fun to discuss our favorite moments and characters from the show. And there are so many great moments to choose from!
Person 2: Definitely. I think one of the most memorable moments for me was the "goodbye" scene between G'Kar and Londo in the episode "Objects at Rest." It was such a poignant and emotional moment, and it really showed how far their characters had come.
Person 1: Yes, that was a really powerful scene. It was great to see these two former enemies come together and find common ground. And it was a great way to wrap up their character arcs.
Person 2: Another memorable moment for me was the speech that Sheridan gives in "Severed Dreams." It's such an iconic moment in the show, and it really encapsulates the themes of the series.
Person 1: Yes, that speech is definitely one of the highlights of the series. It's so well-written and well-delivered, and it really captures the sense of hope and defiance that the show is all about.
Person 2: And speaking of great speeches, what did you think of the "Ivanova is always right" speech from "Moments of Transition"?
Person 1: Oh man, that speech gives me chills every time I watch it. It's such a powerful moment for Ivanova, and it really shows her strength and determination as a leader.
Person 2: Yes, that speech is definitely a standout moment for Ivanova's character. And it's just one example of the great writing and character development in the show.
Person 1: Absolutely. It's a testament to the talent of the writers and actors that they were able to create such rich and complex characters with so much depth and nuance.
Person 2: And it's not just the main characters that are well-developed, but also the supporting characters like Marcus, Zack, and Lyta. They all have their own stories and struggles, and they all contribute to the larger narrative in meaningful ways.
Person 1: Definitely. Babylon 5 is just such a well-rounded and satisfying show in every way. It's no wonder that it's still beloved by fans all these years later.
Person 2: Agreed. It's a show that has stood the test of time, and it will always hold a special place in my heart as one of my favorite TV shows of all time.
Person 1: One of the most interesting ethical dilemmas presented in Babylon 5 is the treatment of the Narn by the Centauri. What do you think about that storyline?
Person 2: Yeah, it's definitely a difficult issue to grapple with. On the one hand, the Centauri were portrayed as the aggressors, and their treatment of the Narn was brutal and unjust. But on the other hand, the show also presented some nuance to the situation, with characters like Londo and Vir struggling with their own complicity in the conflict.
Person 1: Exactly. I think one of the strengths of the show is its willingness to explore complex ethical issues like this. It's not just about good guys versus bad guys, but about the shades of grey in between.
Person 2: Yeah, and it raises interesting questions about power and oppression. The Centauri had more advanced technology and military might than the Narn, which allowed them to dominate and subjugate the Narn people. But at the same time, there were also political and economic factors at play that contributed to the conflict.
Person 1: And it's not just about the actions of the Centauri government, but also about the actions of individual characters. Londo, for example, was initially portrayed as a somewhat sympathetic character, but as the series progressed, we saw how his choices and actions contributed to the suffering of the Narn people.
Person 2: Yes, and that raises interesting questions about personal responsibility and accountability. Can an individual be held responsible for the actions of their government or their society? And if so, to what extent?
Person 1: That's a really good point. And it's also interesting to consider the role of empathy and compassion in situations like this. Characters like G'Kar and Delenn showed compassion towards the Narn people and fought against their oppression, while others like Londo and Cartagia were more indifferent or even sadistic in their treatment of the Narn.
Person 2: Yeah, and that raises the question of whether empathy and compassion are innate traits, or whether they can be cultivated through education and exposure to different cultures and perspectives.
Person 1: Definitely. And it's also worth considering the role of forgiveness and reconciliation. The Narn and Centauri eventually came to a sort of reconciliation in the aftermath of the Shadow War, but it was a difficult and painful process that required a lot of sacrifice and forgiveness on both sides.
Person 2: Yes, and that raises the question of whether forgiveness is always possible or appropriate in situations of oppression and injustice. Can the victims of such oppression ever truly forgive their oppressors, or is that too much to ask?
Person 1: It's a tough question to answer. I think the show presents a hopeful message in the end, with characters like G'Kar and Londo finding a measure of redemption and reconciliation. But it's also clear that the scars of the conflict run deep and that healing takes time and effort.
Person 2: Yeah, that's a good point. Ultimately, I think the show's treatment of the Narn-Centauri conflict raises more questions than it answers, which is a testament to its complexity and nuance. It's a difficult issue to grapple with, but one that's worth exploring and discussing.
Person 1: Let's switch gears a bit and talk about the character of Natasha Alexander. What did you think about her role in the series?
Person 2: I thought Natasha Alexander was a really interesting character. She was a tough and competent security officer, but she also had a vulnerable side and a complicated past.
Person 1: Yeah, I agree. I think she added a lot of depth to the show and was a great foil to characters like Garibaldi and Zack.
Person 2: And I also appreciated the way the show handled her relationship with Garibaldi. It was clear that they had a history and a lot of unresolved tension, but the show never made it too melodramatic or over-the-top.
Person 1: That's a good point. I think the show did a good job of balancing the personal drama with the larger political and sci-fi elements. And it was refreshing to see a female character who was just as tough and competent as the male characters.
Person 2: Definitely. I think Natasha Alexander was a great example of a well-written and well-rounded female character. She wasn't just there to be eye candy or a love interest, but had her own story and agency.
Person 1: However, I did feel like the show could have done more with her character. She was introduced fairly late in the series, and didn't have as much screen time as some of the other characters.
Person 2: That's true. I think the show had a lot of characters to juggle, and sometimes that meant some characters got sidelined or didn't get as much development as they deserved.
Person 1: And I also thought that her storyline with Garibaldi could have been developed a bit more. They had a lot of history and tension between them, but it felt like it was resolved too quickly and neatly.
Person 2: I can see where you're coming from, but I also appreciated the way the show didn't drag out the drama unnecessarily. It was clear that they both had feelings for each other, but they also had to focus on their jobs and the larger conflicts at play.
Person 1: I can see that perspective as well. Overall, I think Natasha Alexander was a great addition to the show and added a lot of value to the series. It's a shame we didn't get to see more of her.
Person 2: Agreed. But at least the show was able to give her a satisfying arc and resolution in the end. And that's a testament to the show's strength as a whole.
Person 1: One thing that really stands out about Babylon 5 is the quality of the special effects. What did you think about the show's use of CGI and other visual effects?
Person 2: I thought the special effects in Babylon 5 were really impressive, especially for a show that aired in the 90s. The use of CGI to create the spaceships and other sci-fi elements was really innovative for its time.
Person 1: Yes, I was really blown away by the level of detail and realism in the effects. The ships looked so sleek and futuristic, and the space battles were really intense and exciting.
Person 2: And I also appreciated the way the show integrated the visual effects with the live-action footage. It never felt like the effects were taking over or overshadowing the characters or the story.
Person 1: Absolutely. The show had a great balance of practical effects and CGI, which helped to ground the sci-fi elements in a more tangible and realistic world.
Person 2: And it's also worth noting the way the show's use of visual effects evolved over the course of the series. The effects in the first season were a bit rough around the edges, but by the end of the series, they had really refined and perfected the look and feel of the show.
Person 1: Yes, I agree. And it's impressive how they were able to accomplish all of this on a TV budget. The fact that the show was able to create such a rich and immersive sci-fi universe with limited resources is a testament to the talent and creativity of the production team.
Person 2: Definitely. And it's one of the reasons why the show has aged so well. Even today, the visual effects still hold up and look impressive, which is a rarity for a show that's almost 30 years old.
Person 1: Agreed. And it's also worth noting the way the show's use of visual effects influenced other sci-fi shows that came after it. Babylon 5 really set the bar for what was possible in terms of sci-fi visuals on TV.
Person 2: Yes, it definitely had a big impact on the genre as a whole. And it's a great example of how innovative and groundbreaking sci-fi can be when it's done right.
Person 1: Another character I wanted to discuss is Zathras. What did you think of his character?
Person 2: Zathras was a really unique and memorable character. He was quirky and eccentric, but also had a lot of heart and sincerity.
Person 1: Yes, I thought he was a great addition to the show. He added some much-needed comic relief, but also had some important moments of character development.
Person 2: And I appreciated the way the show used him as a sort of plot device, with his knowledge of time and space being instrumental in the resolution of some of the show's major storylines.
Person 1: Definitely. It was a great way to integrate a seemingly minor character into the larger narrative. And it was also interesting to see the different versions of Zathras from different points in time.
Person 2: Yeah, that was a clever storytelling device that really added to the sci-fi elements of the show. And it was also a great showcase for actor Tim Choate, who played the character with so much charm and energy.
Person 1: I also thought that Zathras was a great example of the show's commitment to creating memorable and unique characters. Even characters that only appeared in a few episodes, like Zathras or Bester, were given distinct personalities and backstories.
Person 2: Yes, that's a good point. Babylon 5 was really great at creating a diverse and interesting cast of characters, with each one feeling like a fully-realized and distinct individual.
Person 1: And Zathras was just one example of that. He was a small but important part of the show's legacy, and he's still remembered fondly by fans today.
Person 2: Definitely. I think his character is a great example of the show's ability to balance humor and heart, and to create memorable and beloved characters that fans will cherish for years to come.
"""


lines = conversation.lstrip('\n').rstrip('\n').split('\n')
for i in range(len(lines)):
  if lines[i].startswith('Person 1: '):
    key = 'my_user'
    message = lines[i].replace('Person 1: ', '')
  elif lines[i].startswith('Person 2: '):
    key = 'other_user'
    message = lines[i].replace('Person 2: ', '')
  else:
    print(lines[i])
    raise 'invalid line'

  created_at = (now + timedelta(minutes=i)).isoformat()
  create_message(
    client=ddb,
    message_group_uuid=message_group_uuid,
    created_at=created_at,
    message=message,
    my_user_uuid=users[key]['uuid'],
    my_user_display_name=users[key]['display_name'],
    my_user_handle=users[key]['handle']
  )
  ```

> This script will use a predefined conversation, retrieve already configured users in postgres and create the corresponding messages in DynamoDB

> postgres container should be running because it retrieves the users from db

> It uses AWS SDK for python

> https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb/client/put_item.html

> We need to give execute permissions also to this script by executing `chmod u+x ./bin/ddb/seed`

```sh
./bin/ddb/seed
```

Result
```
 SQL STATEMENT-[array]------

    SELECT 
      users.uuid,
      users.display_name,
      users.handle
    FROM users
    WHERE
      users.handle IN(
        %(my_handle)s,
        %(other_handle)s
        )
  
get_user_uuids
{'my_user': {'uuid': '5cb26214-f187-4e2d-8c7a-5be82ae2c1f0', 'display_name': 'Andrew Brown', 'handle': 'andrewbrown'}, 'other_user': {'uuid': '3bab9192-a6bd-44c3-ab37-6acc10409d0f', 'display_name': 'Andrew Bayko', 'handle': 'bayko'}}
{'ResponseMetadata': {'RequestId': '84297905-c19b-4f52-b868-6aa68db8ce9f', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '84297905-c19b-4f52-b868-6aa68db8ce9f', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '7c89f4c2-f801-4987-897e-d0e723d3cf02', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '7c89f4c2-f801-4987-897e-d0e723d3cf02', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '18a1f5cc-82e4-45d6-a776-74bb50cf19be', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '18a1f5cc-82e4-45d6-a776-74bb50cf19be', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '1dd98ef3-16e5-4b41-a0cb-309934759e9e', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '1dd98ef3-16e5-4b41-a0cb-309934759e9e', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'ab5d778d-b785-4b69-b952-efb5a5b2f5a2', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': 'ab5d778d-b785-4b69-b952-efb5a5b2f5a2', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '473bd0b0-056d-4709-8c3a-dfe490e88dc7', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '473bd0b0-056d-4709-8c3a-dfe490e88dc7', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'a40f4c41-34d9-4232-a533-e4a33bfdf695', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': 'a40f4c41-34d9-4232-a533-e4a33bfdf695', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '1abb39fe-3039-4a74-86d8-33581401c9bd', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '1abb39fe-3039-4a74-86d8-33581401c9bd', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '4dc1e47a-d5ee-478e-958a-1bb347a2cb68', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '4dc1e47a-d5ee-478e-958a-1bb347a2cb68', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '33b89f27-c247-44f0-932f-33f232be176c', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '33b89f27-c247-44f0-932f-33f232be176c', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '01086ff8-abfe-446d-849c-a36dc74c6a2d', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '01086ff8-abfe-446d-849c-a36dc74c6a2d', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '10d4c834-a098-48a0-9dcb-6afea286cab3', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '10d4c834-a098-48a0-9dcb-6afea286cab3', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '28825a6d-31e8-4d69-bf40-c7d9bf4ce0bf', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '28825a6d-31e8-4d69-bf40-c7d9bf4ce0bf', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '13230177-3394-4c36-8084-a56bedd961c9', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '13230177-3394-4c36-8084-a56bedd961c9', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '78ccb05c-7ec6-45be-8dc3-4787e3cfdba1', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '78ccb05c-7ec6-45be-8dc3-4787e3cfdba1', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '6ebe04d5-9c3a-419e-b284-5a2d70e14a27', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '6ebe04d5-9c3a-419e-b284-5a2d70e14a27', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '413b2df2-f84f-4198-a5f7-24152893bb6a', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '413b2df2-f84f-4198-a5f7-24152893bb6a', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '172ad1b5-790a-4511-8f97-23e00ae62e09', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '172ad1b5-790a-4511-8f97-23e00ae62e09', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '03792210-b59e-492e-b29f-4693ef2411ff', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '03792210-b59e-492e-b29f-4693ef2411ff', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '7d950d28-4732-4d78-8ea9-10d5d84ddb46', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '7d950d28-4732-4d78-8ea9-10d5d84ddb46', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '3b957b3d-3d3d-49ec-a35c-79c088698db7', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '3b957b3d-3d3d-49ec-a35c-79c088698db7', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '406efa28-51c3-4497-a686-2981b0362ec9', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '406efa28-51c3-4497-a686-2981b0362ec9', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '56dc0181-9e2e-4ae5-9225-dfe4e8e5bfe6', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '56dc0181-9e2e-4ae5-9225-dfe4e8e5bfe6', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '88a16569-0d2e-4204-a739-36389878b312', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '88a16569-0d2e-4204-a739-36389878b312', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '61650f5f-cb93-4afd-a124-aeccf924a4f8', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '61650f5f-cb93-4afd-a124-aeccf924a4f8', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '5c7990ac-5da4-4dda-9abb-1e51f5331815', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '5c7990ac-5da4-4dda-9abb-1e51f5331815', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '83f44d2c-02f4-491c-b226-070f815b57dd', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '83f44d2c-02f4-491c-b226-070f815b57dd', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '16684a1a-eafa-4186-91b5-b1d9da702a21', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '16684a1a-eafa-4186-91b5-b1d9da702a21', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '11cff4d0-220e-4d98-a181-d16ab4911ad0', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '11cff4d0-220e-4d98-a181-d16ab4911ad0', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'f4dd596d-d78e-4b0d-84fd-e0042f3f4245', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': 'f4dd596d-d78e-4b0d-84fd-e0042f3f4245', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'b886177d-edc3-4bbd-86be-22b4a149c10f', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': 'b886177d-edc3-4bbd-86be-22b4a149c10f', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '1fdb312c-a815-44bb-a241-8569de361682', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '1fdb312c-a815-44bb-a241-8569de361682', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'e02bdae3-4807-4bfe-940e-c4572fc3de50', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': 'e02bdae3-4807-4bfe-940e-c4572fc3de50', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'b19c9323-a64e-4dc1-9b21-81f89a8b88a6', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': 'b19c9323-a64e-4dc1-9b21-81f89a8b88a6', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '7fd17942-effb-4b7b-9ac2-a58ffa917d80', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '7fd17942-effb-4b7b-9ac2-a58ffa917d80', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '3171f8df-66c1-4e3a-94db-53e0cc4b98df', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '3171f8df-66c1-4e3a-94db-53e0cc4b98df', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'b53b1645-6f7b-4ec2-a35a-c0e437eba3a6', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': 'b53b1645-6f7b-4ec2-a35a-c0e437eba3a6', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '4ccca54e-e042-4ef2-b102-5640dd34416d', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '4ccca54e-e042-4ef2-b102-5640dd34416d', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '92fe9cba-c265-4fcf-87d2-da8cab3029fd', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '92fe9cba-c265-4fcf-87d2-da8cab3029fd', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '35336e13-27f1-4262-b04a-05ea5dbfe722', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '35336e13-27f1-4262-b04a-05ea5dbfe722', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '827e5939-ccc2-470c-86bc-2d1bce36b3e0', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': '827e5939-ccc2-470c-86bc-2d1bce36b3e0', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'eb588f73-e9c6-45f0-8828-20624f8d1b75', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': 'eb588f73-e9c6-45f0-8828-20624f8d1b75', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'cdb666ac-caf3-4836-a35d-5568527dd9b7', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:46 GMT', 'x-amzn-requestid': 'cdb666ac-caf3-4836-a35d-5568527dd9b7', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '29c92d65-0035-4d24-9055-3831dc3767bd', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '29c92d65-0035-4d24-9055-3831dc3767bd', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'da88acf0-b57b-4ff1-b98f-60bfe69c5db4', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'da88acf0-b57b-4ff1-b98f-60bfe69c5db4', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '7638a960-fe2c-4c28-8a4e-0d2f4aaed5b5', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '7638a960-fe2c-4c28-8a4e-0d2f4aaed5b5', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'c6c8172d-b2d9-45de-a178-688005975669', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'c6c8172d-b2d9-45de-a178-688005975669', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '12b2797c-11c8-427f-9183-6b715cc56436', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '12b2797c-11c8-427f-9183-6b715cc56436', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'a5eac846-6686-4f49-9a8a-7b4d31cf0f9e', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'a5eac846-6686-4f49-9a8a-7b4d31cf0f9e', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '75ca5a5e-aae5-4714-ab2a-880a485c32c2', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '75ca5a5e-aae5-4714-ab2a-880a485c32c2', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '21402fa1-0daf-4713-843b-5c34f082d2fa', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '21402fa1-0daf-4713-843b-5c34f082d2fa', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'e709a346-0958-470d-b7b6-285680a33cdb', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'e709a346-0958-470d-b7b6-285680a33cdb', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'd985da41-86b6-463b-aa69-f19369e1fcca', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'd985da41-86b6-463b-aa69-f19369e1fcca', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '44b99b07-75d2-4baf-8f72-11691d44b38c', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '44b99b07-75d2-4baf-8f72-11691d44b38c', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'a85dbf40-a1ce-45c1-9bfc-0a62126eb1ff', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'a85dbf40-a1ce-45c1-9bfc-0a62126eb1ff', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '3608ea0c-2509-4f47-a1c7-f2dc5dd96595', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '3608ea0c-2509-4f47-a1c7-f2dc5dd96595', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '482cded2-560b-4d43-84ea-21b4d63cd312', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '482cded2-560b-4d43-84ea-21b4d63cd312', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'e1c11b88-9bcb-4fa3-9b13-f1e89904c3d8', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'e1c11b88-9bcb-4fa3-9b13-f1e89904c3d8', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'ccf33d3a-84d8-426c-902b-403549817b46', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'ccf33d3a-84d8-426c-902b-403549817b46', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'a6886ab8-63ec-463d-926d-67b56448bdb1', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'a6886ab8-63ec-463d-926d-67b56448bdb1', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'd39a408a-483c-47b3-8f21-334f3ddc604c', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'd39a408a-483c-47b3-8f21-334f3ddc604c', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '5171dab1-850c-4bb8-a086-37faa4b3a09e', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '5171dab1-850c-4bb8-a086-37faa4b3a09e', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '2a9f8a82-2f68-47ea-be24-2a899d5d44e4', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '2a9f8a82-2f68-47ea-be24-2a899d5d44e4', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '85619c14-4e8f-4f28-88d5-36ed742e20d4', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '85619c14-4e8f-4f28-88d5-36ed742e20d4', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '1508fa40-4890-48ef-8079-d64ad4601415', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '1508fa40-4890-48ef-8079-d64ad4601415', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '71a7c6f7-5117-4c1a-bb26-1d9dfba2409f', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '71a7c6f7-5117-4c1a-bb26-1d9dfba2409f', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'f35de31a-8187-481f-a478-46240e306c6c', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'f35de31a-8187-481f-a478-46240e306c6c', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '50682df9-1269-4edb-81ad-ead4d4ccb76b', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '50682df9-1269-4edb-81ad-ead4d4ccb76b', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '6d3079e6-6b97-42bc-b8a6-d7199579e63b', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '6d3079e6-6b97-42bc-b8a6-d7199579e63b', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '724f7ceb-ce44-434c-9ed5-d920bf966cbd', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '724f7ceb-ce44-434c-9ed5-d920bf966cbd', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '143d1cfc-7475-47c1-8552-b48deda7cb0a', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '143d1cfc-7475-47c1-8552-b48deda7cb0a', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '14897324-4f79-4dad-b393-e9d965ff6b48', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '14897324-4f79-4dad-b393-e9d965ff6b48', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'f945632b-0f58-43aa-8e0c-a2e99f445696', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'f945632b-0f58-43aa-8e0c-a2e99f445696', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '4d6bc67c-3a17-4b93-8a43-4fdb2e4a3710', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '4d6bc67c-3a17-4b93-8a43-4fdb2e4a3710', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '5a21274b-1532-4c03-af90-20d5bda74b54', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '5a21274b-1532-4c03-af90-20d5bda74b54', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'b3e0cf40-9e32-4988-a261-99a983266b06', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'b3e0cf40-9e32-4988-a261-99a983266b06', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'd997e56b-2206-4c01-9119-5844974cd914', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'd997e56b-2206-4c01-9119-5844974cd914', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'dc0740b1-793a-4cf6-96a6-92fb3b8fb02b', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'dc0740b1-793a-4cf6-96a6-92fb3b8fb02b', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'd8ba97aa-68b3-4078-a342-519eb2dacad9', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'd8ba97aa-68b3-4078-a342-519eb2dacad9', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '7239ab29-8b60-4989-aad3-a93f45e46eca', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '7239ab29-8b60-4989-aad3-a93f45e46eca', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '49835f29-e3e4-4c13-aa0b-1ec3aa2e469c', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '49835f29-e3e4-4c13-aa0b-1ec3aa2e469c', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'fb949667-1f93-4ac9-b50c-ca8493f4b392', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'fb949667-1f93-4ac9-b50c-ca8493f4b392', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'c1a10c1d-cf31-434f-bf7a-f5768bf1ebe2', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'c1a10c1d-cf31-434f-bf7a-f5768bf1ebe2', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'c385842c-331d-4ee1-8d3e-e667aaf28dd1', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'c385842c-331d-4ee1-8d3e-e667aaf28dd1', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '49ea828e-d313-45fc-93eb-5b7dd164426e', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '49ea828e-d313-45fc-93eb-5b7dd164426e', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'e93232df-32e4-4646-b456-cae5953b0f12', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'e93232df-32e4-4646-b456-cae5953b0f12', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '5fa0c896-f6c5-4be7-b5ce-220a24fb482f', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '5fa0c896-f6c5-4be7-b5ce-220a24fb482f', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '107f94d6-e8ca-48c5-9908-90599e2bb848', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '107f94d6-e8ca-48c5-9908-90599e2bb848', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'a5d5d184-2c12-4b36-808c-cbc87a77aa69', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'a5d5d184-2c12-4b36-808c-cbc87a77aa69', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'fee589d4-4cf1-475d-8ca6-b5836a276fda', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'fee589d4-4cf1-475d-8ca6-b5836a276fda', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'f6441b3a-161d-4a28-8f34-4deaf3e6bc4e', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'f6441b3a-161d-4a28-8f34-4deaf3e6bc4e', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '46b55f0a-88fb-4e6e-a9ed-9914a4dc70b7', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '46b55f0a-88fb-4e6e-a9ed-9914a4dc70b7', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'e0cb89f2-eb41-45c5-a055-10037bc4dd78', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'e0cb89f2-eb41-45c5-a055-10037bc4dd78', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '547c1c2e-b990-4996-8d03-b1869e57baf4', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '547c1c2e-b990-4996-8d03-b1869e57baf4', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '9d1f469f-ea52-4549-89fa-db3cf45ce727', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '9d1f469f-ea52-4549-89fa-db3cf45ce727', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '17d5d5f1-1a0f-4ef5-9baf-d24e6b140a8a', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '17d5d5f1-1a0f-4ef5-9baf-d24e6b140a8a', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '8ea87121-dac7-4fa3-b76f-5148f4dfc403', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '8ea87121-dac7-4fa3-b76f-5148f4dfc403', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '10560324-ce8f-499d-9601-2cc9b6a3d2ff', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '10560324-ce8f-499d-9601-2cc9b6a3d2ff', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '0db9073d-ad54-4630-acc8-b627960b753b', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '0db9073d-ad54-4630-acc8-b627960b753b', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '37620493-fc90-4532-8f5c-baf4c182f517', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '37620493-fc90-4532-8f5c-baf4c182f517', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'ec28c240-213c-4d18-81e6-ef4a919f0742', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'ec28c240-213c-4d18-81e6-ef4a919f0742', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'f7ad3b22-6ffe-45f1-b14f-47be6a9d56df', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'f7ad3b22-6ffe-45f1-b14f-47be6a9d56df', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '587b630a-2a74-494c-aac0-9118e792d807', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '587b630a-2a74-494c-aac0-9118e792d807', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '2deec2cc-a2c9-4f87-99bd-2f56f57d3d5d', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '2deec2cc-a2c9-4f87-99bd-2f56f57d3d5d', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': '9e67cf32-296f-4a12-9d02-c52684777fe9', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': '9e67cf32-296f-4a12-9d02-c52684777fe9', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
{'ResponseMetadata': {'RequestId': 'a4aca7b2-c361-4f5e-b258-c1d7eef8f876', 'HTTPStatusCode': 200, 'HTTPHeaders': {'date': 'Wed, 22 Mar 2023 17:56:47 GMT', 'x-amzn-requestid': 'a4aca7b2-c361-4f5e-b258-c1d7eef8f876', 'content-type': 'application/x-amz-json-1.0', 'x-amz-crc32': '2745614147', 'content-length': '2', 'server': 'Jetty(9.4.48.v20220622)'}, 'RetryAttempts': 0}}
```


#### scan

This script will be used for returning messages from DynamoDB table

`./backend-flask/bin/ddb/scan`

```py
#!/usr/bin/env python3

import boto3

attrs = {
  'endpoint_url': 'http://localhost:8000'
}
ddb = boto3.resource('dynamodb',**attrs)
table_name = 'cruddur-messages'

table = ddb.Table(table_name)
response = table.scan()

items = response['Items']
for item in items:
  print(item)
```

> This script is using AWS SDK boto3

> https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb/client/scan.html

> We need to give execute permissions also to this script by executing `chmod u+x ./bin/ddb/scan`

```sh
./bin/ddb/scan
```

Result
```
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'message_group_uuid': '5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'user_handle': 'bayko', 'sk': '2023-03-22T22:55:12.482495+00:00', 'pk': 'GRP#3a28a20f-964f-42fe-8356-79804e4f753d', 'message': 'this is a filler message', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'message_group_uuid': '5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T22:55:12.482495+00:00', 'pk': 'GRP#59e8e5e9-288b-4422-80d1-66eadd0fa162', 'message': 'this is a filler message', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T22:55:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '57fa54d5-e0af-43fe-a887-74358bb39397', 'message': "Have you ever watched Babylon 5? It's one of my favorite TV shows!", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T22:56:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'cef01047-9619-4a1a-8cb5-41ab3065311a', 'message': "Yes, I have! I love it too. What's your favorite season?", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T22:57:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '1b5d88b8-a489-4499-94c7-dcc93b97f2f6', 'message': 'I think my favorite season has to be season 3. So many great episodes, like "Severed Dreams" and "War Without End."', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T22:58:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '32d09357-dee9-4eb5-98ff-c576a6d83589', 'message': 'Yeah, season 3 was amazing! I also loved season 4, especially with the Shadow War heating up and the introduction of the White Star.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T22:59:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '666da4c0-51cc-40be-b299-24c608e485e2', 'message': 'Agreed, season 4 was really great as well. I was so glad they got to wrap up the storylines with the Shadows and the Vorlons in that season.', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:00:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'aecfc3de-a0a0-4f72-b21b-8dfc7b62c1f0', 'message': 'Definitely. What about your favorite character? Mine is probably Londo Mollari.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:01:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'd580a893-fdb5-45f1-99bb-1018c276ffc9', 'message': "Londo is great! My favorite character is probably G'Kar. I loved his character development throughout the series.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:02:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'beefe5a5-b71e-4182-a45e-60ef6f1af94c', 'message': "G'Kar was definitely a standout character. I also really liked Delenn's character arc and how she grew throughout the series.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:03:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '8ea48a93-bd60-47bc-8c61-04ca73c60f9e', 'message': 'Delenn was amazing too, especially with her role in the Minbari Civil War and her relationship with Sheridan. Speaking of which, what did you think of the Sheridan character?', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:04:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '2fc467b1-59b4-4647-87ab-fcc95545c110', 'message': 'I thought Sheridan was a great protagonist. He was a strong leader and had a lot of integrity. And his relationship with Delenn was so well-done.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:05:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '5c9172ca-4e82-4f84-9642-a2b0432de4c2', 'message': 'I totally agree! I also really liked the dynamic between Garibaldi and Bester. Those two had some great scenes together.', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:06:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '182c475f-c0ed-4617-b4f4-dcdd652bcac1', 'message': 'Yes! Their interactions were always so intense and intriguing. And speaking of intense scenes, what did you think of the episode "Intersections in Real Time"?', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:07:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'bfb7553a-5e8a-4c8c-a052-e98f36f0ec6f', 'message': 'Oh man, that episode was intense. It was so well-done, but I could barely watch it. It was just too much.', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:08:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '5760f5d6-ac9c-4946-abb2-dc8daa583277', 'message': 'Yeah, it was definitely hard to watch. But it was also one of the best episodes of the series in my opinion.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:09:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '9783ca79-b4fd-4614-8b44-568ab55668d8', 'message': 'Absolutely. Babylon 5 had so many great episodes like that. Do you have a favorite standalone episode?', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:10:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'f8b9058e-3c56-40a7-a2d9-2c2e987abd79', 'message': 'Hmm, that\'s a tough one. I really loved "The Coming of Shadows" in season 2, but "A Voice in the Wilderness" in season 1 was also great. What about you?', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:11:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '11002915-43d1-4997-8fab-99e789bdd5e4', 'message': 'I think my favorite standalone episode might be "The Long Twilight Struggle" in season 2. It had some great moments with G\'Kar and Londo.', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:12:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '9b7c9727-d32a-481a-a656-84cf33682535', 'message': 'Yes, "The Long Twilight Struggle" was definitely a standout episode. Babylon 5 really had so many great episodes and moments throughout its run.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:13:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '7612136b-a5b7-4b51-bdd9-16af05ec84f5', 'message': "Definitely. It's a shame it ended after only five seasons, but I'm glad we got the closure we did with the series finale.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:14:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '7eb2ca72-e4cf-476b-b9cf-24b9a2a7e084', 'message': 'Yeah, the series finale was really well-done. It tied up a lot of loose ends and left us with a great sense of closure.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:15:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '9d422aec-f008-4964-97aa-289f548f851d', 'message': 'It really did. Overall, Babylon 5 is just such a great show with fantastic characters, writing, and world-building.', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:16:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '854d8244-4fc6-407b-be17-4d5429a42422', 'message': "Agreed. It's one of my favorite sci-fi shows of all time and I'm always happy to revisit it.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:17:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '4ef2bd70-2ec0-41c0-8888-eeacb711d66e', 'message': "Same here. I think one of the things that makes Babylon 5 so special is its emphasis on politics and diplomacy. It's not just a show about space battles and aliens, but about the complex relationships between different species and their political maneuvering.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:18:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '4919281c-0fc8-48e9-b9d4-bd9e68903a27', 'message': "Yes, that's definitely one of the show's strengths. And it's not just about big-picture politics, but also about personal relationships and the choices characters make.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:19:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '2e31b598-6ad2-44a7-b3b4-e7d16d2c38d5', 'message': "Exactly. I love how Babylon 5 explores themes of redemption, forgiveness, and sacrifice. Characters like G'Kar and Londo have such compelling arcs that are driven by their choices and actions.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:20:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '36666718-fbec-4a5c-aba8-c8558dfe4547', 'message': 'Yes, the character development in Babylon 5 is really top-notch. Even minor characters like Vir and Franklin get their moments to shine and grow over the course of the series.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:21:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '5c6487b5-6854-41c4-a3b5-52bd66f31c16', 'message': 'I couldn\'t agree more. And the way the show handles its themes is so nuanced and thought-provoking. For example, the idea of "the one" and how it\'s used by different characters in different ways.', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:22:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '62b46820-45f4-4158-ad72-8a29f7a0c4a5', 'message': "Yes, that's a really interesting theme to explore. And it's not just a one-dimensional concept, but something that's explored in different contexts and with different characters.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:23:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '3db9ec47-b37e-4fd0-ad84-9acc77d46425', 'message': 'And the show also does a great job of balancing humor and drama. There are so many funny moments in the show, but it never detracts from the serious themes and the high stakes.', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:24:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '5b24ccc2-e8b8-461e-a35f-ff6691b84018', 'message': 'Absolutely. The humor is always organic and never feels forced. And the show isn\'t afraid to go dark when it needs to, like in "Intersections in Real Time" or the episode "Sleeping in Light."', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:25:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '4cc9317e-cdae-4067-ac07-36738606b454', 'message': "Yeah, those episodes are definitely tough to watch, but they're also some of the most powerful and memorable episodes of the series. And it's not just the writing that's great, but also the acting and the production values.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:26:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'd86b947e-fecf-4c09-988c-6d448280701c', 'message': "Yes, the acting is fantastic across the board. From Bruce Boxleitner's performance as Sheridan to Peter Jurasik's portrayal of Londo, every actor brings their A-game. And the production design and special effects are really impressive for a TV show from the 90s.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:27:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '4862f7c0-6f5d-4c01-bd2e-6798446a4b2f', 'message': 'Definitely. Babylon 5 was really ahead of its time in terms of its visuals and special effects. And the fact that it was all done on a TV budget makes it even more impressive.', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:28:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '327c4d56-7463-43b4-b248-c4b43dac8dba', 'message': "Yeah, it's amazing what they were able to accomplish with the limited resources they had. It just goes to show how talented the people behind the show were.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:29:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '055a6646-30af-4529-8620-667fafbedd4f', 'message': "Agreed. It's no wonder that Babylon 5 has such a devoted fanbase, even all these years later. It's just such a well-crafted and timeless show.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:30:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'f9cf1d91-c701-4bb0-91b6-3782085b2663', 'message': "Absolutely. I'm glad we can still appreciate it and talk about it all these years later. It really is a show that stands the test of time.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:31:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '59b48e37-8d73-405a-b5f3-404531a2582c', 'message': "One thing I really appreciate about Babylon 5 is how it handles diversity and representation. It has a really diverse cast of characters from different species and backgrounds, and it doesn't shy away from exploring issues of prejudice and discrimination.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:32:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '62ad7e12-7a58-4fdc-a3a9-85a19790ad63', 'message': "Yes, that's a great point. The show was really ahead of its time in terms of its diverse cast and the way it tackled issues of race, gender, and sexuality. And it did so in a way that felt natural and integrated into the story.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:33:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '33c4a8d1-1193-4321-86d0-2b056fe294e7', 'message': "Definitely. It's great to see a show that's not afraid to tackle these issues head-on and address them in a thoughtful and nuanced way. And it's not just about representation, but also about exploring different cultures and ways of life.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:34:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '07072a0d-bbdc-4ec7-b85c-254324a00ce4', 'message': "Yes, the show does a great job of world-building and creating distinct cultures for each of the species. And it's not just about their physical appearance, but also about their customs, beliefs, and values.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:35:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '28212622-2857-4b47-b000-1cc19dd5e760', 'message': "Absolutely. It's one of the things that sets Babylon 5 apart from other sci-fi shows. The attention to detail and the thought that went into creating this universe is really impressive.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:36:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '5c876d59-66b3-489d-9305-0ab5e97bd5a4', 'message': "And it's not just the aliens that are well-developed, but also the human characters. The show explores the different factions and political ideologies within EarthGov, as well as the different cultures and traditions on Earth.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:37:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '14ce6e3d-0479-4b3f-a850-e6b9594218d0', 'message': "Yes, that's another great aspect of the show. It's not just about the conflicts between different species, but also about the internal struggles within humanity. And it's all tied together by the overarching plot of the Shadow War and the fate of the galaxy.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:38:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'f8caf551-5c37-40c0-8915-32efe10deafd', 'message': 'Definitely. The show does a great job of balancing the episodic stories with the larger arc, so that every episode feels important and contributes to the overall narrative.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:39:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '13e51deb-3732-4a2e-ad35-ee60fc6ebb21', 'message': 'And the show is also great at building up tension and suspense. The slow burn of the Shadow War and the mystery of the Vorlons and the Shadows kept me on the edge of my seat throughout the series.', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:40:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '34440247-6b4d-418b-987f-ddbf63b76d93', 'message': "Yes, the show is really good at building up anticipation and delivering satisfying payoffs. Whether it's the resolution of a character arc or the climax of a season-long plotline, Babylon 5 always delivers.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:41:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '87e3db77-3d8c-4684-bca1-5da9727177b3', 'message': "Agreed. It's just such a well-crafted and satisfying show, with so many memorable moments and characters. I'm really glad we got to talk about it today.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:42:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '1047c56d-3225-426d-b3e5-b55ade4aa00e', 'message': "Me too. It's always great to geek out about Babylon 5 with someone who appreciates it as much as I do!", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:43:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'eca31bbf-61d2-43a3-afc5-e689c62cb26c', 'message': "Yeah, it's always fun to discuss our favorite moments and characters from the show. And there are so many great moments to choose from!", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:44:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'c4f49c7c-14e5-4ee8-aea5-b4a4ed75ffa3', 'message': 'Definitely. I think one of the most memorable moments for me was the "goodbye" scene between G\'Kar and Londo in the episode "Objects at Rest." It was such a poignant and emotional moment, and it really showed how far their characters had come.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:45:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '061fef86-db0d-4b96-b01d-bf61dd1697d9', 'message': 'Yes, that was a really powerful scene. It was great to see these two former enemies come together and find common ground. And it was a great way to wrap up their character arcs.', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:46:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'e6622720-818f-488c-991e-ef63a7628a0a', 'message': 'Another memorable moment for me was the speech that Sheridan gives in "Severed Dreams." It\'s such an iconic moment in the show, and it really encapsulates the themes of the series.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:47:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '4663d6a6-81c2-4d98-a80c-5c45292629c5', 'message': "Yes, that speech is definitely one of the highlights of the series. It's so well-written and well-delivered, and it really captures the sense of hope and defiance that the show is all about.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:48:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'b1f7e7d4-b894-4e0a-8372-ce8be7128162', 'message': 'And speaking of great speeches, what did you think of the "Ivanova is always right" speech from "Moments of Transition"?', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:49:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'a48d63a2-aa13-4f79-bcdc-f85b9ab3ebd8', 'message': "Oh man, that speech gives me chills every time I watch it. It's such a powerful moment for Ivanova, and it really shows her strength and determination as a leader.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:50:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '60c6aaf4-6fa1-468b-bac1-e4cbb96b7796', 'message': "Yes, that speech is definitely a standout moment for Ivanova's character. And it's just one example of the great writing and character development in the show.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:51:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '5ae4f75f-a7ae-459a-8a24-6c8317ad25f9', 'message': "Absolutely. It's a testament to the talent of the writers and actors that they were able to create such rich and complex characters with so much depth and nuance.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:52:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'c9da08b4-d05d-4fde-b9f8-d487502c3d64', 'message': "And it's not just the main characters that are well-developed, but also the supporting characters like Marcus, Zack, and Lyta. They all have their own stories and struggles, and they all contribute to the larger narrative in meaningful ways.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:53:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '19931c31-a674-4817-a2a4-a2b31a2a904d', 'message': "Definitely. Babylon 5 is just such a well-rounded and satisfying show in every way. It's no wonder that it's still beloved by fans all these years later.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:54:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '0dd3b41c-99db-406a-b8d9-0460502862df', 'message': "Agreed. It's a show that has stood the test of time, and it will always hold a special place in my heart as one of my favorite TV shows of all time.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:55:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '4cbdbce3-2b24-4611-9e47-3cea5e37f639', 'message': 'One of the most interesting ethical dilemmas presented in Babylon 5 is the treatment of the Narn by the Centauri. What do you think about that storyline?', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:56:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'd1f85b2b-8203-43df-b3af-7cae059c15cc', 'message': "Yeah, it's definitely a difficult issue to grapple with. On the one hand, the Centauri were portrayed as the aggressors, and their treatment of the Narn was brutal and unjust. But on the other hand, the show also presented some nuance to the situation, with characters like Londo and Vir struggling with their own complicity in the conflict.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:57:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '78119115-a4bd-4482-b119-89d0425d6bd4', 'message': "Exactly. I think one of the strengths of the show is its willingness to explore complex ethical issues like this. It's not just about good guys versus bad guys, but about the shades of grey in between.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-22T23:58:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '2249a935-d06a-467b-9dba-a060d14ca320', 'message': 'Yeah, and it raises interesting questions about power and oppression. The Centauri had more advanced technology and military might than the Narn, which allowed them to dominate and subjugate the Narn people. But at the same time, there were also political and economic factors at play that contributed to the conflict.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-22T23:59:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '17508e4e-deb3-4148-9241-9ea2837d98a6', 'message': "And it's not just about the actions of the Centauri government, but also about the actions of individual characters. Londo, for example, was initially portrayed as a somewhat sympathetic character, but as the series progressed, we saw how his choices and actions contributed to the suffering of the Narn people.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:00:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'e5c44880-0711-43f3-b340-205574f628c0', 'message': 'Yes, and that raises interesting questions about personal responsibility and accountability. Can an individual be held responsible for the actions of their government or their society? And if so, to what extent?', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:01:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '4351e664-b0eb-4b56-9d17-51fa678b5a1a', 'message': "That's a really good point. And it's also interesting to consider the role of empathy and compassion in situations like this. Characters like G'Kar and Delenn showed compassion towards the Narn people and fought against their oppression, while others like Londo and Cartagia were more indifferent or even sadistic in their treatment of the Narn.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:02:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '3c69c19e-749c-4eaa-91e4-c5a8dbf4782a', 'message': 'Yeah, and that raises the question of whether empathy and compassion are innate traits, or whether they can be cultivated through education and exposure to different cultures and perspectives.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:03:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '38edf44e-8b5a-433c-85f3-9093cb43ef4a', 'message': "Definitely. And it's also worth considering the role of forgiveness and reconciliation. The Narn and Centauri eventually came to a sort of reconciliation in the aftermath of the Shadow War, but it was a difficult and painful process that required a lot of sacrifice and forgiveness on both sides.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:04:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '2753a30e-7070-46b3-a27c-a9e059b8469c', 'message': 'Yes, and that raises the question of whether forgiveness is always possible or appropriate in situations of oppression and injustice. Can the victims of such oppression ever truly forgive their oppressors, or is that too much to ask?', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:05:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'a676de10-b6af-40b6-a915-56963a9d61c2', 'message': "It's a tough question to answer. I think the show presents a hopeful message in the end, with characters like G'Kar and Londo finding a measure of redemption and reconciliation. But it's also clear that the scars of the conflict run deep and that healing takes time and effort.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:06:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '32b58818-4eed-49a0-bd47-2bae7b567841', 'message': "Yeah, that's a good point. Ultimately, I think the show's treatment of the Narn-Centauri conflict raises more questions than it answers, which is a testament to its complexity and nuance. It's a difficult issue to grapple with, but one that's worth exploring and discussing.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:07:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'a3db1ec0-39f2-43f9-8ed8-a388985d7167', 'message': "Let's switch gears a bit and talk about the character of Natasha Alexander. What did you think about her role in the series?", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:08:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '9c7bf3b9-086d-4ef1-9314-0cfe56ff4ada', 'message': 'I thought Natasha Alexander was a really interesting character. She was a tough and competent security officer, but she also had a vulnerable side and a complicated past.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:09:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '074991a5-99de-408e-95ba-3234fc6b4403', 'message': 'Yeah, I agree. I think she added a lot of depth to the show and was a great foil to characters like Garibaldi and Zack.', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:10:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '79a7fc13-ac7f-48a8-a6e4-a2827ecd67bd', 'message': 'And I also appreciated the way the show handled her relationship with Garibaldi. It was clear that they had a history and a lot of unresolved tension, but the show never made it too melodramatic or over-the-top.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:11:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '62d5334f-e41f-4c76-9b90-ca2cd2ee2fb5', 'message': "That's a good point. I think the show did a good job of balancing the personal drama with the larger political and sci-fi elements. And it was refreshing to see a female character who was just as tough and competent as the male characters.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:12:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'abc04271-97fe-45c8-896d-e040bc2c531c', 'message': "Definitely. I think Natasha Alexander was a great example of a well-written and well-rounded female character. She wasn't just there to be eye candy or a love interest, but had her own story and agency.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:13:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '3f6fc222-8271-45e8-b1e8-4ed8f3ba87e4', 'message': "However, I did feel like the show could have done more with her character. She was introduced fairly late in the series, and didn't have as much screen time as some of the other characters.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:14:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'ce8843b7-77d9-46fb-a070-83be95926114', 'message': "That's true. I think the show had a lot of characters to juggle, and sometimes that meant some characters got sidelined or didn't get as much development as they deserved.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:15:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '875ea0a8-7a44-4cc8-a1e1-18f981709276', 'message': 'And I also thought that her storyline with Garibaldi could have been developed a bit more. They had a lot of history and tension between them, but it felt like it was resolved too quickly and neatly.', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:16:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '6dafa7a9-d609-4938-89f2-3c4ac103c43e', 'message': "I can see where you're coming from, but I also appreciated the way the show didn't drag out the drama unnecessarily. It was clear that they both had feelings for each other, but they also had to focus on their jobs and the larger conflicts at play.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:17:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '794ba291-9040-4a2b-8886-197dffafd018', 'message': "I can see that perspective as well. Overall, I think Natasha Alexander was a great addition to the show and added a lot of value to the series. It's a shame we didn't get to see more of her.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:18:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '67a17153-3a24-43d2-a760-800a7baed545', 'message': "Agreed. But at least the show was able to give her a satisfying arc and resolution in the end. And that's a testament to the show's strength as a whole.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:19:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '1b43d373-30bd-4457-998a-32f26da24adf', 'message': "One thing that really stands out about Babylon 5 is the quality of the special effects. What did you think about the show's use of CGI and other visual effects?", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:20:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'a70206c1-2f59-4dc4-a3c1-97fe06daefe0', 'message': 'I thought the special effects in Babylon 5 were really impressive, especially for a show that aired in the 90s. The use of CGI to create the spaceships and other sci-fi elements was really innovative for its time.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:21:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '5fe2e5a5-9bca-4cb2-a363-06f05039560c', 'message': 'Yes, I was really blown away by the level of detail and realism in the effects. The ships looked so sleek and futuristic, and the space battles were really intense and exciting.', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:22:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '1cc31b8f-4dd5-45c4-a76a-89551dc64739', 'message': 'And I also appreciated the way the show integrated the visual effects with the live-action footage. It never felt like the effects were taking over or overshadowing the characters or the story.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:23:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '85f57922-ec95-4139-9579-5f0ec3bc41f5', 'message': 'Absolutely. The show had a great balance of practical effects and CGI, which helped to ground the sci-fi elements in a more tangible and realistic world.', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:24:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '390fa78c-26e9-4776-a2cb-578c8c0a8df9', 'message': "And it's also worth noting the way the show's use of visual effects evolved over the course of the series. The effects in the first season were a bit rough around the edges, but by the end of the series, they had really refined and perfected the look and feel of the show.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:25:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'e41d2d32-5521-49be-8f82-fa373aa47e7e', 'message': "Yes, I agree. And it's impressive how they were able to accomplish all of this on a TV budget. The fact that the show was able to create such a rich and immersive sci-fi universe with limited resources is a testament to the talent and creativity of the production team.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:26:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'a138ad9a-4a4a-488b-8a58-8db8cc5a90e0', 'message': "Definitely. And it's one of the reasons why the show has aged so well. Even today, the visual effects still hold up and look impressive, which is a rarity for a show that's almost 30 years old.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:27:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'e62ae9ec-2605-401e-a1f0-2d2ddbe246e8', 'message': "Agreed. And it's also worth noting the way the show's use of visual effects influenced other sci-fi shows that came after it. Babylon 5 really set the bar for what was possible in terms of sci-fi visuals on TV.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:28:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'fb59d70b-4c6d-4f5d-957a-3c4cdc57571f', 'message': "Yes, it definitely had a big impact on the genre as a whole. And it's a great example of how innovative and groundbreaking sci-fi can be when it's done right.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:29:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '64e7f307-ba5e-4442-b95b-63266c9b4130', 'message': 'Another character I wanted to discuss is Zathras. What did you think of his character?', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:30:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '698887bf-2f8b-4326-bed0-8750597222ca', 'message': 'Zathras was a really unique and memorable character. He was quirky and eccentric, but also had a lot of heart and sincerity.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:31:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '4456b314-83a2-466f-8c3b-3bc80f5b122f', 'message': 'Yes, I thought he was a great addition to the show. He added some much-needed comic relief, but also had some important moments of character development.', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:32:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'a6d1a691-6306-4cb2-97d3-fa2fdaee0fb1', 'message': "And I appreciated the way the show used him as a sort of plot device, with his knowledge of time and space being instrumental in the resolution of some of the show's major storylines.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:33:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '3d932de0-e3cb-476a-a158-3e2ef891ff85', 'message': 'Definitely. It was a great way to integrate a seemingly minor character into the larger narrative. And it was also interesting to see the different versions of Zathras from different points in time.', 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:34:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'ddf54714-48fd-4fc6-ae9d-6158b8309b3d', 'message': 'Yeah, that was a clever storytelling device that really added to the sci-fi elements of the show. And it was also a great showcase for actor Tim Choate, who played the character with so much charm and energy.', 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:35:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '3360a9a6-810c-4e03-aea2-9cd7a7f18cf2', 'message': "I also thought that Zathras was a great example of the show's commitment to creating memorable and unique characters. Even characters that only appeared in a few episodes, like Zathras or Bester, were given distinct personalities and backstories.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:36:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '0d451a45-0243-4cbf-88ee-b0876bdd5cb6', 'message': "Yes, that's a good point. Babylon 5 was really great at creating a diverse and interesting cast of characters, with each one feeling like a fully-realized and distinct individual.", 'user_display_name': 'Andrew Bayko'}
{'user_uuid': '3a28a20f-964f-42fe-8356-79804e4f753d', 'user_handle': 'andrewbrown', 'sk': '2023-03-23T00:37:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '27167e93-2b7f-452e-a457-d9b2eb1eaeb3', 'message': "And Zathras was just one example of that. He was a small but important part of the show's legacy, and he's still remembered fondly by fans today.", 'user_display_name': 'Andrew Brown'}
{'user_uuid': '59e8e5e9-288b-4422-80d1-66eadd0fa162', 'user_handle': 'bayko', 'sk': '2023-03-23T00:38:12.482495+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'a9b0a2ae-ac54-4eba-97a6-0bf012799fac', 'message': "Definitely. I think his character is a great example of the show's ability to balance humor and heart, and to create memorable and beloved characters that fans will cherish for years to come.", 'user_display_name': 'Andrew Bayko'}
```


#### get-conversation

This script will be used for getting conversations

`./backend-flask/bin/ddb/patterns/get-conversation`

```py
#!/usr/bin/env python3

import boto3
import sys
import json
import datetime

attrs = {
  'endpoint_url': 'http://localhost:8000'
}

if len(sys.argv) == 2:
  if "prod" in sys.argv[1]:
    attrs = {}

dynamodb = boto3.client('dynamodb',**attrs)
table_name = 'cruddur-messages'

message_group_uuid = "5ae290ed-55d1-47a0-bc6d-fe2bc2700399"

# define the query parameters
query_params = {
  'TableName': table_name,
  'ScanIndexForward': False,
  'Limit': 20,
  'ReturnConsumedCapacity': 'TOTAL',
  'KeyConditionExpression': 'pk = :pk AND begins_with(sk,:year)',
  #'KeyConditionExpression': 'pk = :pk AND sk BETWEEN :start_date AND :end_date',
  'ExpressionAttributeValues': {
    ':year': {'S': '2023'},
    #":start_date": { "S": "2023-03-01T00:00:00.000000+00:00" },
    #":end_date": { "S": "2023-03-19T23:59:59.999999+00:00" },
    ':pk': {'S': f"MSG#{message_group_uuid}"}
  }
}


# query the table
response = dynamodb.query(**query_params)

# print the items returned by the query
print(json.dumps(response, sort_keys=True, indent=2))

# print the consumed capacity
print(json.dumps(response['ConsumedCapacity'], sort_keys=True, indent=2))

items = response['Items']
reversed_array = items[::-1]

for item in reversed_array:
  sender_handle = item['user_handle']['S']
  message       = item['message']['S']
  timestamp     = item['sk']['S']
  dt_object = datetime.datetime.strptime(timestamp, '%Y-%m-%dT%H:%M:%S.%f%z')
  formatted_datetime = dt_object.strftime('%Y-%m-%d %I:%M %p')
  print(f'{sender_handle: <12}{formatted_datetime: <22}{message[:40]}...')
```

> This script is using AWS SDK boto3

> https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb/client/query.html

> We need to give execute permissions also to this script by executing `chmod u+x ./bin/ddb/patterns/get-conversation`

```sh
./bin/ddb/patterns/get-conversation
```

Result
```
{
  "ConsumedCapacity": {
    "CapacityUnits": 1.0,
    "TableName": "cruddur-messages"
  },
  "Count": 20,
  "Items": [
    {
      "message": {
        "S": "Definitely. I think his character is a great example of the show's ability to balance humor and heart, and to create memorable and beloved characters that fans will cherish for years to come."
      },
      "message_uuid": {
        "S": "a9b0a2ae-ac54-4eba-97a6-0bf012799fac"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:38:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Bayko"
      },
      "user_handle": {
        "S": "bayko"
      },
      "user_uuid": {
        "S": "59e8e5e9-288b-4422-80d1-66eadd0fa162"
      }
    },
    {
      "message": {
        "S": "And Zathras was just one example of that. He was a small but important part of the show's legacy, and he's still remembered fondly by fans today."
      },
      "message_uuid": {
        "S": "27167e93-2b7f-452e-a457-d9b2eb1eaeb3"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:37:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Brown"
      },
      "user_handle": {
        "S": "andrewbrown"
      },
      "user_uuid": {
        "S": "3a28a20f-964f-42fe-8356-79804e4f753d"
      }
    },
    {
      "message": {
        "S": "Yes, that's a good point. Babylon 5 was really great at creating a diverse and interesting cast of characters, with each one feeling like a fully-realized and distinct individual."
      },
      "message_uuid": {
        "S": "0d451a45-0243-4cbf-88ee-b0876bdd5cb6"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:36:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Bayko"
      },
      "user_handle": {
        "S": "bayko"
      },
      "user_uuid": {
        "S": "59e8e5e9-288b-4422-80d1-66eadd0fa162"
      }
    },
    {
      "message": {
        "S": "I also thought that Zathras was a great example of the show's commitment to creating memorable and unique characters. Even characters that only appeared in a few episodes, like Zathras or Bester, were given distinct personalities and backstories."
      },
      "message_uuid": {
        "S": "3360a9a6-810c-4e03-aea2-9cd7a7f18cf2"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:35:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Brown"
      },
      "user_handle": {
        "S": "andrewbrown"
      },
      "user_uuid": {
        "S": "3a28a20f-964f-42fe-8356-79804e4f753d"
      }
    },
    {
      "message": {
        "S": "Yeah, that was a clever storytelling device that really added to the sci-fi elements of the show. And it was also a great showcase for actor Tim Choate, who played the character with so much charm and energy."
      },
      "message_uuid": {
        "S": "ddf54714-48fd-4fc6-ae9d-6158b8309b3d"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:34:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Bayko"
      },
      "user_handle": {
        "S": "bayko"
      },
      "user_uuid": {
        "S": "59e8e5e9-288b-4422-80d1-66eadd0fa162"
      }
    },
    {
      "message": {
        "S": "Definitely. It was a great way to integrate a seemingly minor character into the larger narrative. And it was also interesting to see the different versions of Zathras from different points in time."
      },
      "message_uuid": {
        "S": "3d932de0-e3cb-476a-a158-3e2ef891ff85"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:33:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Brown"
      },
      "user_handle": {
        "S": "andrewbrown"
      },
      "user_uuid": {
        "S": "3a28a20f-964f-42fe-8356-79804e4f753d"
      }
    },
    {
      "message": {
        "S": "And I appreciated the way the show used him as a sort of plot device, with his knowledge of time and space being instrumental in the resolution of some of the show's major storylines."
      },
      "message_uuid": {
        "S": "a6d1a691-6306-4cb2-97d3-fa2fdaee0fb1"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:32:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Bayko"
      },
      "user_handle": {
        "S": "bayko"
      },
      "user_uuid": {
        "S": "59e8e5e9-288b-4422-80d1-66eadd0fa162"
      }
    },
    {
      "message": {
        "S": "Yes, I thought he was a great addition to the show. He added some much-needed comic relief, but also had some important moments of character development."
      },
      "message_uuid": {
        "S": "4456b314-83a2-466f-8c3b-3bc80f5b122f"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:31:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Brown"
      },
      "user_handle": {
        "S": "andrewbrown"
      },
      "user_uuid": {
        "S": "3a28a20f-964f-42fe-8356-79804e4f753d"
      }
    },
    {
      "message": {
        "S": "Zathras was a really unique and memorable character. He was quirky and eccentric, but also had a lot of heart and sincerity."
      },
      "message_uuid": {
        "S": "698887bf-2f8b-4326-bed0-8750597222ca"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:30:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Bayko"
      },
      "user_handle": {
        "S": "bayko"
      },
      "user_uuid": {
        "S": "59e8e5e9-288b-4422-80d1-66eadd0fa162"
      }
    },
    {
      "message": {
        "S": "Another character I wanted to discuss is Zathras. What did you think of his character?"
      },
      "message_uuid": {
        "S": "64e7f307-ba5e-4442-b95b-63266c9b4130"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:29:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Brown"
      },
      "user_handle": {
        "S": "andrewbrown"
      },
      "user_uuid": {
        "S": "3a28a20f-964f-42fe-8356-79804e4f753d"
      }
    },
    {
      "message": {
        "S": "Yes, it definitely had a big impact on the genre as a whole. And it's a great example of how innovative and groundbreaking sci-fi can be when it's done right."
      },
      "message_uuid": {
        "S": "fb59d70b-4c6d-4f5d-957a-3c4cdc57571f"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:28:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Bayko"
      },
      "user_handle": {
        "S": "bayko"
      },
      "user_uuid": {
        "S": "59e8e5e9-288b-4422-80d1-66eadd0fa162"
      }
    },
    {
      "message": {
        "S": "Agreed. And it's also worth noting the way the show's use of visual effects influenced other sci-fi shows that came after it. Babylon 5 really set the bar for what was possible in terms of sci-fi visuals on TV."
      },
      "message_uuid": {
        "S": "e62ae9ec-2605-401e-a1f0-2d2ddbe246e8"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:27:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Brown"
      },
      "user_handle": {
        "S": "andrewbrown"
      },
      "user_uuid": {
        "S": "3a28a20f-964f-42fe-8356-79804e4f753d"
      }
    },
    {
      "message": {
        "S": "Definitely. And it's one of the reasons why the show has aged so well. Even today, the visual effects still hold up and look impressive, which is a rarity for a show that's almost 30 years old."
      },
      "message_uuid": {
        "S": "a138ad9a-4a4a-488b-8a58-8db8cc5a90e0"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:26:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Bayko"
      },
      "user_handle": {
        "S": "bayko"
      },
      "user_uuid": {
        "S": "59e8e5e9-288b-4422-80d1-66eadd0fa162"
      }
    },
    {
      "message": {
        "S": "Yes, I agree. And it's impressive how they were able to accomplish all of this on a TV budget. The fact that the show was able to create such a rich and immersive sci-fi universe with limited resources is a testament to the talent and creativity of the production team."
      },
      "message_uuid": {
        "S": "e41d2d32-5521-49be-8f82-fa373aa47e7e"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:25:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Brown"
      },
      "user_handle": {
        "S": "andrewbrown"
      },
      "user_uuid": {
        "S": "3a28a20f-964f-42fe-8356-79804e4f753d"
      }
    },
    {
      "message": {
        "S": "And it's also worth noting the way the show's use of visual effects evolved over the course of the series. The effects in the first season were a bit rough around the edges, but by the end of the series, they had really refined and perfected the look and feel of the show."
      },
      "message_uuid": {
        "S": "390fa78c-26e9-4776-a2cb-578c8c0a8df9"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:24:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Bayko"
      },
      "user_handle": {
        "S": "bayko"
      },
      "user_uuid": {
        "S": "59e8e5e9-288b-4422-80d1-66eadd0fa162"
      }
    },
    {
      "message": {
        "S": "Absolutely. The show had a great balance of practical effects and CGI, which helped to ground the sci-fi elements in a more tangible and realistic world."
      },
      "message_uuid": {
        "S": "85f57922-ec95-4139-9579-5f0ec3bc41f5"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:23:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Brown"
      },
      "user_handle": {
        "S": "andrewbrown"
      },
      "user_uuid": {
        "S": "3a28a20f-964f-42fe-8356-79804e4f753d"
      }
    },
    {
      "message": {
        "S": "And I also appreciated the way the show integrated the visual effects with the live-action footage. It never felt like the effects were taking over or overshadowing the characters or the story."
      },
      "message_uuid": {
        "S": "1cc31b8f-4dd5-45c4-a76a-89551dc64739"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:22:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Bayko"
      },
      "user_handle": {
        "S": "bayko"
      },
      "user_uuid": {
        "S": "59e8e5e9-288b-4422-80d1-66eadd0fa162"
      }
    },
    {
      "message": {
        "S": "Yes, I was really blown away by the level of detail and realism in the effects. The ships looked so sleek and futuristic, and the space battles were really intense and exciting."
      },
      "message_uuid": {
        "S": "5fe2e5a5-9bca-4cb2-a363-06f05039560c"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:21:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Brown"
      },
      "user_handle": {
        "S": "andrewbrown"
      },
      "user_uuid": {
        "S": "3a28a20f-964f-42fe-8356-79804e4f753d"
      }
    },
    {
      "message": {
        "S": "I thought the special effects in Babylon 5 were really impressive, especially for a show that aired in the 90s. The use of CGI to create the spaceships and other sci-fi elements was really innovative for its time."
      },
      "message_uuid": {
        "S": "a70206c1-2f59-4dc4-a3c1-97fe06daefe0"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:20:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Bayko"
      },
      "user_handle": {
        "S": "bayko"
      },
      "user_uuid": {
        "S": "59e8e5e9-288b-4422-80d1-66eadd0fa162"
      }
    },
    {
      "message": {
        "S": "One thing that really stands out about Babylon 5 is the quality of the special effects. What did you think about the show's use of CGI and other visual effects?"
      },
      "message_uuid": {
        "S": "1b43d373-30bd-4457-998a-32f26da24adf"
      },
      "pk": {
        "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "sk": {
        "S": "2023-03-23T00:19:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Brown"
      },
      "user_handle": {
        "S": "andrewbrown"
      },
      "user_uuid": {
        "S": "3a28a20f-964f-42fe-8356-79804e4f753d"
      }
    }
  ],
  "LastEvaluatedKey": {
    "pk": {
      "S": "MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
    },
    "sk": {
      "S": "2023-03-23T00:19:12.482495+00:00"
    }
  },
  "ResponseMetadata": {
    "HTTPHeaders": {
      "content-type": "application/x-amz-json-1.0",
      "date": "Wed, 22 Mar 2023 22:59:12 GMT",
      "server": "Jetty(9.4.48.v20220622)",
      "transfer-encoding": "chunked",
      "x-amz-crc32": "2675742161",
      "x-amzn-requestid": "5f364235-e25e-4d16-93b7-a648b40761ec"
    },
    "HTTPStatusCode": 200,
    "RequestId": "5f364235-e25e-4d16-93b7-a648b40761ec",
    "RetryAttempts": 0
  },
  "ScannedCount": 20
}
{
  "CapacityUnits": 1.0,
  "TableName": "cruddur-messages"
}
andrewbrown 2023-03-23 12:19 AM   One thing that really stands out about B...
bayko       2023-03-23 12:20 AM   I thought the special effects in Babylon...
andrewbrown 2023-03-23 12:21 AM   Yes, I was really blown away by the leve...
bayko       2023-03-23 12:22 AM   And I also appreciated the way the show ...
andrewbrown 2023-03-23 12:23 AM   Absolutely. The show had a great balance...
bayko       2023-03-23 12:24 AM   And it's also worth noting the way the s...
andrewbrown 2023-03-23 12:25 AM   Yes, I agree. And it's impressive how th...
bayko       2023-03-23 12:26 AM   Definitely. And it's one of the reasons ...
andrewbrown 2023-03-23 12:27 AM   Agreed. And it's also worth noting the w...
bayko       2023-03-23 12:28 AM   Yes, it definitely had a big impact on t...
andrewbrown 2023-03-23 12:29 AM   Another character I wanted to discuss is...
bayko       2023-03-23 12:30 AM   Zathras was a really unique and memorabl...
andrewbrown 2023-03-23 12:31 AM   Yes, I thought he was a great addition t...
bayko       2023-03-23 12:32 AM   And I appreciated the way the show used ...
andrewbrown 2023-03-23 12:33 AM   Definitely. It was a great way to integr...
bayko       2023-03-23 12:34 AM   Yeah, that was a clever storytelling dev...
andrewbrown 2023-03-23 12:35 AM   I also thought that Zathras was a great ...
bayko       2023-03-23 12:36 AM   Yes, that's a good point. Babylon 5 was ...
andrewbrown 2023-03-23 12:37 AM   And Zathras was just one example of that...
bayko       2023-03-23 12:38 AM   Definitely. I think his character is a g...
```


#### list-conversation

This script will be used for listing conversations

`./backend-flask/bin/ddb/patterns/list-conversations`

```py
#!/usr/bin/env python3

import boto3
import sys
import json
import os

current_path = os.path.dirname(os.path.abspath(__file__))
parent_path = os.path.abspath(os.path.join(current_path, '..', '..', '..'))
sys.path.append(parent_path)
from lib.db import db

attrs = {
  'endpoint_url': 'http://localhost:8000'
}

if len(sys.argv) == 2:
  if "prod" in sys.argv[1]:
    attrs = {}

dynamodb = boto3.client('dynamodb',**attrs)
table_name = 'cruddur-messages'

def get_my_user_uuid():
  sql = """
    SELECT 
      users.uuid
    FROM users
    WHERE
      users.handle =%(handle)s
  """
  uuid = db.query_value(sql,{
    'handle':  'andrewbrown'
  })
  return uuid

my_user_uuid = get_my_user_uuid()
print(f"my-uuid: {my_user_uuid}")

# define the query parameters
query_params = {
  'TableName': table_name,
  'KeyConditionExpression': 'pk = :pk',
  'ExpressionAttributeValues': {
    ':pk': {'S': f"GRP#{my_user_uuid}"}
  },
  'ReturnConsumedCapacity': 'TOTAL'
}

# query the table
response = dynamodb.query(**query_params)

# print the items returned by the query
print(json.dumps(response, sort_keys=True, indent=2))
```

> This script is also using AWS SDK boto3

> https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb/client/query.html

> We need to give execute permissions also to this script by executing `chmod u+x ./bin/ddb/patterns/list-conversations`

For this script, we need to add new functionality to `db.py` file

```py
  def query_value(self,sql,params={}):
    self.print_sql('value',sql,params)
    with self.pool.connection() as conn:
      with conn.cursor() as cur:
        cur.execute(sql,params)
        json = cur.fetchone()
        return json[0]
```

```sh
./bin/ddb/patterns/list-conversations
```

Result
```
 SQL STATEMENT-[value]------

    SELECT 
      users.uuid
    FROM users
    WHERE
      users.handle =%(handle)s
   {'handle': 'andrewbrown'}
my-uuid: 3a28a20f-964f-42fe-8356-79804e4f753d
{
  "ConsumedCapacity": {
    "CapacityUnits": 0.5,
    "TableName": "cruddur-messages"
  },
  "Count": 1,
  "Items": [
    {
      "message": {
        "S": "this is a filler message"
      },
      "message_group_uuid": {
        "S": "5ae290ed-55d1-47a0-bc6d-fe2bc2700399"
      },
      "pk": {
        "S": "GRP#3a28a20f-964f-42fe-8356-79804e4f753d"
      },
      "sk": {
        "S": "2023-03-22T22:55:12.482495+00:00"
      },
      "user_display_name": {
        "S": "Andrew Bayko"
      },
      "user_handle": {
        "S": "bayko"
      },
      "user_uuid": {
        "S": "59e8e5e9-288b-4422-80d1-66eadd0fa162"
      }
    }
  ],
  "ResponseMetadata": {
    "HTTPHeaders": {
      "content-length": "447",
      "content-type": "application/x-amz-json-1.0",
      "date": "Wed, 22 Mar 2023 23:10:55 GMT",
      "server": "Jetty(9.4.48.v20220622)",
      "x-amz-crc32": "2446654267",
      "x-amzn-requestid": "cd6b5c30-4859-4c98-92c9-3aa342012e5a"
    },
    "HTTPStatusCode": 200,
    "RequestId": "cd6b5c30-4859-4c98-92c9-3aa342012e5a",
    "RetryAttempts": 0
  },
  "ScannedCount": 1
}
```


### Update DB bash scripts

In drop script, we will need to check first if database exists before trying to drop it

`./backend-flask/bin/db/drop`

```sh
psql $NO_DB_CONNECTION_URL -c "drop database IF EXISTS cruddur;"
```



### Implement DynamoDB in cruddur


#### Python library for accessing DynamoDB

As we did with postgres, we are going to implement a python library for interacting with DynamoDB using AWS SDK for python (boto3)

`./backend-flask/lib/ddb.py`

```py
import boto3
import sys
from datetime import datetime, timedelta, timezone
import uuid
import os

class Ddb:
  def client():
    endpoint_url = os.getenv("AWS_ENDPOINT_URL")
    if endpoint_url:
      attrs = { 'endpoint_url': endpoint_url }
    else:
      attrs = {}
    dynamodb = boto3.client('dynamodb',**attrs)
    return dynamodb  
  
  def list_message_groups(client,my_user_uuid):
    current_year = datetime.datetime.now().year
    table_name = 'cruddur-messages'
    query_params = {
      'TableName': table_name,
      'KeyConditionExpression': 'pk = :pk AND begins_with(sk,:year)',
      'ScanIndexForward': False,
      'Limit': 20,
      'ExpressionAttributeValues': {
        ':year': {'S': str(current_year) },
        ':pkey': {'S': f"GRP#{my_user_uuid}"}
      }
    }
    print('query-params')
    print(query_params)
    print('client')
    print(client)

    # query the table
    response = client.query(**query_params)
    items = response['Items']

    results = []
    for item in items:
      last_sent_at = item['sk']['S']
      results.append({
        'uuid': item['message_group_uuid']['S'],
        'display_name': item['user_display_name']['S'],
        'handle': item['user_handle']['S'],
        'message': item['message']['S'],
        'created_at': last_sent_at
      })
    return results
```

> This library will contain the DynamoDB client and a function to list the message groups

> https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/dynamodb/client/query.html

> Table name is hardcoded, but a prefix can be added to separating Production from Staging data

 
#### Change in list-conversations pattern

To be synchronized with ddb library, we are going to apply some changes in `list-conversations` script

`./backend-flask/bin/ddb/patterns/

```py
query_params = {
  'TableName': table_name,
  'KeyConditionExpression': 'pk = :pk',
      'KeyConditionExpression': 'pk = :pk AND begins_with(sk,:year)',
  'ScanIndexForward': False,
  'ExpressionAttributeValues': {
    ':year': {'S': str(current_year) },
    ':pk': {'S': f"GRP#{my_user_uuid}"}
  },
  'ReturnConsumedCapacity': 'TOTAL'
```

> In here we are adding 'ScanIndexForward': False


#### Implement DynamoDB in app.py

In `app.py` file we have some methods using 'andrewbrown' as the user handle

```py
@app.route("/api/message_groups", methods=['GET'])
def data_message_groups():
  user_handle  = 'andrewbrown'
  model = MessageGroups.run(user_handle=user_handle)
  if model['errors'] is not None:
    return model['errors'], 422
  else:
    return model['data'], 200
```

We are going to grab the corresponding user handle for the logged user in Cognito, so first we need to implement a way to retrieve current users in Cognito


#### Implement bash script to list users

We will implement a script for listing users in Cognito, using boto3 SDK

`./backend-flask/bin/cognito/list-users`

```py
#!/usr/bin/env python3

import boto3
import os
import json

userpool_id = os.getenv("AWS_COGNITO_USER_POOL_ID")
client = boto3.client('cognito-idp')
params = {
  'UserPoolId': userpool_id,
  'AttributesToGet': [
      'preferred_username',
      'sub'
  ]
}
response = client.list_users(**params)
users = response['Users']

print(json.dumps(users, sort_keys=True, indent=2, default=str))

dict_users = {}
for user in users:
  attrs = user['Attributes']
  sub    = next((a for a in attrs if a["Name"] == 'sub'), None)
  handle = next((a for a in attrs if a["Name"] == 'preferred_username'), None)
  dict_users[handle['Value']] = sub['Value']

print(json.dumps(dict_users, sort_keys=True, indent=2, default=str))
```

> https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/cognito-idp/client/list_users.html

> We need to give execute permissions also to this script by executing `chmod u+x ./bin/cognito/list-users`

> For this script to run, we need to add a new environment variable, passing the corresponding User Pool ID in Cognito (this pool ID is already set in REACT_APP_AWS_USER_POOLS_ID env variable)

```sh
export AWS_COGNITO_USER_POOL_ID="XXX"
gp env AWS_COGNITO_USER_POOL_ID="XXX"
```

```sh
./bin/cognito/list-users
 ```

Result
```
```


#### Implement bash script to update Cognito UserIDs

We need to build a new bash script for updating current users in postgres, and updating the UserID from AWS Cognito

`./backend-flask/bin/db/update_cognito_user_ids`

```py
#!/usr/bin/env python3

import boto3
import os
import sys

print("== db-update-cognito-user-ids")

current_path = os.path.dirname(os.path.abspath(__file__))
parent_path = os.path.abspath(os.path.join(current_path, '..', '..'))
sys.path.append(parent_path)
from lib.db import db

def update_users_with_cognito_user_id(handle,sub):
  sql = """
    UPDATE public.users
    SET cognito_user_id = %(sub)s
    WHERE
      users.handle = %(handle)s;
  """
  db.query_commit(sql,{
    'handle' : handle,
    'sub' : sub
  })

def get_cognito_user_ids():
  userpool_id = os.getenv("AWS_COGNITO_USER_POOL_ID")
  client = boto3.client('cognito-idp')
  params = {
    'UserPoolId': userpool_id,
    'AttributesToGet': [
        'preferred_username',
        'sub'
    ]
  }
  response = client.list_users(**params)
  users = response['Users']
  dict_users = {}
  for user in users:
    attrs = user['Attributes']
    sub    = next((a for a in attrs if a["Name"] == 'sub'), None)
    handle = next((a for a in attrs if a["Name"] == 'preferred_username'), None)
    dict_users[handle['Value']] = sub['Value']
  return dict_users


users = get_cognito_user_ids()

for handle, sub in users.items():
  print('----',handle,sub)
  update_users_with_cognito_user_id(
    handle=handle,
    sub=sub
  )
```

> This script is also using AWS SDK boto3

> https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/cognito-idp/client/list_users.html

> We need to give execute permissions also to this script by executing `chmod u+x ./bin/db/update_cognito_user_ids`

```sh
./bin/db/update_cognito_user_ids
```

Result
```
```


#### Add update_cognito_user_ids to setup script

We need to add the previous script to setup script initialization, so it can be called while initializing db

`./backend-flask/bin/db/setup`

```sh
source "$bin_path/db/update_cognito_user_ids"
```

```sh
./bin/db/setup
```

Result
```
```


#### Implement ddb in '/api/message_groups' endpoint

Replace current implementation with the one accessing ddb

`./backend-flask/app.py`

```py
@app.route("/api/message_groups", methods=['GET'])
def data_message_groups():
  #user_handle  = 'andrewbrown'
  #model = MessageGroups.run(user_handle=user_handle)
  #if model['errors'] is not None:
  #  return model['errors'], 422
  #else:
  #  return model['data'], 200
  access_token = extract_access_token(request.headers)
  try:
    claims = cognito_jwt_token.verify(access_token)
    # authenticated request
    app.logger.debug("authenticated")
    app.logger.debug(claims)
    cognito_user_id = claims['sub']
    model = MessageGroups.run(cognito_user_id=cognito_user_id)
    if model['errors'] is not None:
      return model['errors'], 422
    else:
      return model['data'], 200
  except TokenVerifyError as e:
    # unauthenticated request
    app.logger.debug(e)
    return {}, 401
```

We will need to change also `MessageGroups` implementation, replacing the hardcoded data with information from DynamoDB

`./backend-flask/services/message_groups.py`

```py
from datetime import datetime, timedelta, timezone

from lib.ddb import Ddb
from lib.db import db

class MessageGroups:
  #def run(user_handle):
  def run(cognito_user_id):
    model = {
      'errors': None,
      'data': None
    }

    #now = datetime.now(timezone.utc).astimezone()
    #results = [
    #  {
    #    'uuid': '24b95582-9e7b-4e0a-9ad1-639773ab7552',
    #    'display_name': 'Andrew Brown',
    #    'handle':  'andrewbrown',
    #    'created_at': now.isoformat()
    #  },
    #  {
    #    'uuid': '417c360e-c4e6-4fce-873b-d2d71469b4ac',
    #    'display_name': 'Worf',
    #    'handle':  'worf',
    #    'created_at': now.isoformat()
    #}]
    #model['data'] = results
    sql = db.template('users','uuid_from_cognito_user_id')
    my_user_uuid = db.query_value(sql,{
      'cognito_user_id': cognito_user_id
    })

    print(f"UUID: {my_user_uuid}")

    ddb = Ddb.client()
    data = Ddb.list_message_groups(ddb, my_user_uuid)
    print("list_message_groups:",data)

    model['data'] = data
    return model
```

The new implementation requires a new sql query file to be created as well

`./backend-flask/db/sql/users/uuid_from_cognito_user_id.sql`

```sql
SELECT
  users.uuid
FROM public.users
WHERE 
  users.cognito_user_id = %(cognito_user_id)s
LIMIT 1
```

Another change we need to implement is in frontend code, in which we need to pass the token to the backend, as we already did in `HomeFeedPage.js`

In this case, we need to add it to `frontend-react-js/src/pages/MessageGroupsPage.js`

```js
// [TODO] Authenication
//import Cookies from 'js-cookie'

  const loadData = async () => {
    try {
      const backend_url = `${process.env.REACT_APP_BACKEND_URL}/api/message_groups`
      const res = await fetch(backend_url, {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`
        },
        method: "GET"
      });
      let resJson = await res.json();
      if (res.status === 200) {
        setMessageGroups(resJson)
      } else {
        console.log(res)
      }
    } catch (err) {
      console.log(err);
    }
  };  
```

> We also comment out `import Cookies from 'js-cookie'` because we are not using cookies authentication any more

We are going to make the same change in `frontend-react-js/src/pages/MessageGroupPage.js`

```js
// [TODO] Authenication
//import Cookies from 'js-cookie'

  const loadMessageGroupsData = async () => {
    try {
      const backend_url = `${process.env.REACT_APP_BACKEND_URL}/api/message_groups`
      const res = await fetch(backend_url, {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`
        },
        method: "GET"
      });
      let resJson = await res.json();
      if (res.status === 200) {
        setMessageGroups(resJson)
      } else {
        console.log(res)
      }
    } catch (err) {
      console.log(err);
    }
  };  
```

Another place we need to make the same change is in `frontend-react-js/src/components/MessageForm.js` (in this case there is already some headers passing to backend)

```js
  const onsubmit = async (event) => {
    event.preventDefault();
    try {
      const backend_url = `${process.env.REACT_APP_BACKEND_URL}/api/messages`
      console.log('onsubmit payload', message)
      const res = await fetch(backend_url, {
        method: "POST",
        headers: {
          'Authorization': `Bearer ${localStorage.getItem("access_token")}`,
          'Accept': 'application/json',
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          message: message,
          user_receiver_handle: params.handle
        }),
      });
      let data = await res.json();
      if (res.status === 200) {
        props.setMessages(current => [...current,data]);
      } else {
        console.log(res)
      }
    } catch (err) {
      console.log(err);
    }
  }
```

Now we can check it from the application, trying to retrieve message groups using cruddur

![](./assets/week-5/XXX)

> Remember to run the script to update cognito_user_ids in db

> Some thing I've learned from Andrew: Ctrl-P in Gitpod VS allows to search for a file 


Just for simplicity and to make a better code, we will extract the code used for validating the token in a separate file

`frontend-react-js/src/lib/CheckAuth.js`

```js
import { Auth } from 'aws-amplify';

const checkAuth = async (setUser) => {
  Auth.currentAuthenticatedUser({
    // Optional, By default is false. 
    // If set to true, this call will send a 
    // request to Cognito to get the latest user data
    bypassCache: false 
  })
  .then((user) => {
    console.log('user',user);
    return Auth.currentAuthenticatedUser()
  }).then((cognito_user) => {
      setUser({
        display_name: cognito_user.attributes.name,
        handle: cognito_user.attributes.preferred_username
      })
  })
  .catch((err) => console.log(err));
};

export default checkAuth;
```

So, now we can remove this function (and Auth import) from `HomeFeedPage.js`

`frontend-react-js/src/pages/HomeFeedPage.js`

```js
// Authentication
//import Cookies from 'js-cookie'
//import { Auth } from 'aws-amplify';

  /*
  const checkAuth = async () => {
    Auth.currentAuthenticatedUser({
      // Optional, By default is false. 
      // If set to true, this call will send a 
      // request to Cognito to get the latest user data
      bypassCache: false 
    })
    .then((user) => {
      console.log('user',user);
      return Auth.currentAuthenticatedUser()
    }).then((cognito_user) => {
        setUser({
          display_name: cognito_user.attributes.name,
          handle: cognito_user.attributes.preferred_username
        })
    })
    .catch((err) => console.log(err));
  };
  */
```

Now we should add the new implementation in `HomeFeedPage.js`, importing the new library and adding the setUser parameter to checkAuth function

```js
import checkAuth from '../lib/CheckAuth';

checkAuth(setUser);
```

It's time to add CheckAuth library to the other pages

`MessageGroupsPage.js`

```js
import checkAuth from '../lib/CheckAuth';

  /*
  const checkAuth = async () => {
    console.log('checkAuth')
    // [TODO] Authenication
    if (Cookies.get('user.logged_in')) {
      setUser({
        display_name: Cookies.get('user.name'),
        handle: Cookies.get('user.username')
      })
    }
  };
  */

checkAuth(setUser);
```

`MessageGroupPage.js`

```js
import checkAuth from '../lib/CheckAuth';

  /*
  const checkAuth = async () => {
    console.log('checkAuth')
    // [TODO] Authenication
    if (Cookies.get('user.logged_in')) {
      setUser({
        display_name: Cookies.get('user.name'),
        handle: Cookies.get('user.username')
      })
    }
  };
  */

checkAuth(setUser);
```

Another change we need to implement in `MessageGroupPage.js` is in loadMessageGroupData function:

```js
  const loadMessageGroupData = async () => {
    try {
      const handle = `@${params.handle}`;
      //const backend_url = `${process.env.REACT_APP_BACKEND_URL}/api/messages/${handle}`
      const backend_url = `${process.env.REACT_APP_BACKEND_URL}/api/messages/${params.message_group_uuid}`
      const res = await fetch(backend_url, {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`
        },
        method: "GET"
      });
      let resJson = await res.json();
      if (res.status === 200) {
        setMessages(resJson)
      } else {
        console.log(res)
      }
    } catch (err) {
      console.log(err);
    }
  };  
```

> We need to add Authorization header and change the backend_url, using the message_group_uuid instead of handle

Also, in `ddb.py` we will add the filter for current year

```py
  def list_message_groups(client,my_user_uuid):
    year = str(datetime.now().year)
    table_name = 'cruddur-messages'
    query_params = {
      'TableName': table_name,
      'KeyConditionExpression': 'pk = :pk AND begins_with(sk,:year)',
      'ScanIndexForward': False,
      'Limit': 20,
      'ExpressionAttributeValues': {
        ':year': {'S': year },
        ':pk': {'S': f"GRP#{my_user_uuid}"}
      }
    }
```

In `frontend-react-js/src/components/MessageGroupItem.js` we need to change param name

```py
  const classes = () => {
    let classes = ["message_group_item"];
    if (params.message_group_uuid == props.message_group.uuid){
      classes.push('active')
    }
    return classes.join(' ');
  }

  return (
    <Link className={classes()} to={`/messages/`+props.message_group.uuid}>
```

Now we need to change implementation for "/api/messages/@<string:handle>" endpoint in `app.py`, adding authentication and the correct parameters

```py
#@app.route("/api/messages/@<string:handle>", methods=['GET'])
#def data_messages(handle):
#  user_sender_handle = 'andrewbrown'
#  user_receiver_handle = request.args.get('user_reciever_handle')

#  model = Messages.run(user_sender_handle=user_sender_handle, user_receiver_handle=user_receiver_handle)
#  if model['errors'] is not None:
#    return model['errors'], 422
#  else:
#    return model['data'], 200
#  return
@app.route("/api/messages/<string:message_group_uuid>", methods=['GET'])
def data_messages(message_group_uuid):
  access_token = extract_access_token(request.headers)
  try:
    claims = cognito_jwt_token.verify(access_token)
    # authenticated request
    app.logger.debug("authenticated")
    app.logger.debug(claims)
    cognito_user_id = claims['sub']
    model = Messages.run(
        cognito_user_id=cognito_user_id,
        message_group_uuid=message_group_uuid
      )
    if model['errors'] is not None:
      return model['errors'], 422
    else:
      return model['data'], 200
  except TokenVerifyError as e:
    # unauthenticated request
    app.logger.debug(e)
    return {}, 401
```

This change requires changing also the implementation in `messages.py`, for using DynamoDB and not returning hardcoded data

```py
from datetime import datetime, timedelta, timezone
from lib.ddb import Ddb
from lib.db import db

class Messages:
  #def run(user_sender_handle, user_receiver_handle):
  def run(message_group_uuid, cognito_user_id):
    model = {
      'errors': None,
      'data': None
    }

    #now = datetime.now(timezone.utc).astimezone()

    #results = [
    #  {
    #    'uuid': '4e81c06a-db0f-4281-b4cc-98208537772a' ,
    #    'display_name': 'Andrew Brown',
    #    'handle':  'andrewbrown',
    #    'message': 'Cloud is fun!',
    #    'created_at': now.isoformat()
    #  },
    #  {
    #    'uuid': '66e12864-8c26-4c3a-9658-95a10f8fea67',
    #    'display_name': 'Andrew Brown',
    #    'handle':  'andrewbrown',
    #    'message': 'This platform is great!',
    #    'created_at': now.isoformat()
    #}]
    #model['data'] = results
    
    sql = db.template('users', 'uuid_from_cognito_user_id')
    my_user_uuid = db.query_value(sql,{
      'cognito_user_id': cognito_user_id
    })

    print(f"UUID: {my_user_uuid}")

    ddb = Ddb.client()
    data = Ddb.list_messages(ddb, message_group_uuid)
    print("list_messages")
    print(data)
    model['data'] = data

    return model
```

A new function should be implemented in `ddb.py` for listing the messages for a given message_group_uuid

```py
  def list_messages(client,message_group_uuid):
    year = str(datetime.now().year)
    table_name = 'cruddur-messages'
    query_params = {
      'TableName': table_name,
      'KeyConditionExpression': 'pk = :pk AND begins_with(sk,:year)',
      'ScanIndexForward': False,
      'Limit': 20,
      'ExpressionAttributeValues': {
        ':year': {'S': year },
        ':pk': {'S': f"MSG#{message_group_uuid}"}
      }
    }

    response = client.query(**query_params)
    items = response['Items']
    items.reverse()
    results = []
    for item in items:
      created_at = item['sk']['S']
      results.append({
        'uuid': item['message_uuid']['S'],
        'display_name': item['user_display_name']['S'],
        'handle': item['user_handle']['S'],
        'message': item['message']['S'],
        'created_at': created_at
      })
    return results
```

In `MessageForm.js` we need to change the way of passing parameters to backend (depending if we need to create a new message group or update an existing one)

```py
//import { useParams } from 'react-router-dom';
import { json, useParams } from 'react-router-dom';

  const onsubmit = async (event) => {
    event.preventDefault();
    try {
      const backend_url = `${process.env.REACT_APP_BACKEND_URL}/api/messages`
      console.log('onsubmit payload', message)
      let json = { 'message': message }
      if (params.handle) {
        json.handle = params.handle
      } else {
        json.message_group_uuid = params.message_group_uuid
      }

      const res = await fetch(backend_url, {
        method: "POST",
        headers: {
          'Authorization': `Bearer ${localStorage.getItem("access_token")}`,
          'Accept': 'application/json',
          'Content-Type': 'application/json'
        },
        //body: JSON.stringify({
        //  message: message,
        //  user_receiver_handle: params.handle
        //}),
        body: JSON.stringify(json)
      });
      let data = await res.json();
      if (res.status === 200) {
        //props.setMessages(current => [...current,data]);
        console.log('data:',data)
        if (data.message_group_uuid) {
          console.log('redirect to message group')
          window.location.href = `/messages/${data.message_group_uuid}`
        } else {
          props.setMessages(current => [...current,data]);
        }        
      } else {
        console.log(res)
      }
    } catch (err) {
      console.log(err);
    }
  }
```

This change makes us review the backend endpoint "/api/messages" in `app.py` for creating messages

```py
@app.route("/api/messages", methods=['POST','OPTIONS'])
@cross_origin()
def data_create_message():
  #user_sender_handle = 'andrewbrown'
  #user_receiver_handle = request.json['user_receiver_handle']
  message_group_uuid   = request.json.get('message_group_uuid',None)
  user_receiver_handle = request.json.get('handle',None)  
  message = request.json['message']

  #model = CreateMessage.run(message=message,user_sender_handle=user_sender_handle,user_receiver_handle=user_receiver_handle)
  #if model['errors'] is not None:
  #  return model['errors'], 422
  #else:
  #  return model['data'], 200
  #return

  access_token = extract_access_token(request.headers)
  try:
    claims = cognito_jwt_token.verify(access_token)
    # authenticated request
    app.logger.debug("authenticated")
    app.logger.debug(claims)
    cognito_user_id = claims['sub']
    if message_group_uuid == None:
      # Create for the first time
      model = CreateMessage.run(
        mode="create",
        message=message,
        cognito_user_id=cognito_user_id,
        user_receiver_handle=user_receiver_handle
      )
    else:
      # Push onto existing Message Group
      model = CreateMessage.run(
        mode="update",
        message=message,
        message_group_uuid=message_group_uuid,
        cognito_user_id=cognito_user_id
      )
    if model['errors'] is not None:
      return model['errors'], 422
    else:
      return model['data'], 200
  except TokenVerifyError as e:
    # unauthenticated request
    app.logger.debug(e)
    return {}, 401
```

Then we need to check `create_message.py`, for adding DynamoDB implementation instead of hardcoded one

```py
#import uuid
from datetime import datetime, timedelta, timezone

from lib.db import db
from lib.ddb import Ddb

class CreateMessage:
  #def run(message, user_sender_handle, user_receiver_handle):
  # mode indicates if we want to create a new message_group or using an existing one
  def run(mode, message, cognito_user_id, message_group_uuid=None, user_receiver_handle=None):  
    model = {
      'errors': None,
      'data': None
    }
    #if user_sender_handle == None or len(user_sender_handle) < 1:
    #  model['errors'] = ['user_sender_handle_blank']

    #if user_receiver_handle == None or len(user_receiver_handle) < 1:
    #  model['errors'] = ['user_reciever_handle_blank']

    if (mode == "update"):
      if message_group_uuid == None or len(message_group_uuid) < 1:
        model['errors'] = ['message_group_uuid_blank']


    if cognito_user_id == None or len(cognito_user_id) < 1:
      model['errors'] = ['cognito_user_id_blank']

    if (mode == "create"):
      if user_receiver_handle == None or len(user_receiver_handle) < 1:
        model['errors'] = ['user_reciever_handle_blank']

    if message == None or len(message) < 1:
      model['errors'] = ['message_blank'] 
    elif len(message) > 1024:
      model['errors'] = ['message_exceed_max_chars'] 

    if model['errors']:
      # return what we provided
      model['data'] = {
        'display_name': 'Andrew Brown',
        'handle':  user_sender_handle,
        'message': message
      }
    else:
      #now = datetime.now(timezone.utc).astimezone()
      #model['data'] = {
      #  'uuid': uuid.uuid4(),
      #  'display_name': 'Andrew Brown',
      #  'handle':  user_sender_handle,
      #  'message': message,
      #  'created_at': now.isoformat()
      #}
    
      sql = db.template('users','create_message_users')

      if user_receiver_handle == None:
        rev_handle = ''
      else:
        rev_handle = user_receiver_handle
      users = db.query_array_json(sql,{
        'cognito_user_id': cognito_user_id,
        'user_receiver_handle': rev_handle
      })
      print("USERS =-=-=-=-==")
      print(users)

      my_user    = next((item for item in users if item["kind"] == 'sender'), None)
      other_user = next((item for item in users if item["kind"] == 'recv')  , None)

      print("USERS=[my-user]==")
      print(my_user)
      print("USERS=[other-user]==")
      print(other_user)

      ddb = Ddb.client()

      if (mode == "update"):
        data = Ddb.create_message(
          client=ddb,
          message_group_uuid=message_group_uuid,
          message=message,
          my_user_uuid=my_user['uuid'],
          my_user_display_name=my_user['display_name'],
          my_user_handle=my_user['handle']
        )
      elif (mode == "create"):
        data = Ddb.create_message_group(
          client=ddb,
          message=message,
          my_user_uuid=my_user['uuid'],
          my_user_display_name=my_user['display_name'],
          my_user_handle=my_user['handle'],
          other_user_uuid=other_user['uuid'],
          other_user_display_name=other_user['display_name'],
          other_user_handle=other_user['handle']
        )
      model['data'] = data    
    
    return model
```

These changes references a new template sql script file that we should be adding as well `create_message_users.sql`

```sql
SELECT 
  users.uuid,
  users.display_name,
  users.handle,
  CASE users.cognito_user_id = %(cognito_user_id)s
  WHEN TRUE THEN
    'sender'
  WHEN FALSE THEN
    'recv'
  ELSE
    'other'
  END as kind
FROM public.users
WHERE
  users.cognito_user_id = %(cognito_user_id)s
  OR 
  users.handle = %(user_receiver_handle)s
```

Also we need some new functions in `ddb.py` for creating new message groups and new messages

```py
  def create_message(client,message_group_uuid, message, my_user_uuid, my_user_display_name, my_user_handle):
    now = datetime.now(timezone.utc).astimezone().isoformat()
    created_at = now
    message_uuid = str(uuid.uuid4())

    record = {
      'pk':   {'S': f"MSG#{message_group_uuid}"},
      'sk':   {'S': created_at },
      'message': {'S': message},
      'message_uuid': {'S': message_uuid},
      'user_uuid': {'S': my_user_uuid},
      'user_display_name': {'S': my_user_display_name},
      'user_handle': {'S': my_user_handle}
    }
    # insert the record into the table
    table_name = 'cruddur-messages'
    response = client.put_item(
      TableName=table_name,
      Item=record
    )
    # print the response
    print(response)
    return {
      'message_group_uuid': message_group_uuid,
      'uuid': my_user_uuid,
      'display_name': my_user_display_name,
      'handle':  my_user_handle,
      'message': message,
      'created_at': created_at
    }
```

```py
  def create_message_group(client, message,my_user_uuid, my_user_display_name, my_user_handle, other_user_uuid, other_user_display_name, other_user_handle):
    print('== create_message_group.1')
    table_name = 'cruddur-messages'

    message_group_uuid = str(uuid.uuid4())
    message_uuid = str(uuid.uuid4())
    now = datetime.now(timezone.utc).astimezone().isoformat()
    last_message_at = now
    created_at = now
    print('== create_message_group.2')

    my_message_group = {
      'pk': {'S': f"GRP#{my_user_uuid}"},
      'sk': {'S': last_message_at},
      'message_group_uuid': {'S': message_group_uuid},
      'message': {'S': message},
      'user_uuid': {'S': other_user_uuid},
      'user_display_name': {'S': other_user_display_name},
      'user_handle':  {'S': other_user_handle}
    }

    print('== create_message_group.3')
    other_message_group = {
      'pk': {'S': f"GRP#{other_user_uuid}"},
      'sk': {'S': last_message_at},
      'message_group_uuid': {'S': message_group_uuid},
      'message': {'S': message},
      'user_uuid': {'S': my_user_uuid},
      'user_display_name': {'S': my_user_display_name},
      'user_handle':  {'S': my_user_handle}
    }

    print('== create_message_group.4')
    message = {
      'pk':   {'S': f"MSG#{message_group_uuid}"},
      'sk':   {'S': created_at },
      'message': {'S': message},
      'message_uuid': {'S': message_uuid},
      'user_uuid': {'S': my_user_uuid},
      'user_display_name': {'S': my_user_display_name},
      'user_handle': {'S': my_user_handle}
    }

    items = {
      table_name: [
        {'PutRequest': {'Item': my_message_group}},
        {'PutRequest': {'Item': other_message_group}},
        {'PutRequest': {'Item': message}}
      ]
    }

    try:
      print('== create_message_group.try')
      # Begin the transaction
      response = client.batch_write_item(RequestItems=items)
      return {
        'message_group_uuid': message_group_uuid
      }
    except botocore.exceptions.ClientError as e:
      print('== create_message_group.error')
      print(e)
```

This conclude with the pattern in which we add a new message to an existing conversation. 

Now we need to make the corresponding changes to create a new fresh conversation.

In frontend, in `app.js` we need to add a new path (and also we are updating the path for MessageGroupPage)

We also need to add the new import

```js
import MessageGroupNewPage from './pages/MessageGroupNewPage';

{
    path: "/messages/new/:handle",
    element: <MessageGroupNewPage />
  },
  {
    path: "/messages/:message_group_uuid",
    element: <MessageGroupPage />
  },
```

We are going to add this new page to `pages` folder as well (`MessageGroupNewPage.js`)

```js
import './MessageGroupPage.css';
import React from "react";
import { useParams } from 'react-router-dom';

import DesktopNavigation  from '../components/DesktopNavigation';
import MessageGroupFeed from '../components/MessageGroupFeed';
import MessagesFeed from '../components/MessageFeed';
import MessagesForm from '../components/MessageForm';
import checkAuth from '../lib/CheckAuth';

export default function MessageGroupPage() {
  const [otherUser, setOtherUser] = React.useState([]);
  const [messageGroups, setMessageGroups] = React.useState([]);
  const [messages, setMessages] = React.useState([]);
  const [popped, setPopped] = React.useState([]);
  const [user, setUser] = React.useState(null);
  const dataFetchedRef = React.useRef(false);
  const params = useParams();

  const loadUserShortData = async () => {
    try {
      const backend_url = `${process.env.REACT_APP_BACKEND_URL}/api/users/@${params.handle}/short`
      const res = await fetch(backend_url, {
        headers: {
            Authorization: `Bearer ${localStorage.getItem("access_token")}`
        },
        method: "GET"
      });
      let resJson = await res.json();
      if (res.status === 200) {
        console.log('other user:',resJson)
        setOtherUser(resJson)
      } else {
        console.log(res)
      }
    } catch (err) {
      console.log(err);
    }
  };  

  const loadMessageGroupsData = async () => {
    try {
      const backend_url = `${process.env.REACT_APP_BACKEND_URL}/api/message_groups`
      const res = await fetch(backend_url, {
        headers: {
          Authorization: `Bearer ${localStorage.getItem("access_token")}`
        },
        method: "GET"
      });
      let resJson = await res.json();
      if (res.status === 200) {
        setMessageGroups(resJson)
      } else {
        console.log(res)
      }
    } catch (err) {
      console.log(err);
    }
  };  

  React.useEffect(()=>{
    //prevents double call
    if (dataFetchedRef.current) return;
    dataFetchedRef.current = true;

    loadMessageGroupsData();
    loadUserShortData();
    checkAuth(setUser);
  }, [])
  return (
    <article>
      <DesktopNavigation user={user} active={'home'} setPopped={setPopped} />
      <section className='message_groups'>
        <MessageGroupFeed otherUser={otherUser} message_groups={messageGroups} />
      </section>
      <div className='content messages'>
        <MessagesFeed messages={messages} />
        <MessagesForm setMessages={setMessages} />
      </div>
    </article>
  );
}
```

For testing it, we will need to add another user to our seed sql script (`seed.sql`)

```sql
INSERT INTO public.users (display_name, handle, email, cognito_user_id)
VALUES
  ('Andrew Brown', 'andrewbrown', 'andrew@exampro.co', 'MOCK'),
  ('Andrew Bayko', 'bayko', 'bayko@exampro.co', 'MOCK'),
  ('Londo Mollari', 'londo', 'lmollari@centari.com', 'MOCK');
```

> For the sake of testing it, we will insert this new user manually into db

In `app.py` we will make some changes, such as adding the new endpoint for the new created page

```py
from services.users_short import *

@app.route("/api/users/@<string:handle>/short", methods=['GET'])
def data_users_short(handle):
  data = UsersShort.run(handle)
  return data, 200
```

We will need a new service for returning this data (`users_short.py`)

```py
from lib.db import db

class UsersShort:
  def run(handle):
    sql = db.template('users','short')
    results = db.query_object_json(sql,{
      'handle': handle
    })
    return results
```

> As we are grabbing public information we don't need to protect this endpoint

This new service will require a new sql template file, so we need to create it (`short.sql`)

```sql
SELECT
  users.uuid,
  users.handle,
  users.display_name
FROM public.users
WHERE 
  users.handle = %(handle)s
```

> Now the user short endpoint should work

We need to create a new component `MessageGroupNewItem.js`

```js
import './MessageGroupItem.css';
import { Link } from "react-router-dom";

export default function MessageGroupNewItem(props) {
  return (

    <Link className='message_group_item active' to={`/messages/new/`+props.user.handle}>
      <div className='message_group_avatar'></div>
      <div className='message_content'>
        <div classsName='message_group_meta'>
          <div className='message_group_identity'>
            <div className='display_name'>{props.user.display_name}</div>
            <div className="handle">@{props.user.handle}</div>
          </div>{/* activity_identity */}
        </div>{/* message_meta */}
      </div>{/* message_content */}
    </Link>
  );
}
```

We also need to update `MessageGroupFeed.js` for adding this new component

```js
import './MessageGroupFeed.css';
import MessageGroupItem from './MessageGroupItem';

//add
import MessageGroupNewItem from './MessageGroupNewItem'; 

export default function MessageGroupFeed(props) {
  
  // add
  let message_group_new_item;
  if (props.otherUser) {
    message_group_new_item = <MessageGroupNewItem user={props.otherUser} />
  }

  return (
    <div className='message_group_feed'>
      <div className='message_group_feed_heading'>
        <div className='title'>Messages</div>
      </div>
      <div className='message_group_feed_collection'>
        {message_group_new_item}
        {props.message_groups.map(message_group => {
        return  <MessageGroupItem key={message_group.uuid} message_group={message_group} />
        })}
      </div>
    </div>
  );
}

Now we need to change something in `MessageForm.js`, for adding the redirect (we have already done that change)

```js
      if (res.status === 200) {
        //props.setMessages(current => [...current,data]);
        console.log('data:',data)
        if (data.message_group_uuid) {
          console.log('redirect to message group')
          window.location.href = `/messages/${data.message_group_uuid}`
        } else {
          props.setMessages(current => [...current,data]);
        }        
      } else {
        console.log(res)
      }
```

Now, in `create_message.py`, we can uncomment the section for creating a new message group

```py
      elif (mode == "create"):
        data = Ddb.create_message_group(
          client=ddb,
          message=message,
          my_user_uuid=my_user['uuid'],
          my_user_display_name=my_user['display_name'],
          my_user_handle=my_user['handle'],
          other_user_uuid=other_user['uuid'],
          other_user_display_name=other_user['display_name'],
          other_user_handle=other_user['handle']
        )
```

In `ddb.py` we need to create the function create_message_group

```py
  def create_message_group(client, message,my_user_uuid, my_user_display_name, my_user_handle, other_user_uuid, other_user_display_name, other_user_handle):
    print('== create_message_group.1')
    table_name = 'cruddur-messages'

    message_group_uuid = str(uuid.uuid4())
    message_uuid = str(uuid.uuid4())
    now = datetime.now(timezone.utc).astimezone().isoformat()
    last_message_at = now
    created_at = now
    print('== create_message_group.2')

    my_message_group = {
      'pk': {'S': f"GRP#{my_user_uuid}"},
      'sk': {'S': last_message_at},
      'message_group_uuid': {'S': message_group_uuid},
      'message': {'S': message},
      'user_uuid': {'S': other_user_uuid},
      'user_display_name': {'S': other_user_display_name},
      'user_handle':  {'S': other_user_handle}
    }

    print('== create_message_group.3')
    other_message_group = {
      'pk': {'S': f"GRP#{other_user_uuid}"},
      'sk': {'S': last_message_at},
      'message_group_uuid': {'S': message_group_uuid},
      'message': {'S': message},
      'user_uuid': {'S': my_user_uuid},
      'user_display_name': {'S': my_user_display_name},
      'user_handle':  {'S': my_user_handle}
    }

    print('== create_message_group.4')
    message = {
      'pk':   {'S': f"MSG#{message_group_uuid}"},
      'sk':   {'S': created_at },
      'message': {'S': message},
      'message_uuid': {'S': message_uuid},
      'user_uuid': {'S': my_user_uuid},
      'user_display_name': {'S': my_user_display_name},
      'user_handle': {'S': my_user_handle}
    }

    items = {
      table_name: [
        {'PutRequest': {'Item': my_message_group}},
        {'PutRequest': {'Item': other_message_group}},
        {'PutRequest': {'Item': message}}
      ]
    }

    try:
      print('== create_message_group.try')
      # Begin the transaction
      response = client.batch_write_item(RequestItems=items)
      return {
        'message_group_uuid': message_group_uuid
      }
    except botocore.exceptions.ClientError as e:
      print('== create_message_group.error')
      print(e)
```

Also, we need to make sure to include this import

```py
import botocore.exceptions
```

After this, we can successfully create a direct message to a user

### Setting DynamoDB stream to update message

We are going to execute `./bin/ddb/schema-load prod` to create table in AWS DynamoDB

```sh
./bin/ddb/schema-load prod`
```

After doing that we are going to turn on DynamoDB stream, by using AWS console

![](./assets/week-5/XXX)

> View type = New image

We are going to create a Gateway endpoint, and there is no additional charge for using that

Let's go into AWS console, VPC, Endpoints, Create endpoint

- Name tag = ddb-cruddur

- Service category = AWS services

- Services = DynamoDB (com.amazonaws.ca-central-1.dynamodb)

- VPC = select the existing default VPC

- Route tables = check the existing one

- Policy = Full access

Once the endpoint is created, let's create the lambda in AWS console (save the code also in `journal/aws/lambdas/cruddur-messaging-stream.py`)

- Function name = cruddur-messaging-stream

- Runtime = Python 3.9

- Architecture  = x86_64

> We need to grant the lambda IAM role permission to read the DynamoDB stream event

> AWSLambdaInvocation-DynamoDB





```py
import json
import boto3
from boto3.dynamodb.conditions import Key, Attr

dynamodb = boto3.resource(
 'dynamodb',
 region_name='ca-central-1',
 endpoint_url="http://dynamodb.ca-central-1.amazonaws.com"
)

def lambda_handler(event, context):
  print('event-data',event)

  eventName = event['Records'][0]['eventName']
  if (eventName == 'REMOVE'):
    print("skip REMOVE event")
    return
  pk = event['Records'][0]['dynamodb']['Keys']['pk']['S']
  sk = event['Records'][0]['dynamodb']['Keys']['sk']['S']
  if pk.startswith('MSG#'):
    group_uuid = pk.replace("MSG#","")
    message = event['Records'][0]['dynamodb']['NewImage']['message']['S']
    print("GRUP ===>",group_uuid,message)

    table_name = 'cruddur-messages'
    index_name = 'message-group-sk-index'
    table = dynamodb.Table(table_name)
    data = table.query(
      IndexName=index_name,
      KeyConditionExpression=Key('message_group_uuid').eq(group_uuid)
    )
    print("RESP ===>",data['Items'])

    # recreate the message group rows with new SK value
    for i in data['Items']:
      delete_item = table.delete_item(Key={'pk': i['pk'], 'sk': i['sk']})
      print("DELETE ===>",delete_item)

      response = table.put_item(
        Item={
          'pk': i['pk'],
          'sk': sk,
          'message_group_uuid':i['message_group_uuid'],
          'message':message,
          'user_display_name': i['user_display_name'],
          'user_handle': i['user_handle'],
          'user_uuid': i['user_uuid']
        }
      )
      print("CREATE ===>",response)
```







## Knowledge Challenges

### Security Quiz

Which additional DynamoDB feature can be enabled to improve the response times of your tables?
- DynamoDB Accelerator (DAX) (X)
- DynamoDB Optimizer (DAO)
- NoSQL Accelerator (NAX)
- Mega Accelerator (MAX)

What should you use to ensure any DynamoDB traffic from your web app hosted on AWS is not routed over the internet?
- DynamoDB Proxy
- VPN Gateway
- VPC Gateway
- VPC Endpoint (X)

Which is the best way to secure and manage user access permissions to DynamoDB
- IAM Roles (X)
- Individual IAM User Accounts
- Open all access
- Security Groups

What type of access should be granted to the DynamoDB Accelerator (DAX) IAM role?
- Read Only (X)
- Read and Write
- Full Access
- No Access is required

True or False, best practice dictates using Client side encryption with DynamoDB
- TRUE (X)
- FALSE

## Links

- [AWS Cloud Project Bootcamp – Week 5: Unofficial Homework Guide](https://www.linuxtek.ca/2023/03/19/aws-cloud-project-bootcamp-week-5-unofficial-homework-guide/)

-[Andrew’s OmenKing Repository – Week 5 Notes](https://github.com/omenking/aws-bootcamp-cruddur-2023/blob/week-5/journal/week5.md)

-[Livestream DynamoDB Modelling Diagram](https://lucid.app/lucidchart/8f58a19d-3821-4529-920f-5bb802d6c6a3/edit?viewport_loc=2163%2C508%2C2219%2C1161%2C0_0&invitationId=inv_e47bc316-9caa-4aee-940f-161e01e22715)

-[Livestream DynamoDB Data Modelling Excel File](https://docs.google.com/spreadsheets/d/1LrTC_y2X_YBEthFNlnwbo8TgxGlpHV3TCpM6XeVyiGg/edit#gid=0)

-[AWS DynamoDB Documentation](https://docs.aws.amazon.com/dynamodb/?icmpid=docs_homepage_featuredsvcs)

-[Momento Serverless Cache Documentation](https://docs.momentohq.com/getting-started)

-[Alex DeBrie – The DynamoDB Book – Boot Camp Discount in Student Portal](https://www.dynamodbbook.com/)







### Implement Seed Script

https://www.youtube.com/watch?v=pIGi_9E_GwA&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=52

### Implement Scan Script

https://www.youtube.com/watch?v=pIGi_9E_GwA&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=52

### Implement Pattern Scripts for Read and List Conversations

https://www.youtube.com/watch?v=pIGi_9E_GwA&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=52

### Implement Update Cognito ID Script for Postgres Database

https://www.youtube.com/watch?v=dWHOsXiAIBU&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=54

### Implement (Pattern A) Listing Messages in Message Group into Application

https://www.youtube.com/watch?v=dWHOsXiAIBU&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=54

### Implement (Pattern B) Listing Messages Group into Application

https://www.youtube.com/watch?v=dWHOsXiAIBU&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=54

### Implement (Pattern B) Listing Messages Group into Application

https://www.youtube.com/watch?v=dWHOsXiAIBU&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=54

### Implement (Pattern C) Creating a Message for an existing Message Group into Application

https://www.youtube.com/watch?v=dWHOsXiAIBU&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=54

### Implement (Pattern D) Creating a Message for a new Message Group into Application

https://www.youtube.com/watch?v=dWHOsXiAIBU&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=54

### Implement (Pattern E) Updating a Message Group using DynamoDB Streams

https://www.youtube.com/watch?v=zGnzM_YdMJU&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=55

