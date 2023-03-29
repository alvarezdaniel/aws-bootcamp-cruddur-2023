```sh
docker compose  -f "docker-compose.yml" up -d --build db dynamodb-local
docker compose  -f "docker-compose.yml" up -d

cd backend-flask
./bin/db/setup
./bin/db/drop

./bin/ddb/schema-load
./bin/ddb/list-tables
./bin/ddb/drop cruddur-messages

./bin/ddb/seed

./bin/ddb/scan

./bin/ddb/patterns/get-conversation
./bin/ddb/patterns/list-conversations

./bin/cognito/list-users
./bin/db/update_cognito_user_ids
```