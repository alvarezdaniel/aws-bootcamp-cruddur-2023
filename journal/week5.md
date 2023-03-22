# Week 5 — DynamoDB and Serverless Caching

## Week 5 instructors

- Andrew Brown [@andrewbrown](https://twitter.com/andrewbrown)
- Shala Warner [@GiftedLane](https://twitter.com/GiftedLane)
- Kirk Kirk? []()

## Class Summary

- 

## Todo Checklist


### Watch lecture



### Enable local DynamoDB container

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

### Restructure db bash scripts

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

### Add DynamoDB bash scripts

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
docker compose  -f "docker-compose.yml" up -d --build dynamodb-local 
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

```


#### get-conversation

This script will be used for getting conversations

`./backend-flask/bin/ddb/patterns/get-conversations`

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

> We need to give execute permissions also to this script by executing `chmod u+x ./bin/ddb/patterns/get-conversations`

```sh
./bin/ddb/patterns/get-conversations
```

Result
```

```







