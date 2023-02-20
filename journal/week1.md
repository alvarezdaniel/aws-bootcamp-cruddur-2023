# Week 1 — App Containerization

## Introduction

Andrew Brown hosts AWS Ontario Virtual User Group

Sponsors:
- Adrian Cantrill = Free docker fundamentals courses
- WeCloudData = free videos

AWS user groups around the world

## Week 1 hosts

- James Spurin @jamesspurin
- Edith Puclla @EditPuclla
- Shala Warner @Gifted Lane

## Spend considerations

AWS Bills => check every week
Free Tier => Services usage tracking
Gitpod Billing => monitor credits

## Last Week

- Architectural designs
- AWS CLI
- Billing Alarm
- Budget

## New week

- Create new branch week-1 from week-0
- Gitpod start with AWS CLI already installed
- We will containerize apps => more portable, no configuration in environments, easier deployment on multiple environments
- linuxserver.io = multiple images
- dockerhub = registry from docker images (host own images free)
- There are others
- Open Container Initiative (OCI), docker is part of it
- Docker Hub ~= github for docker images
- JFrog Artifactory, more for artifacts
- Install VSCode docker extension => easy to work in docker

## Class Summary

- Create a new GitHub repo
- Launch the repo within a Gitpod workspace
- Configure Gitpod.yml configuration, eg. I’m VSCode Extensions
- Clone the frontend and backend repo
- Explore the codebases
- Ensure we can get the apps running locally
- Write a Dockerfile for each app
- Ensure we get the apps running via individual container
- Create a docker-compose file
- Ensure we can orchestrate multiple containers to run side by side
- Mount directories so we can make changes while we code

## Watched videos

