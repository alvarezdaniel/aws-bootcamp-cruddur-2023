```sh
#docker compose  -f "docker-compose.yml" up -d --build db dynamodb-local
docker compose  -f "docker-compose.yml" up -d

cd backend-flask
./bin/db/setup
#./bin/db/drop
#./bin/db/update_cognito_user_ids

./bin/ddb/schema-load
./bin/ddb/list-tables
#./bin/ddb/drop cruddur-messages

./bin/ddb/seed

./bin/ddb/scan

./bin/ddb/patterns/get-conversation
./bin/ddb/patterns/list-conversations

./bin/cognito/list-users
./bin/db/update_cognito_user_ids
```



{'user_uuid': 'ed6dd963-d34a-4e82-be39-f73189728ba5', 'message_group_uuid': '5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'user_handle': 'dalvarez', 'sk': '2023-04-02T22:02:18.052306+00:00', 'pk': 'GRP#7a703a5c-036c-4064-a263-3a70a35771e1', 'message': 'this is a filler message', 'user_display_name': 'Daniel Alvarez'}
{'user_uuid': '7a703a5c-036c-4064-a263-3a70a35771e1', 'message_group_uuid': '5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'user_handle': 'andrewbrown', 'sk': '2023-04-02T22:02:18.052306+00:00', 'pk': 'GRP#ed6dd963-d34a-4e82-be39-f73189728ba5', 'message': 'this is a filler message', 'user_display_name': 'Andrew Brown'}
{'user_uuid': 'ed6dd963-d34a-4e82-be39-f73189728ba5', 'user_handle': 'dalvarez', 'sk': '2023-04-02T22:02:18.052306+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '2c3341cf-0c68-436f-b75c-77eda96f40cc', 'message': "Have you ever watched Babylon 5? It's one of my favorite TV shows!", 'user_display_name': 'Daniel Alvarez'}
{'user_uuid': '7a703a5c-036c-4064-a263-3a70a35771e1', 'user_handle': 'andrewbrown', 'sk': '2023-04-02T22:03:18.052306+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '4a539ee4-dc65-472f-9aeb-826a592dcac1', 'message': "Yes, I have! I love it too. What's your favorite season?", 'user_display_name': 'Andrew Brown'}
{'user_uuid': 'ed6dd963-d34a-4e82-be39-f73189728ba5', 'user_handle': 'dalvarez', 'sk': '2023-04-02T22:04:18.052306+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': '9e979bb4-1db5-4829-9859-927e379a7e73', 'message': 'I think my favorite season has to be season 3. So many great episodes, like "Severed Dreams" and "War Without End."', 'user_display_name': 'Daniel Alvarez'}
{'user_uuid': '7a703a5c-036c-4064-a263-3a70a35771e1', 'user_handle': 'andrewbrown', 'sk': '2023-04-02T22:05:18.052306+00:00', 'pk': 'MSG#5ae290ed-55d1-47a0-bc6d-fe2bc2700399', 'message_uuid': 'e789bba8-4b1a-464a-9efc-bfb04d827806', 'message': 'Yeah, season 3 was amazing! I also loved season 4, especially with the Shadow War heating up and the introduction of the White Star.', 'user_display_name': 'Andrew Brown'}



