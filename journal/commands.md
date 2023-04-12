https://gitpod.io/#https://github.com/alvarezdaniel/aws-bootcamp-cruddur-2023

```sh
docker compose  -f "docker-compose.yml" up -d --build db dynamodb-local
docker compose  -f "docker-compose.yml" up -d --build db
docker compose  -f "docker-compose.yml" up -d
docker compose  -f "docker-compose.yml" up -d --build backend-flask frontend-react-js db xray-daemon

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

./bin/ddb/schema-load prod
./bin/ddb/list-tables prod




Week 6-7 - ECS Fargate (Part 1) 46.19