- [Grading Homework Summaries](https://www.youtube.com/watch?v=FKAScachFgk&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=25)
- [Week 1 - Live Streamed Video](https://www.youtube.com/watch?v=zJnNe5Nv4tE&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=22)
- [Remember to Commit Your Code](https://www.youtube.com/watch?v=b-idMgFFcpg&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=23)
- [Chirag's Week 1 - Spending Considerations](https://www.youtube.com/watch?v=OAMHu1NiYoI&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=24)
- [Ashish's Week 1 - Container Security Considerations](https://www.youtube.com/watch?v=OjZz4D0B-cA&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=25)


## Containerize Application (Dockerfiles, Docker Compose)

### Run Backend Flask no container

First of all, test backend application without container.
Start Gitpod in week-1 branch.
Run backend manually.

```python
cd backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
pip3 install -r requirements.txt
python3 -m flask run --host=0.0.0.0 --port=4567
cd ..
```

Unlock port 4567 (make it public)
Open browser: https://4567-alvarezdani-awsbootcamp-3kuy02th1gt.ws-us87.gitpod.io/api/activities/home
Returns json with activities

```json
[{"created_at":"2023-02-17T23:21:16.236823+00:00","expires_at":"2023-02-24T23:21:16.236823+00:00","handle":"Andrew Brown","likes_count":5,"message":"Cloud is fun!","replies":[{"created_at":"2023-02-17T23:21:16.236823+00:00","handle":"Worf","likes_count":0,"message":"This post has no honor!","replies_count":0,"reply_to_activity_uuid":"68f126b0-1ceb-4a33-88be-d90fa7109eee","reposts_count":0,"uuid":"26e12864-1c26-5c3a-9658-97a10f8fea67"}],"replies_count":1,"reposts_count":0,"uuid":"68f126b0-1ceb-4a33-88be-d90fa7109eee"},{"created_at":"2023-02-12T23:21:16.236823+00:00","expires_at":"2023-02-28T23:21:16.236823+00:00","handle":"Worf","likes":0,"message":"I am out of prune juice","replies":[],"uuid":"66e12864-8c26-4c3a-9658-95a10f8fea67"},{"created_at":"2023-02-19T22:21:16.236823+00:00","expires_at":"2023-02-20T11:21:16.236823+00:00","handle":"Garek","likes":0,"message":"My dear doctor, I am just simple tailor","replies":[],"uuid":"248959df-3079-4947-b847-9e0892d1bab4"}]
```

Check app log:
```
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:4567
 * Running on http://10.0.5.2:4567
Press CTRL+C to quit
192.168.64.201 - - [19/Feb/2023 23:20:54] "GET / HTTP/1.1" 404 -
192.168.64.201 - - [19/Feb/2023 23:20:56] "GET /favicon.ico HTTP/1.1" 404 -
192.168.64.201 - - [19/Feb/2023 23:21:12] "GET /api/activities HTTP/1.1" 405 -
192.168.64.201 - - [19/Feb/2023 23:21:16] "GET /api/activities/home HTTP/1.1" 200 -
```

Close app

### Containerize Backend Flask app

Let's try to containerize it.

Clean env vars

```bash
env | grep D_URL
unset FRONTEND_URL
unset BACKEND_URL
env | grep D_URL
```

Add Dockerfile file to /backend-flask folder.

```Dockerfile
FROM python:3.10-slim-buster

# Inside Container
# make a new folder inside container
WORKDIR /backend-flask

# Outside Container -> Inside Container
# this contains the libraries want to install to run the app
COPY requirements.txt requirements.txt

# Inside Container
# Install the python libraries used for the app
RUN pip3 install -r requirements.txt

# Outside Container -> Inside Container
# . means everything in the current directory
# first period . - /backend-flask (outside container)
# second period . /backend-flask (inside container)
COPY . .

# Set Enviroment Variables (Env Vars)
# Inside Container and wil remain set when the container is running
ENV FLASK_ENV=development

EXPOSE ${PORT}

# CMD (Command)
# python3 -m flask run --host=0.0.0.0 --port=4567
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```

Notes:
- Dockerfile = recipe to create image
- FROM = base image from registry
- scratch = unofficial empty image, starting point for every image
- Every FROM is a new layer (reusable)
- Uses Union File System (UnionFS)

1. FROM python:3.10-slim-buster
2. WORKDIR /backend-flask = work directory in guest OS?, ~cd workdir, set context to that folder, make new folder in container
3. COPY requirements.txt requirements.txt = outside->inside, libs to install
4. RUN pip3 install -r requirements.txt = install libs
5. COPY . . = outside to inside, .=/backend-flask/*
6. ENV FLASK_ENV=development = set env vars inside container
7. EXPOSE {$PORT}
8. CMD ["python", "-m", "flask", "run", "--host=0.0.0.0", "--port=4567" ] = runs the app, -m=module

```Dockerfile
FROM python:3.10-slim-buster

WORKDIR /backend-flask

COPY requirements.txt requirements.txt
RUN pip3 install -r requirements.txt

COPY . .

ENV FLASK_ENV=development

EXPOSE ${PORT}
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
```

Build image

```bash
docker build -t  backend-flask ./backend-flask
```

-t = Image name, can include tag or not. If not present = latest

Run container

```bash
docker run --rm -p 4567:4567 -it backend-flask
FRONTEND_URL="*" BACKEND_URL="*" docker run --rm -p 4567:4567 -it backend-flask
export FRONTEND_URL="*"
export BACKEND_URL="*"
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
docker run --rm -p 4567:4567 -it  -e FRONTEND_URL -e BACKEND_URL backend-flask
unset FRONTEND_URL="*"
unset BACKEND_URL="*"
```

Env vars can be set while running container, passing values or using host values

Install VSCODE Docker extension
Check created image

```bash
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
```

Open port and open browser to inspect
https://4567-alvarezdani-awsbootcamp-3kuy02th1gt.ws-us87.gitpod.io/api/activities/home

Json is returned:

```json
[
  {
    "created_at": "2023-02-17T23:35:55.899073+00:00",
    "expires_at": "2023-02-24T23:35:55.899073+00:00",
    "handle": "Andrew Brown",
    "likes_count": 5,
    "message": "Cloud is fun!",
    "replies": [
      {
        "created_at": "2023-02-17T23:35:55.899073+00:00",
        "handle": "Worf",
        "likes_count": 0,
        "message": "This post has no honor!",
        "replies_count": 0,
        "reply_to_activity_uuid": "68f126b0-1ceb-4a33-88be-d90fa7109eee",
        "reposts_count": 0,
        "uuid": "26e12864-1c26-5c3a-9658-97a10f8fea67"
      }
    ],
    "replies_count": 1,
    "reposts_count": 0,
    "uuid": "68f126b0-1ceb-4a33-88be-d90fa7109eee"
  },
  {
    "created_at": "2023-02-12T23:35:55.899073+00:00",
    "expires_at": "2023-02-28T23:35:55.899073+00:00",
    "handle": "Worf",
    "likes": 0,
    "message": "I am out of prune juice",
    "replies": [],
    "uuid": "66e12864-8c26-4c3a-9658-95a10f8fea67"
  },
  {
    "created_at": "2023-02-19T22:35:55.899073+00:00",
    "expires_at": "2023-02-20T11:35:55.899073+00:00",
    "handle": "Garek",
    "likes": 0,
    "message": "My dear doctor, I am just simple tailor",
    "replies": [],
    "uuid": "248959df-3079-4947-b847-9e0892d1bab4"
  }
]
```

Inspect logs and attach shell using Docker extension.

To run container in detached mode, add -d flag

```bash
docker container run --rm -p 4567:4567 -d backend-flask
```

-rm indicates remove container on exit

To check for running containers, use ps

```bash
docker ps
```

CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS                                       NAMES
c3420db1fc33   backend-flask   "python3 -m flask ru…"   10 seconds ago   Up 10 seconds   0.0.0.0:4567->4567/tcp, :::4567->4567/tcp   fervent_mestorf

For returning already removed images, use docker ps -a

How to return the container id into an env var

```bash
CONTAINER_ID=$(docker run --rm -p 4567:4567 -d backend-flask)
```

> docker container run is idiomatic, docker run is legacy syntax but is commonly used.

Get Container Images or Running Container Ids

```bash
docker ps
docker images
```

Send Curl to Test Server

```bash
curl -X GET http://localhost:4567/api/activities/home -H "Accept: application/json" -H "Content-Type: application/json"
```

Check Container Logs

```bash
docker logs CONTAINER_ID -f
docker logs backend-flask -f
docker logs $CONTAINER_ID -f
```

Debugging  adjacent containers with other containers

```sh
docker run --rm -it curlimages/curl "-X GET http://localhost:4567/api/activities/home -H \"Accept: application/json\" -H \"Content-Type: application/json\""
```

busybosy is often used for debugging since it install a bunch of thing

```sh
docker run --rm -it busybosy
```

Gain Access to a Container

```sh
docker exec CONTAINER_ID -it /bin/bash
```

> You can just right click a container and see logs in VSCode with Docker extension

Delete an Image

```sh
docker image rm backend-flask --force
```

> docker rmi backend-flask is the legacy syntax, you might see this is old docker tutorials and articles.

> There are some cases where you need to use the --force

Overriding Ports

```sh
FLASK_ENV=production PORT=8080 docker run -p 4567:4567 -it backend-flask
```

> Look at Dockerfile to see how ${PORT} is interpolated

### Containerize Frontend React app

Add Dockerfile to frontend-react-js folder

```Dockerfile
FROM node:16.18

ENV PORT=3000

COPY . /frontend-react-js
WORKDIR /frontend-react-js
RUN npm install
EXPOSE ${PORT}
CMD ["npm", "start"]
```

Build Image

```sh
docker build -t frontend-react-js ./frontend-react-js
```

Run Container

```sh
docker run -p 3000:3000 -d frontend-react-js
```

Unlock port 3000 and test it in browser:
https://3000-alvarezdani-awsbootcamp-3kuy02th1gt.ws-us87.gitpod.io/

### Multiple Containers using compose

Create `docker-compose.yml` at the root of your project.

```yaml
version: "3.8"
services:
  backend-flask:
    environment:
      FRONTEND_URL: "https://3000-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
      BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./backend-flask
    ports:
      - "4567:4567"
    volumes:
      - ./backend-flask:/backend-flask
  frontend-react-js:
    environment:
      REACT_APP_BACKEND_URL: "https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
    build: ./frontend-react-js
    ports:
      - "3000:3000"
    volumes:
      - ./frontend-react-js:/frontend-react-js

# the name flag is a hack to change the default prepend folder
# name when outputting the image names
networks: 
  internal-network:
    driver: bridge
    name: cruddur
```

It includes 2 services and orchestrate them starting multiple containers

Start compose by running

```sh
docker compose up
```

or by using VSCode extension, right click and Compose Up

Open browser to test it:
https://3000-alvarezdani-awsbootcamp-bpzo93rvx4h.ws-us87.gitpod.io/
Error, Port 3000 Not Found

Inspect logs in frontend container:

```sh
docker logs --tail 1000 -f 25093c1067d63ddbe707c0ad4b680429b25c59a2bff78f1f5243b827a465101b 
```

```
> frontend@0.1.0 start
> react-scripts start

sh: 1: react-scripts: not found

> frontend@0.1.0 start
> react-scripts start

sh: 1: react-scripts: not found
```

Test frontend again using docker build and run
https://3000-alvarezdani-awsbootcamp-bpzo93rvx4h.ws-us87.gitpod.io/
It works

I could make it work by manually running npm install on frontend folder before starting compose file

https://3000-alvarezdani-awsbootcamp-bpzo93rvx4h.ws-us87.gitpod.io/

> Note on docker compose vs docker-compose, James explained docker plugins or extension works this way, same as kubectl in kubernetes



