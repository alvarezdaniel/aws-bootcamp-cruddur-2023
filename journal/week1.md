# Week 1 — App Containerization

## Introduction

Andrew Brown hosts AWS Ontario Virtual User Group

Sponsors:
- Adrian Cantrill = Free docker fundamentals courses
- WeCloudData = free videos

AWS user groups around the world

## Week 1 instructors

- James Spurin [@jamesspurin](https://twitter.com/jamesspurin)
- Edith Puclla [@EdithPuclla](https://twitter.com/EdithPuclla)
- Shala Warner [@GiftedLane](https://twitter.com/GiftedLane)

## Spend considerations

- AWS Bills => check every week
- Free Tier => Services usage tracking
- Gitpod Billing => monitor credits

## Last Week

- Architectural designs
- AWS CLI
- Billing Alarm
- Budget

## New week

- Create new branch week-1 from week-0
- Gitpod start with AWS CLI already installed
- We will containerize apps => more portable, no configuration in environments, easier deployment on multiple environments
- linuxserver.io = multiple images https://www.linuxserver.io/
- dockerhub = registry from docker images (host own images free) https://hub.docker.com/
- There are others
- Open Container Initiative (OCI), docker is part of it https://opencontainers.org/
- Docker Hub ~= github for docker images
- JFrog Artifactory, more for artifacts https://jfrog.com/artifactory/
- Install VSCode docker extension => makes it easy to work with docker from VSCode https://code.visualstudio.com/docs/containers/overview (already preinstalled in Gitpod)
- Good article for Debugging Connection Refused https://pythonspeed.com/articles/docker-connection-refused/

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

- [How to Ask for Technical Help](https://www.youtube.com/watch?v=tDPqmwKMP7Y&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=29)
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

```sh
gitpod /workspace/aws-bootcamp-cruddur-2023 (week-1) $ cd backend-flask
gitpod /workspace/aws-bootcamp-cruddur-2023/backend-flask (week-1) $ export FRONTEND_URL="*"
gitpod /workspace/aws-bootcamp-cruddur-2023/backend-flask (week-1) $ export BACKEND_URL="*"
gitpod /workspace/aws-bootcamp-cruddur-2023/backend-flask (week-1) $ pip3 install -r requirements.txt
Collecting flask
  Downloading Flask-2.2.3-py3-none-any.whl (101 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 101.8/101.8 kB 2.9 MB/s eta 0:00:00
Collecting flask-cors
  Downloading Flask_Cors-3.0.10-py2.py3-none-any.whl (14 kB)
Collecting itsdangerous>=2.0
  Downloading itsdangerous-2.1.2-py3-none-any.whl (15 kB)
Requirement already satisfied: importlib-metadata>=3.6.0 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from flask->-r requirements.txt (line 1)) (6.0.0)
Requirement already satisfied: Jinja2>=3.0 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from flask->-r requirements.txt (line 1)) (3.1.2)
Collecting click>=8.0
  Downloading click-8.1.3-py3-none-any.whl (96 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 96.6/96.6 kB 3.9 MB/s eta 0:00:00
Collecting Werkzeug>=2.2.2
  Downloading Werkzeug-2.2.3-py3-none-any.whl (233 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 233.6/233.6 kB 10.0 MB/s eta 0:00:00
Requirement already satisfied: Six in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from flask-cors->-r requirements.txt (line 2)) (1.16.0)
Requirement already satisfied: zipp>=0.5 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from importlib-metadata>=3.6.0->flask->-r requirements.txt (line 1)) (3.11.0)
Requirement already satisfied: MarkupSafe>=2.0 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from Jinja2>=3.0->flask->-r requirements.txt (line 1)) (2.1.1)
Installing collected packages: Werkzeug, itsdangerous, click, flask, flask-cors
Successfully installed Werkzeug-2.2.3 click-8.1.3 flask-2.2.3 flask-cors-3.0.10 itsdangerous-2.1.2

[notice] A new release of pip available: 22.3.1 -> 23.0.1
[notice] To update, run: pip install --upgrade pip
gitpod /workspace/aws-bootcamp-cruddur-2023/backend-flask (week-1) $ python3 -m flask run --host=0.0.0.0 --port=4567
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:4567
 * Running on http://10.0.5.2:4567
Press CTRL+C to quit
```

![Run backend directly](assets/week-1/Run%20backend%20directly.png)

Unlock port 4567 (make it public)

![Unlock port 4567](assets/week-1/Unlock%20port%204567.png)

![Port 4567 unlocked](assets/week-1/Port%204567%20unlocked.png)

Open browser in backend, getting activities endpoint: 
https://4567-alvarezdani-awsbootcamp-n9d8o604pch.ws-us87.gitpod.io/api/activities/home

![Get activities](assets/week-1/Get%20activities.png)

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

![Get backend logs](assets/week-1/Get%20backend%20logs.png)

Close app

### Containerize Backend Flask app

Let's try to containerize backend application.

Clean environment variables.

```sh
unset FRONTEND_URL
unset BACKEND_URL
env | grep D_URL
```

Create `Dockerfile` file in `/backend-flask` folder.

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

Explanation:

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

```sh
docker build -t  backend-flask ./backend-flask
```

-t = Image name, can include tag or not. If not present = latest

```
Sending build context to Docker daemon   59.9kB
Step 1/8 : FROM python:3.10-slim-buster
3.10-slim-buster: Pulling from library/python
29cd48154c03: Pull complete 
2c59e55cfd71: Pull complete 
3b4b58298de0: Pull complete 
6239e464c1ab: Pull complete 
047dd5665bb1: Pull complete 
Digest: sha256:6e96825731607f9d49d382e302a78e994d60db2871f3447152f56621069e6114
Status: Downloaded newer image for python:3.10-slim-buster
 ---> b5d627f77479
Step 2/8 : WORKDIR /backend-flask
 ---> Running in 70be5b413c7b
Removing intermediate container 70be5b413c7b
 ---> aac6c24bfd67
Step 3/8 : COPY requirements.txt requirements.txt
 ---> 967f5d7947c1
Step 4/8 : RUN pip3 install -r requirements.txt
 ---> Running in 75a67eabe92c
Collecting flask
  Downloading Flask-2.2.3-py3-none-any.whl (101 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 101.8/101.8 kB 2.5 MB/s eta 0:00:00
Collecting flask-cors
  Downloading Flask_Cors-3.0.10-py2.py3-none-any.whl (14 kB)
Collecting itsdangerous>=2.0
  Downloading itsdangerous-2.1.2-py3-none-any.whl (15 kB)
Collecting click>=8.0
  Downloading click-8.1.3-py3-none-any.whl (96 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 96.6/96.6 kB 3.5 MB/s eta 0:00:00
Collecting Jinja2>=3.0
  Downloading Jinja2-3.1.2-py3-none-any.whl (133 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 133.1/133.1 kB 6.0 MB/s eta 0:00:00
Collecting Werkzeug>=2.2.2
  Downloading Werkzeug-2.2.3-py3-none-any.whl (233 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 233.6/233.6 kB 11.7 MB/s eta 0:00:00
Collecting Six
  Downloading six-1.16.0-py2.py3-none-any.whl (11 kB)
Collecting MarkupSafe>=2.0
  Downloading MarkupSafe-2.1.2-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (25 kB)
Installing collected packages: Six, MarkupSafe, itsdangerous, click, Werkzeug, Jinja2, flask, flask-cors
Successfully installed Jinja2-3.1.2 MarkupSafe-2.1.2 Six-1.16.0 Werkzeug-2.2.3 click-8.1.3 flask-2.2.3 flask-cors-3.0.10 itsdangerous-2.1.2
WARNING: Running pip as the 'root' user can result in broken permissions and conflicting behaviour with the system package manager. It is recommended to use a virtual environment instead: https://pip.pypa.io/warnings/venv

[notice] A new release of pip available: 22.3.1 -> 23.0.1
[notice] To update, run: pip install --upgrade pip
Removing intermediate container 75a67eabe92c
 ---> 8834b2f9e561
Step 5/8 : COPY . .
 ---> 6ee623320bbe
Step 6/8 : ENV FLASK_ENV=development
 ---> Running in e3caf6a6f1cc
Removing intermediate container e3caf6a6f1cc
 ---> a4003c88d92d
Step 7/8 : EXPOSE ${PORT}
 ---> Running in 2f3618873a1f
Removing intermediate container 2f3618873a1f
 ---> 44fc7a53571d
Step 8/8 : CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
 ---> Running in f1a3fc2a0fd8
Removing intermediate container f1a3fc2a0fd8
 ---> 4be1710de3b8
Successfully built 4be1710de3b8
Successfully tagged backend-flask:latest
```

Run container

```sh
#docker run --rm -p 4567:4567 -it backend-flask
#FRONTEND_URL="*" BACKEND_URL="*" docker run --rm -p 4567:4567 -it backend-flask
#export FRONTEND_URL="*"
#export BACKEND_URL="*"
docker run --rm -p 4567:4567 -it -e FRONTEND_URL='*' -e BACKEND_URL='*' backend-flask
#docker run --rm -p 4567:4567 -it  -e FRONTEND_URL -e BACKEND_URL backend-flask
#unset FRONTEND_URL="*"
#unset BACKEND_URL="*"
```

```
'FLASK_ENV' is deprecated and will not be used in Flask 2.3. Use 'FLASK_DEBUG' instead.
'FLASK_ENV' is deprecated and will not be used in Flask 2.3. Use 'FLASK_DEBUG' instead.
'FLASK_ENV' is deprecated and will not be used in Flask 2.3. Use 'FLASK_DEBUG' instead.
 * Debug mode: on
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on all addresses (0.0.0.0)
 * Running on http://127.0.0.1:4567
 * Running on http://172.17.0.2:4567
Press CTRL+C to quit
 * Restarting with stat
'FLASK_ENV' is deprecated and will not be used in Flask 2.3. Use 'FLASK_DEBUG' instead.
'FLASK_ENV' is deprecated and will not be used in Flask 2.3. Use 'FLASK_DEBUG' instead.
'FLASK_ENV' is deprecated and will not be used in Flask 2.3. Use 'FLASK_DEBUG' instead.
 * Debugger is active!
 * Debugger PIN: 538-943-984
 ```

![Run backend container](assets/week-1/Run%20backend%20container.png)

Environment variables can be set while running container, passing values or using host values

In VSCode Docker extension check created image

![Check backend container](assets/week-1/Check%20backend%20container.png)

Open port and open browser to inspect the result
https://4567-alvarezdani-awsbootcamp-3kuy02th1gt.ws-us87.gitpod.io/api/activities/home

![Get activities container](assets/week-1/Get%20activities%20container.png)

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

Using Docker extension logs can be inspected and a shell can be attached:

![Inspect logs](assets/week-1/Inspect%20logs.png)

![Attach shell](assets/week-1/Attach%20shell.png)

To run container in detached mode, add -d flag

```sh
docker container run --rm -p 4567:4567 -d backend-flask
```

-rm indicates remove container on exit

To check for running containers, use ps

```sh
docker ps
```

CONTAINER ID   IMAGE           COMMAND                  CREATED          STATUS          PORTS                                       NAMES
c3420db1fc33   backend-flask   "python3 -m flask ru…"   10 seconds ago   Up 10 seconds   0.0.0.0:4567->4567/tcp, :::4567->4567/tcp   fervent_mestorf

For returning stopped containers, use docker ps -a

How to return the container id into an env var

```sh
CONTAINER_ID=$(docker run --rm -p 4567:4567 -d backend-flask)
```

> docker container run is idiomatic, docker run is legacy syntax but is commonly used.

Get Container Images or Running Container Ids

```sh
docker ps
docker images
```

Send curl to test server

```sh
curl -X GET http://localhost:4567/api/activities/home -H "Accept: application/json" -H "Content-Type: application/json"
```

Returns

```json
[
  {
    "created_at": "2023-02-21T14:57:07.404279+00:00",
    "expires_at": "2023-02-28T14:57:07.404279+00:00",
    "handle": "Andrew Brown",
    "likes_count": 5,
    "message": "Cloud is fun!",
    "replies": [
      {
        "created_at": "2023-02-21T14:57:07.404279+00:00",
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
    "created_at": "2023-02-16T14:57:07.404279+00:00",
    "expires_at": "2023-03-04T14:57:07.404279+00:00",
    "handle": "Worf",
    "likes": 0,
    "message": "I am out of prune juice",
    "replies": [],
    "uuid": "66e12864-8c26-4c3a-9658-95a10f8fea67"
  },
  {
    "created_at": "2023-02-23T13:57:07.404279+00:00",
    "expires_at": "2023-02-24T02:57:07.404279+00:00",
    "handle": "Garek",
    "likes": 0,
    "message": "My dear doctor, I am just simple tailor",
    "replies": [],
    "uuid": "248959df-3079-4947-b847-9e0892d1bab4"
  }
]
```

![Run curl](assets/week-1/Run%20curl.png)

Check Container Logs

```sh
docker logs CONTAINER_ID -f
docker logs backend-flask -f
docker logs $CONTAINER_ID -f
```

Debugging adjacent containers with other containers

```sh
docker run --rm -it curlimages/curl "-X GET http://localhost:4567/api/activities/home -H \"Accept: application/json\" -H \"Content-Type: application/json\""
```

busybosy is often used for debugging since it installs a bunch of things

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

```
Sending build context to Docker daemon  267.7MB
Step 1/7 : FROM node:16.18
16.18: Pulling from library/node
620af4e91dbf: Pull complete 
fae29f309a72: Pull complete 
28fca74d99b6: Pull complete 
0b5db87f5b42: Pull complete 
fa488706ea13: Pull complete 
0380b9b3282f: Pull complete 
383dfecd3687: Pull complete 
ca59981dc274: Pull complete 
4fa5c4b55a85: Pull complete 
Digest: sha256:7f404d09ceb780c51f4fac7592c46b8f21211474aacce25389eb0df06aaa7472
Status: Downloaded newer image for node:16.18
 ---> 993a4cf9c1e8
Step 2/7 : ENV PORT=3000
 ---> Running in 9058f4dac05b
Removing intermediate container 9058f4dac05b
 ---> 98f840170e87
Step 3/7 : COPY . /frontend-react-js
 ---> 72dfcb1178dc
Step 4/7 : WORKDIR /frontend-react-js
 ---> Running in c13ff6c61b92
Removing intermediate container c13ff6c61b92
 ---> 6055685d15fd
Step 5/7 : RUN npm install
 ---> Running in 5146b7d87263

up to date, audited 1472 packages in 4s

225 packages are looking for funding
  run `npm fund` for details

8 high severity vulnerabilities

To address issues that do not require attention, run:
  npm audit fix

To address all issues (including breaking changes), run:
  npm audit fix --force

Run `npm audit` for details.
npm notice 
npm notice New major version of npm available! 8.19.2 -> 9.5.1
npm notice Changelog: <https://github.com/npm/cli/releases/tag/v9.5.1>
npm notice Run `npm install -g npm@9.5.1` to update!
npm notice 
Removing intermediate container 5146b7d87263
 ---> 508731d669fa
Step 6/7 : EXPOSE ${PORT}
 ---> Running in 9b64037d77f8
Removing intermediate container 9b64037d77f8
 ---> cf3496e860bf
Step 7/7 : CMD ["npm", "start"]
 ---> Running in fbab96423b0a
Removing intermediate container fbab96423b0a
 ---> af97b8e6ad43
Successfully built af97b8e6ad43
Successfully tagged frontend-react-js:latest
```

![Build frontend](assets/week-1/Build%20frontend.png)

Run Container

```sh
docker run -p 3000:3000 -d frontend-react-js
```

```
986128c25ef17d0abeca6b2171c80e83907b14251e66f21c35e53456acffb225
```

![Run frontend](assets/week-1/Run%20frontend.png)

Unlock port 3000 and test it in browser:
https://3000-alvarezdani-awsbootcamp-3kuy02th1gt.ws-us87.gitpod.io/

![Browse frontend](assets/week-1/Browse%20frontend.png)

### Multiple Containers using compose

Create `docker-compose.yml` in the root folder.

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

It includes 2 services and orchestrate multiple containers

Start compose by running

```sh
docker compose up
```

```
[+] Running 14/14
 ⠿ db Pulled                                                                                                                                                                                       13.8s
   ⠿ 63b65145d645 Pull complete                                                                                                                                                                     0.6s
   ⠿ c441836541d9 Pull complete                                                                                                                                                                     0.9s
   ⠿ d49de1a24361 Pull complete                                                                                                                                                                     1.3s
   ⠿ 6c609d08dc3c Pull complete                                                                                                                                                                    10.0s
   ⠿ e801cecfb07c Pull complete                                                                                                                                                                    11.0s
   ⠿ 98756d431840 Pull complete                                                                                                                                                                    11.4s
   ⠿ d991736c9c5a Pull complete                                                                                                                                                                    11.8s
   ⠿ c708ebe2e647 Pull complete                                                                                                                                                                    12.1s
 ⠿ dynamodb-local Pulled                                                                                                                                                                           15.1s
   ⠿ 1e78b99dd1fd Pull complete                                                                                                                                                                     6.9s
   ⠿ 6aabd9041a5c Pull complete                                                                                                                                                                    11.7s
   ⠿ 4f4fb700ef54 Pull complete                                                                                                                                                                    12.1s
   ⠿ 601cf5909367 Pull complete                                                                                                                                                                    13.3s
[+] Building 47.2s (18/18) FINISHED                                                                                                                                                                      
 => [aws-bootcamp-cruddur-2023-backend-flask internal] load build definition from Dockerfile                                                                                                        0.6s
 => => transferring dockerfile: 306B                                                                                                                                                                0.0s
 => [aws-bootcamp-cruddur-2023-frontend-react-js internal] load build definition from Dockerfile                                                                                                    0.9s
 => => transferring dockerfile: 183B                                                                                                                                                                0.0s
 => [aws-bootcamp-cruddur-2023-backend-flask internal] load .dockerignore                                                                                                                           0.7s
 => => transferring context: 2B                                                                                                                                                                     0.0s
 => [aws-bootcamp-cruddur-2023-frontend-react-js internal] load .dockerignore                                                                                                                       1.0s
 => => transferring context: 2B                                                                                                                                                                     0.0s
 => [aws-bootcamp-cruddur-2023-backend-flask internal] load metadata for docker.io/library/python:3.10-slim-buster                                                                                  1.2s
 => [aws-bootcamp-cruddur-2023-frontend-react-js internal] load metadata for docker.io/library/node:16.18                                                                                           1.6s
 => [aws-bootcamp-cruddur-2023-backend-flask 1/5] FROM docker.io/library/python:3.10-slim-buster@sha256:6e96825731607f9d49d382e302a78e994d60db2871f3447152f56621069e6114                           12.4s
 => => resolve docker.io/library/python:3.10-slim-buster@sha256:6e96825731607f9d49d382e302a78e994d60db2871f3447152f56621069e6114                                                                    0.3s
 => => sha256:b5d627f77479c9a736353483a055a40e32399f10e0485dcdd760611f66343b38 7.79kB / 7.79kB                                                                                                      0.0s
 => => sha256:6e96825731607f9d49d382e302a78e994d60db2871f3447152f56621069e6114 988B / 988B                                                                                                          0.0s
 => => sha256:46ad79f446eb9cca09dc5c231b75e13129b663815aedd3c9e622f7e05feab164 1.37kB / 1.37kB                                                                                                      0.0s
 => => sha256:29cd48154c03e9242f1ff4f9895cf886a344fb94c9b71029455e76e11214328f 27.14MB / 27.14MB                                                                                                    0.6s
 => => sha256:2c59e55cfd719490e5070eb007a3b0ffde33d3e36171a573a3224a050c1e341d 2.78MB / 2.78MB                                                                                                      0.5s
 => => sha256:3b4b58298de0d8fc8f69c675c463afab2beb87c60a81d13b5620c6b87ee42cbb 11.47MB / 11.47MB                                                                                                    0.6s
 => => extracting sha256:29cd48154c03e9242f1ff4f9895cf886a344fb94c9b71029455e76e11214328f                                                                                                           3.4s
 => => sha256:6239e464c1ab2cca0154db8e88c6b2eb8969c76dd4cdf8e67e03be54df84ac33 232B / 232B                                                                                                          0.9s
 => => sha256:047dd5665bb1efccfca5ee9b67835b9accb753c462f1d531d62ea0c78923420f 3.35MB / 3.35MB                                                                                                      1.0s
 => => extracting sha256:2c59e55cfd719490e5070eb007a3b0ffde33d3e36171a573a3224a050c1e341d                                                                                                           0.5s
 => => extracting sha256:3b4b58298de0d8fc8f69c675c463afab2beb87c60a81d13b5620c6b87ee42cbb                                                                                                           1.0s
 => => extracting sha256:6239e464c1ab2cca0154db8e88c6b2eb8969c76dd4cdf8e67e03be54df84ac33                                                                                                           0.0s
 => => extracting sha256:047dd5665bb1efccfca5ee9b67835b9accb753c462f1d531d62ea0c78923420f                                                                                                           0.2s
 => [aws-bootcamp-cruddur-2023-backend-flask internal] load build context                                                                                                                           0.3s
 => => transferring context: 21.39kB                                                                                                                                                                0.0s
 => [aws-bootcamp-cruddur-2023-frontend-react-js internal] load build context                                                                                                                       9.4s
 => => transferring context: 239.37MB                                                                                                                                                               9.0s
 => [aws-bootcamp-cruddur-2023-frontend-react-js 1/4] FROM docker.io/library/node:16.18@sha256:7f404d09ceb780c51f4fac7592c46b8f21211474aacce25389eb0df06aaa7472                                    26.0s
 => => resolve docker.io/library/node:16.18@sha256:7f404d09ceb780c51f4fac7592c46b8f21211474aacce25389eb0df06aaa7472                                                                                 0.2s
 => => sha256:7f404d09ceb780c51f4fac7592c46b8f21211474aacce25389eb0df06aaa7472 776B / 776B                                                                                                          0.0s
 => => sha256:46a10b2d8f2e9ae5c5e8ffedd5ae18a960d64b8c39c09e24fe2ee41d7148c249 2.21kB / 2.21kB                                                                                                      0.0s
 => => sha256:993a4cf9c1e80aa74567d3deea4dfa1488b94dcb024bfca9246f979845763509 7.51kB / 7.51kB                                                                                                      0.0s
 => => sha256:fae29f309a72482bf13fbb1a8f4889ab9107fcad0c9fda76586aa55445e93ded 7.86MB / 7.86MB                                                                                                      0.7s
 => => sha256:620af4e91dbf80032eee9f1ff66a8b04119d7a329b2a13e007d69c8a0b337bf0 50.45MB / 50.45MB                                                                                                    1.1s
 => => sha256:28fca74d99b6532401bfe63d36e1bafb1ac839564d48aa4e6e0a6aa2706a4d12 10.00MB / 10.00MB                                                                                                    1.2s
 => => sha256:0b5db87f5b42af9f258f14f367616814cb9b518ea0141f46bdd2706bb256d408 51.84MB / 51.84MB                                                                                                    1.9s
 => => extracting sha256:620af4e91dbf80032eee9f1ff66a8b04119d7a329b2a13e007d69c8a0b337bf0                                                                                                           5.2s
 => => sha256:0380b9b3282fe25f00e7d8191dacb0167d90b7b881f05ff9b1ca72bcd38b9a6b 4.20kB / 4.20kB                                                                                                      1.6s
 => => sha256:fa488706ea13a788b351252b655f6ccb88201c4bace57cf25408fda65758c518 191.89MB / 191.89MB                                                                                                  5.8s
 => => sha256:383dfecd36873ac6c0fc1feb73b2febfa30a5502bea0566c51cf170234248004 34.97MB / 34.97MB                                                                                                    2.6s
 => => sha256:ca59981dc274124ea3f9d421b039e54062b53aba0c04f536bbe2545a80bdba51 2.28MB / 2.28MB                                                                                                      2.4s
 => => sha256:4fa5c4b55a850ac871248b30c51c35c6c77adbc12258b823a725d84a6853dbc2 450B / 450B                                                                                                          2.9s
 => => extracting sha256:fae29f309a72482bf13fbb1a8f4889ab9107fcad0c9fda76586aa55445e93ded                                                                                                           0.6s
 => => extracting sha256:28fca74d99b6532401bfe63d36e1bafb1ac839564d48aa4e6e0a6aa2706a4d12                                                                                                           0.2s
 => => extracting sha256:0b5db87f5b42af9f258f14f367616814cb9b518ea0141f46bdd2706bb256d408                                                                                                           2.3s
 => => extracting sha256:fa488706ea13a788b351252b655f6ccb88201c4bace57cf25408fda65758c518                                                                                                           7.1s
 => => extracting sha256:0380b9b3282fe25f00e7d8191dacb0167d90b7b881f05ff9b1ca72bcd38b9a6b                                                                                                           0.1s
 => => extracting sha256:383dfecd36873ac6c0fc1feb73b2febfa30a5502bea0566c51cf170234248004                                                                                                           1.4s
 => => extracting sha256:ca59981dc274124ea3f9d421b039e54062b53aba0c04f536bbe2545a80bdba51                                                                                                           0.1s
 => => extracting sha256:4fa5c4b55a850ac871248b30c51c35c6c77adbc12258b823a725d84a6853dbc2                                                                                                           0.0s
 => [aws-bootcamp-cruddur-2023-backend-flask 2/5] WORKDIR /backend-flask                                                                                                                            0.6s
 => [aws-bootcamp-cruddur-2023-backend-flask 3/5] COPY requirements.txt requirements.txt                                                                                                            0.6s
 => [aws-bootcamp-cruddur-2023-backend-flask 4/5] RUN pip3 install -r requirements.txt                                                                                                              5.4s
 => [aws-bootcamp-cruddur-2023-backend-flask 5/5] COPY . .                                                                                                                                          0.9s
 => [aws-bootcamp-cruddur-2023-frontend-react-js] exporting to image                                                                                                                                9.2s
 => => exporting layers                                                                                                                                                                             7.3s
 => => writing image sha256:677ef7ac203970662f69d76bb57792fec393221178eb4a57e5d3cae06f31f78d                                                                                                        0.0s
 => => naming to docker.io/library/aws-bootcamp-cruddur-2023-backend-flask                                                                                                                          0.0s
 => => writing image sha256:66e7db21b9cda2226c41346d35102f0df6103d0f21345af3df532a8cb2727cdc                                                                                                        0.0s
 => => naming to docker.io/library/aws-bootcamp-cruddur-2023-frontend-react-js                                                                                                                      0.0s
 => [aws-bootcamp-cruddur-2023-frontend-react-js 2/4] COPY . /frontend-react-js                                                                                                                     5.9s
 => [aws-bootcamp-cruddur-2023-frontend-react-js 3/4] WORKDIR /frontend-react-js                                                                                                                    0.5s
 => [aws-bootcamp-cruddur-2023-frontend-react-js 4/4] RUN npm install                                                                                                                               4.4s
[+] Running 6/6
 ⠿ Network aws-bootcamp-cruddur-2023_default                Created                                                                                                                                 0.1s
 ⠿ Volume "aws-bootcamp-cruddur-2023_db"                    Created                                                                                                                                 0.0s
 ⠿ Container aws-bootcamp-cruddur-2023-backend-flask-1      Created                                                                                                                                 0.5s
 ⠿ Container dynamodb-local                                 Created                                                                                                                                 0.5s
 ⠿ Container aws-bootcamp-cruddur-2023-frontend-react-js-1  Created                                                                                                                                 0.6s
 ⠿ Container aws-bootcamp-cruddur-2023-db-1                 Created                                                                                                                                 0.5s
Attaching to aws-bootcamp-cruddur-2023-backend-flask-1, aws-bootcamp-cruddur-2023-db-1, aws-bootcamp-cruddur-2023-frontend-react-js-1, dynamodb-local
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | > frontend@0.1.0 start
aws-bootcamp-cruddur-2023-frontend-react-js-1  | > react-scripts start
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-db-1                 | The files belonging to this database system will be owned by user "postgres".
aws-bootcamp-cruddur-2023-db-1                 | This user must also own the server process.
aws-bootcamp-cruddur-2023-db-1                 | 
aws-bootcamp-cruddur-2023-db-1                 | The database cluster will be initialized with locale "en_US.utf8".
aws-bootcamp-cruddur-2023-db-1                 | The default database encoding has accordingly been set to "UTF8".
aws-bootcamp-cruddur-2023-db-1                 | The default text search configuration will be set to "english".
aws-bootcamp-cruddur-2023-db-1                 | 
aws-bootcamp-cruddur-2023-db-1                 | Data page checksums are disabled.
aws-bootcamp-cruddur-2023-db-1                 | 
aws-bootcamp-cruddur-2023-db-1                 | fixing permissions on existing directory /var/lib/postgresql/data ... ok
aws-bootcamp-cruddur-2023-db-1                 | creating subdirectories ... ok
aws-bootcamp-cruddur-2023-db-1                 | selecting dynamic shared memory implementation ... posix
aws-bootcamp-cruddur-2023-db-1                 | selecting default max_connections ... 100
aws-bootcamp-cruddur-2023-db-1                 | selecting default shared_buffers ... 128MB
aws-bootcamp-cruddur-2023-db-1                 | selecting default time zone ... UTC
aws-bootcamp-cruddur-2023-db-1                 | creating configuration files ... ok
dynamodb-local                                 | Initializing DynamoDB Local with the following configuration:
dynamodb-local                                 | Port:  8000
dynamodb-local                                 | InMemory:      false
dynamodb-local                                 | DbPath:        ./data
dynamodb-local                                 | SharedDb:      true
dynamodb-local                                 | shouldDelayTransientStatuses:  false
dynamodb-local                                 | CorsParams:    null
dynamodb-local                                 | 
aws-bootcamp-cruddur-2023-db-1                 | running bootstrap script ... ok
aws-bootcamp-cruddur-2023-backend-flask-1      | 'FLASK_ENV' is deprecated and will not be used in Flask 2.3. Use 'FLASK_DEBUG' instead.
aws-bootcamp-cruddur-2023-backend-flask-1      | 'FLASK_ENV' is deprecated and will not be used in Flask 2.3. Use 'FLASK_DEBUG' instead.
aws-bootcamp-cruddur-2023-backend-flask-1      | 'FLASK_ENV' is deprecated and will not be used in Flask 2.3. Use 'FLASK_DEBUG' instead.
aws-bootcamp-cruddur-2023-backend-flask-1      |  * Debug mode: on
aws-bootcamp-cruddur-2023-backend-flask-1      | WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
aws-bootcamp-cruddur-2023-backend-flask-1      |  * Running on all addresses (0.0.0.0)
aws-bootcamp-cruddur-2023-backend-flask-1      |  * Running on http://127.0.0.1:4567
aws-bootcamp-cruddur-2023-backend-flask-1      |  * Running on http://172.18.0.5:4567
aws-bootcamp-cruddur-2023-backend-flask-1      | Press CTRL+C to quit
aws-bootcamp-cruddur-2023-backend-flask-1      |  * Restarting with stat
aws-bootcamp-cruddur-2023-db-1                 | performing post-bootstrap initialization ... sh: locale: not found
aws-bootcamp-cruddur-2023-db-1                 | 2023-02-23 15:06:28.827 UTC [48] WARNING:  no usable system locales were found
aws-bootcamp-cruddur-2023-backend-flask-1      | 'FLASK_ENV' is deprecated and will not be used in Flask 2.3. Use 'FLASK_DEBUG' instead.
aws-bootcamp-cruddur-2023-backend-flask-1      | 'FLASK_ENV' is deprecated and will not be used in Flask 2.3. Use 'FLASK_DEBUG' instead.
aws-bootcamp-cruddur-2023-backend-flask-1      | 'FLASK_ENV' is deprecated and will not be used in Flask 2.3. Use 'FLASK_DEBUG' instead.
aws-bootcamp-cruddur-2023-backend-flask-1      |  * Debugger is active!
aws-bootcamp-cruddur-2023-backend-flask-1      |  * Debugger PIN: 703-487-883
aws-bootcamp-cruddur-2023-db-1                 | ok
aws-bootcamp-cruddur-2023-frontend-react-js-1  | (node:44) [DEP_WEBPACK_DEV_SERVER_ON_AFTER_SETUP_MIDDLEWARE] DeprecationWarning: 'onAfterSetupMiddleware' option is deprecated. Please use the 'setupMiddlewares' option.
aws-bootcamp-cruddur-2023-frontend-react-js-1  | (Use `node --trace-deprecation ...` to show where the warning was created)
aws-bootcamp-cruddur-2023-frontend-react-js-1  | (node:44) [DEP_WEBPACK_DEV_SERVER_ON_BEFORE_SETUP_MIDDLEWARE] DeprecationWarning: 'onBeforeSetupMiddleware' option is deprecated. Please use the 'setupMiddlewares' option.
aws-bootcamp-cruddur-2023-frontend-react-js-1  | Starting the development server...
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-db-1                 | syncing data to disk ... initdb: warning: enabling "trust" authentication for local connections
aws-bootcamp-cruddur-2023-db-1                 | You can change this by editing pg_hba.conf or using the option -A, or
aws-bootcamp-cruddur-2023-db-1                 | --auth-local and --auth-host, the next time you run initdb.
aws-bootcamp-cruddur-2023-db-1                 | ok
aws-bootcamp-cruddur-2023-db-1                 | 
aws-bootcamp-cruddur-2023-db-1                 | 
aws-bootcamp-cruddur-2023-db-1                 | Success. You can now start the database server using:
aws-bootcamp-cruddur-2023-db-1                 | 
aws-bootcamp-cruddur-2023-db-1                 |     pg_ctl -D /var/lib/postgresql/data -l logfile start
aws-bootcamp-cruddur-2023-db-1                 | 
aws-bootcamp-cruddur-2023-db-1                 | waiting for server to start....2023-02-23 15:06:38.076 UTC [54] LOG:  starting PostgreSQL 13.10 on x86_64-pc-linux-musl, compiled by gcc (Alpine 12.2.1_git20220924-r4) 12.2.1 20220924, 64-bit
aws-bootcamp-cruddur-2023-db-1                 | 2023-02-23 15:06:38.095 UTC [54] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
aws-bootcamp-cruddur-2023-db-1                 | 2023-02-23 15:06:38.152 UTC [55] LOG:  database system was shut down at 2023-02-23 15:06:29 UTC
aws-bootcamp-cruddur-2023-db-1                 | 2023-02-23 15:06:38.177 UTC [54] LOG:  database system is ready to accept connections
aws-bootcamp-cruddur-2023-frontend-react-js-1  | Compiled with warnings.
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | Warning
aws-bootcamp-cruddur-2023-frontend-react-js-1  | (4:3) autoprefixer: start value has mixed support, consider using flex-start instead
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | Warning
aws-bootcamp-cruddur-2023-frontend-react-js-1  | (3:3) autoprefixer: start value has mixed support, consider using flex-start instead
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | Warning
aws-bootcamp-cruddur-2023-frontend-react-js-1  | (3:3) autoprefixer: start value has mixed support, consider using flex-start instead
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | [eslint] 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/App.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 13:8:  'process' is defined but never used  no-unused-vars
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/components/DesktopNavigationLink.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 18:5:  Expected a default case  default-case
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 21:9:  Unreachable code         no-unreachable
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 24:9:  Unreachable code         no-unreachable
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 27:9:  Unreachable code         no-unreachable
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 30:9:  Unreachable code         no-unreachable
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 33:9:  Unreachable code         no-unreachable
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/components/DesktopSidebar.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 41:9:  The href attribute requires a valid value to be accessible. Provide a valid, navigable address as the href value. If you cannot provide a valid href, but still need the element to resemble a link, use a button and change it with appropriate styles. Learn more: https://github.com/jsx-eslint/eslint-plugin-jsx-a11y/blob/HEAD/docs/rules/anchor-is-valid.md  jsx-a11y/anchor-is-valid
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 42:9:  The href attribute requires a valid value to be accessible. Provide a valid, navigable address as the href value. If you cannot provide a valid href, but still need the element to resemble a link, use a button and change it with appropriate styles. Learn more: https://github.com/jsx-eslint/eslint-plugin-jsx-a11y/blob/HEAD/docs/rules/anchor-is-valid.md  jsx-a11y/anchor-is-valid
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 43:9:  The href attribute requires a valid value to be accessible. Provide a valid, navigable address as the href value. If you cannot provide a valid href, but still need the element to resemble a link, use a button and change it with appropriate styles. Learn more: https://github.com/jsx-eslint/eslint-plugin-jsx-a11y/blob/HEAD/docs/rules/anchor-is-valid.md  jsx-a11y/anchor-is-valid
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/components/MessageGroupItem.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 27:23:  Expected '===' and instead saw '=='  eqeqeq
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/components/ProfileInfo.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 29:16:  Expected '===' and instead saw '=='  eqeqeq
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/components/ReplyForm.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 4:27:  'BombIcon' is defined but never used  no-unused-vars
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/components/TrendItem.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 13:5:  The href attribute requires a valid value to be accessible. Provide a valid, navigable address as the href value. If you cannot provide a valid href, but still need the element to resemble a link, use a button and change it with appropriate styles. Learn more: https://github.com/jsx-eslint/eslint-plugin-jsx-a11y/blob/HEAD/docs/rules/anchor-is-valid.md  jsx-a11y/anchor-is-valid
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/pages/ConfirmationPage.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 13:20:  'setCodeSent' is assigned a value but never used                                                                       no-unused-vars
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 67:6:   React Hook React.useEffect has a missing dependency: 'params.email'. Either include it or remove the dependency array  react-hooks/exhaustive-deps
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/pages/MessageGroupPage.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 16:10:  'popped' is assigned a value but never used                                                                                    no-unused-vars
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 75:6:   React Hook React.useEffect has a missing dependency: 'loadMessageGroupData'. Either include it or remove the dependency array  react-hooks/exhaustive-deps
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/pages/MessageGroupsPage.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 12:10:  'popped' is assigned a value but never used  no-unused-vars
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/pages/RecoverPage.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 12:18:   'setErrors' is assigned a value but never used     no-unused-vars
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 13:21:   'setFormState' is assigned a value but never used  no-unused-vars
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 118:17:  Expected '===' and instead saw '=='                eqeqeq
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 121:22:  Expected '===' and instead saw '=='                eqeqeq
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 124:22:  Expected '===' and instead saw '=='                eqeqeq
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/pages/SignupPage.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 16:18:  'setErrors' is assigned a value but never used  no-unused-vars
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/pages/UserFeedPage.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 57:6:  React Hook React.useEffect has a missing dependency: 'loadData'. Either include it or remove the dependency array  react-hooks/exhaustive-deps
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | Search for the keywords to learn more about each warning.
aws-bootcamp-cruddur-2023-frontend-react-js-1  | To ignore, add // eslint-disable-next-line to the line before.
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | WARNING in ./src/components/ActivityContent.css (./node_modules/css-loader/dist/cjs.js??ruleSet[1].rules[1].oneOf[5].use[1]!./node_modules/postcss-loader/dist/cjs.js??ruleSet[1].rules[1].oneOf[5].use[2]!./node_modules/source-map-loader/dist/cjs.js!./src/components/ActivityContent.css)
aws-bootcamp-cruddur-2023-frontend-react-js-1  | Module Warning (from ./node_modules/postcss-loader/dist/cjs.js):
aws-bootcamp-cruddur-2023-frontend-react-js-1  | Warning
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | (4:3) autoprefixer: start value has mixed support, consider using flex-start instead
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | WARNING in ./src/components/MessageGroupItem.css (./node_modules/css-loader/dist/cjs.js??ruleSet[1].rules[1].oneOf[5].use[1]!./node_modules/postcss-loader/dist/cjs.js??ruleSet[1].rules[1].oneOf[5].use[2]!./node_modules/source-map-loader/dist/cjs.js!./src/components/MessageGroupItem.css)
aws-bootcamp-cruddur-2023-frontend-react-js-1  | Module Warning (from ./node_modules/postcss-loader/dist/cjs.js):
aws-bootcamp-cruddur-2023-frontend-react-js-1  | Warning
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | (3:3) autoprefixer: start value has mixed support, consider using flex-start instead
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | WARNING in ./src/components/MessageItem.css (./node_modules/css-loader/dist/cjs.js??ruleSet[1].rules[1].oneOf[5].use[1]!./node_modules/postcss-loader/dist/cjs.js??ruleSet[1].rules[1].oneOf[5].use[2]!./node_modules/source-map-loader/dist/cjs.js!./src/components/MessageItem.css)
aws-bootcamp-cruddur-2023-frontend-react-js-1  | Module Warning (from ./node_modules/postcss-loader/dist/cjs.js):
aws-bootcamp-cruddur-2023-frontend-react-js-1  | Warning
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | (3:3) autoprefixer: start value has mixed support, consider using flex-start instead
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | WARNING in [eslint] 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/App.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 13:8:  'process' is defined but never used  no-unused-vars
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/components/DesktopNavigationLink.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 18:5:  Expected a default case  default-case
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 21:9:  Unreachable code         no-unreachable
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 24:9:  Unreachable code         no-unreachable
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 27:9:  Unreachable code         no-unreachable
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 30:9:  Unreachable code         no-unreachable
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 33:9:  Unreachable code         no-unreachable
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/components/DesktopSidebar.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 41:9:  The href attribute requires a valid value to be accessible. Provide a valid, navigable address as the href value. If you cannot provide a valid href, but still need the element to resemble a link, use a button and change it with appropriate styles. Learn more: https://github.com/jsx-eslint/eslint-plugin-jsx-a11y/blob/HEAD/docs/rules/anchor-is-valid.md  jsx-a11y/anchor-is-valid
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 42:9:  The href attribute requires a valid value to be accessible. Provide a valid, navigable address as the href value. If you cannot provide a valid href, but still need the element to resemble a link, use a button and change it with appropriate styles. Learn more: https://github.com/jsx-eslint/eslint-plugin-jsx-a11y/blob/HEAD/docs/rules/anchor-is-valid.md  jsx-a11y/anchor-is-valid
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 43:9:  The href attribute requires a valid value to be accessible. Provide a valid, navigable address as the href value. If you cannot provide a valid href, but still need the element to resemble a link, use a button and change it with appropriate styles. Learn more: https://github.com/jsx-eslint/eslint-plugin-jsx-a11y/blob/HEAD/docs/rules/anchor-is-valid.md  jsx-a11y/anchor-is-valid
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/components/MessageGroupItem.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 27:23:  Expected '===' and instead saw '=='  eqeqeq
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/components/ProfileInfo.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 29:16:  Expected '===' and instead saw '=='  eqeqeq
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/components/ReplyForm.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 4:27:  'BombIcon' is defined but never used  no-unused-vars
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/components/TrendItem.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 13:5:  The href attribute requires a valid value to be accessible. Provide a valid, navigable address as the href value. If you cannot provide a valid href, but still need the element to resemble a link, use a button and change it with appropriate styles. Learn more: https://github.com/jsx-eslint/eslint-plugin-jsx-a11y/blob/HEAD/docs/rules/anchor-is-valid.md  jsx-a11y/anchor-is-valid
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/pages/ConfirmationPage.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 13:20:  'setCodeSent' is assigned a value but never used                                                                       no-unused-vars
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 67:6:   React Hook React.useEffect has a missing dependency: 'params.email'. Either include it or remove the dependency array  react-hooks/exhaustive-deps
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/pages/MessageGroupPage.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 16:10:  'popped' is assigned a value but never used                                                                                    no-unused-vars
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 75:6:   React Hook React.useEffect has a missing dependency: 'loadMessageGroupData'. Either include it or remove the dependency array  react-hooks/exhaustive-deps
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/pages/MessageGroupsPage.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 12:10:  'popped' is assigned a value but never used  no-unused-vars
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/pages/RecoverPage.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 12:18:   'setErrors' is assigned a value but never used     no-unused-vars
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 13:21:   'setFormState' is assigned a value but never used  no-unused-vars
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 118:17:  Expected '===' and instead saw '=='                eqeqeq
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 121:22:  Expected '===' and instead saw '=='                eqeqeq
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 124:22:  Expected '===' and instead saw '=='                eqeqeq
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/pages/SignupPage.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 16:18:  'setErrors' is assigned a value but never used  no-unused-vars
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | src/pages/UserFeedPage.js
aws-bootcamp-cruddur-2023-frontend-react-js-1  |   Line 57:6:  React Hook React.useEffect has a missing dependency: 'loadData'. Either include it or remove the dependency array  react-hooks/exhaustive-deps
aws-bootcamp-cruddur-2023-frontend-react-js-1  | 
aws-bootcamp-cruddur-2023-frontend-react-js-1  | webpack compiled with 4 warnings
aws-bootcamp-cruddur-2023-db-1                 |  done
aws-bootcamp-cruddur-2023-db-1                 | server started
aws-bootcamp-cruddur-2023-db-1                 | 
aws-bootcamp-cruddur-2023-db-1                 | /usr/local/bin/docker-entrypoint.sh: ignoring /docker-entrypoint-initdb.d/*
aws-bootcamp-cruddur-2023-db-1                 | 
aws-bootcamp-cruddur-2023-db-1                 | waiting for server to shut down....2023-02-23 15:06:38.238 UTC [54] LOG:  received fast shutdown request
aws-bootcamp-cruddur-2023-db-1                 | 2023-02-23 15:06:38.258 UTC [54] LOG:  aborting any active transactions
aws-bootcamp-cruddur-2023-db-1                 | 2023-02-23 15:06:38.260 UTC [54] LOG:  background worker "logical replication launcher" (PID 61) exited with exit code 1
aws-bootcamp-cruddur-2023-db-1                 | 2023-02-23 15:06:38.260 UTC [56] LOG:  shutting down
aws-bootcamp-cruddur-2023-db-1                 | 2023-02-23 15:06:38.390 UTC [54] LOG:  database system is shut down
aws-bootcamp-cruddur-2023-db-1                 |  done
aws-bootcamp-cruddur-2023-db-1                 | server stopped
aws-bootcamp-cruddur-2023-db-1                 | 
aws-bootcamp-cruddur-2023-db-1                 | PostgreSQL init process complete; ready for start up.
aws-bootcamp-cruddur-2023-db-1                 | 
aws-bootcamp-cruddur-2023-db-1                 | 2023-02-23 15:06:38.489 UTC [1] LOG:  starting PostgreSQL 13.10 on x86_64-pc-linux-musl, compiled by gcc (Alpine 12.2.1_git20220924-r4) 12.2.1 20220924, 64-bit
aws-bootcamp-cruddur-2023-db-1                 | 2023-02-23 15:06:38.490 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
aws-bootcamp-cruddur-2023-db-1                 | 2023-02-23 15:06:38.490 UTC [1] LOG:  listening on IPv6 address "::", port 5432
aws-bootcamp-cruddur-2023-db-1                 | 2023-02-23 15:06:38.527 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
aws-bootcamp-cruddur-2023-db-1                 | 2023-02-23 15:06:38.569 UTC [67] LOG:  database system was shut down at 2023-02-23 15:06:38 UTC
aws-bootcamp-cruddur-2023-db-1                 | 2023-02-23 15:06:38.593 UTC [1] LOG:  database system is ready to accept connections
```

![Docker compose up](assets/week-1/Docker%20compose%20up.png)

or by using VSCode extension, right click and Compose Up

![Compose up](assets/week-1/Compose%20up.png)

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

It works!

![Browse frontend 2](assets/week-1/Browse%20frontend%202.png)

I could make it work by manually running npm install on frontend folder before starting compose file

https://3000-alvarezdani-awsbootcamp-bpzo93rvx4h.ws-us87.gitpod.io/


> Note on docker compose vs docker-compose, James explained docker plugins or extension works this way, same as kubectl in kubernetes


Test a change in FrontEnd code while compose is running, and it refreshes


## Document the Notification Endpoint for the OpenAPI Document

In backend-flask/openapi-3.0.yml file, add the definition for notification endpoint:

```yaml
  /api/activities/notifications:
    get:
      description: 'Return a feed of activity for all of those that I follow'
      tags:
        - activities
      parameters: []
      responses:
        '200':
          description: Returns an array of activities
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Activity'
```

Let's check the definition in OpenAPI VSCode extension.

![OpenAPI notifications](assets/week-1/OpenAPI%20notifications.png)

## Write a Flask Backend Endpoint for Notifications

Change backend app, adding new endpoint for notifications.

In backend-flask/app.py, add route to new endpoint

```py
@app.route("/api/activities/notifications", methods=['GET'])
def data_notifications():
  data = NotificationsActivities.run()
  return data, 200
```

Add new service to backend-flask/services/notifications_activities.py

```py
from datetime import datetime, timedelta, timezone
class NotificationsActivities:
  def run():
    now = datetime.now(timezone.utc).astimezone()
    results = [{
      'uuid': '68f126b0-1ceb-4a33-88be-d90fa7109eee',
      'handle':  'coco',
      'message': 'I am white unicorn',
      'created_at': (now - timedelta(days=2)).isoformat(),
      'expires_at': (now + timedelta(days=5)).isoformat(),
      'likes_count': 5,
      'replies_count': 1,
      'reposts_count': 0,
      'replies': [{
        'uuid': '26e12864-1c26-5c3a-9658-97a10f8fea67',
        'reply_to_activity_uuid': '68f126b0-1ceb-4a33-88be-d90fa7109eee',
        'handle':  'worf',
        'message': 'this post has no honor!',
        'likes_count': 0,
        'replies_count': 0,
        'reposts_count': 0,
        'created_at': (now - timedelta(days=2)).isoformat()
      }],
    }
    ]
    return results
```

In backend-flask/app.py, add import to new service

```py
from services.notifications_activities import *
```

Launch Gitpod workspace and test new endpoint by starting compose:

GET https://4567-alvarezdani-awsbootcamp-ludvad7g3bm.ws-us87.gitpod.io/api/activities/notifications

```json
[
  {
    "created_at": "2023-02-18T23:45:09.638418+00:00",
    "expires_at": "2023-02-25T23:45:09.638418+00:00",
    "handle": "coco",
    "likes_count": 5,
    "message": "I am white unicorn",
    "replies": [
      {
        "created_at": "2023-02-18T23:45:09.638418+00:00",
        "handle": "worf",
        "likes_count": 0,
        "message": "this post has no honor!",
        "replies_count": 0,
        "reply_to_activity_uuid": "68f126b0-1ceb-4a33-88be-d90fa7109eee",
        "reposts_count": 0,
        "uuid": "26e12864-1c26-5c3a-9658-97a10f8fea67"
      }
    ],
    "replies_count": 1,
    "reposts_count": 0,
    "uuid": "68f126b0-1ceb-4a33-88be-d90fa7109eee"
  }
]
```

![Browse notifications endpoint](assets/week-1/Browse%20notifications%20endpoint.png)

## Write a React Page for Notifications

In frontend-react-js/src/App.js, add route to new Notifications page:

```js
  {
    path: "/notifications",
    element: <NotificationsFeedPage />
  }
```

Add new page frontend-react-js/src/pages/NotificationsFeedPage.js to implement view for notifications:

```js
import './NotificationsFeedPage.css';
import React from "react";

import DesktopNavigation  from '../components/DesktopNavigation';
import DesktopSidebar     from '../components/DesktopSidebar';
import ActivityFeed from '../components/ActivityFeed';
import ActivityForm from '../components/ActivityForm';
import ReplyForm from '../components/ReplyForm';

// [TODO] Authenication
import Cookies from 'js-cookie'

export default function NotificationsFeedPage() {
  const [activities, setActivities] = React.useState([]);
  const [popped, setPopped] = React.useState(false);
  const [poppedReply, setPoppedReply] = React.useState(false);
  const [replyActivity, setReplyActivity] = React.useState({});
  const [user, setUser] = React.useState(null);
  const dataFetchedRef = React.useRef(false);

  const loadData = async () => {
    try {
      const backend_url = `${process.env.REACT_APP_BACKEND_URL}/api/activities/notifications`
      const res = await fetch(backend_url, {
        method: "GET"
      });
      let resJson = await res.json();
      if (res.status === 200) {
        setActivities(resJson)
      } else {
        console.log(res)
      }
    } catch (err) {
      console.log(err);
    }
  };

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

  React.useEffect(()=>{
    //prevents double call
    if (dataFetchedRef.current) return;
    dataFetchedRef.current = true;

    loadData();
    checkAuth();
  }, [])

  return (
    <article>
      <DesktopNavigation user={user} active={'notifications'} setPopped={setPopped} />
      <div className='content'>
        <ActivityForm  
          popped={popped}
          setPopped={setPopped} 
          setActivities={setActivities} 
        />
        <ReplyForm 
          activity={replyActivity} 
          popped={poppedReply} 
          setPopped={setPoppedReply} 
          setActivities={setActivities} 
          activities={activities} 
        />
        <ActivityFeed 
          title="Notifications" 
          setReplyActivity={setReplyActivity} 
          setPopped={setPoppedReply} 
          activities={activities} 
        />
      </div>
      <DesktopSidebar user={user} />
    </article>
  );
}
```

Add referenced css file in notifications page, frontend-react-js/src/pages/NotificationsFeedPage.css (empty content):

```css

```

Finally, in frontend-react-js/src/App.js, add import to new Notifications page:

```js
import NotificationsFeedPage from './pages/NotificationsFeedPage';
```

Test changes by starting Gitpod workspace and running compose file:

Browse https://3000-alvarezdani-awsbootcamp-ludvad7g3bm.ws-us87.gitpod.io/notifications

Returns correct information

![Browse notifications page](assets/week-1/Browse%20notifications%20page.png)

or register a Cruddur account and click on notifications option

![Browse notifications page 2](assets/week-1/Browse%20notifications%20page%202.png)

## Run DynamoDB Local Container and ensure it works

For checking how to run a local DynamoDB container, I've opened the following page as a reference:
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.DownloadingAndRunning.html

So in docker-compose.yml file, I've added the new service. Also, as shown by Andrew in his example, root user was used to get it working:

```yml
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

In this case the volume is used by directory volume mapping

```yml
    volumes:
      - "./docker/dynamodb:/home/dynamodblocal/data"
```

Start compose file, and unlock port 8000.

![Run dynamodb](assets/week-1/Run%20dynamodb.png)

For ensuring it works, I've started the compose file and used some AWS CLI commands to check DynamoDB, extracted from the following repo:
https://github.com/100DaysOfCloud/challenge-dynamodb-local


Create a table

```sh
aws dynamodb create-table \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --attribute-definitions \
        AttributeName=Artist,AttributeType=S \
        AttributeName=SongTitle,AttributeType=S \
    --key-schema AttributeName=Artist,KeyType=HASH AttributeName=SongTitle,KeyType=RANGE \
    --provisioned-throughput ReadCapacityUnits=1,WriteCapacityUnits=1 \
    --table-class STANDARD
```

returns

```json
{
    "TableDescription": {
        "AttributeDefinitions": [
            {
                "AttributeName": "Artist",
                "AttributeType": "S"
            },
            {
                "AttributeName": "SongTitle",
                "AttributeType": "S"
            }
        ],
        "TableName": "Music",
        "KeySchema": [
            {
                "AttributeName": "Artist",
                "KeyType": "HASH"
            },
            {
                "AttributeName": "SongTitle",
                "KeyType": "RANGE"
            }
        ],
        "TableStatus": "ACTIVE",
        "CreationDateTime": "2023-02-22T15:24:07.099000+00:00",
        "ProvisionedThroughput": {
            "LastIncreaseDateTime": "1970-01-01T00:00:00+00:00",
            "LastDecreaseDateTime": "1970-01-01T00:00:00+00:00",
            "NumberOfDecreasesToday": 0,
            "ReadCapacityUnits": 1,
            "WriteCapacityUnits": 1
        },
        "TableSizeBytes": 0,
        "ItemCount": 0,
        "TableArn": "arn:aws:dynamodb:ddblocal:000000000000:table/Music"
    }
}
```

![Create table](assets/week-1/Create%20table.png)

Create an item

```sh
aws dynamodb put-item \
    --endpoint-url http://localhost:8000 \
    --table-name Music \
    --item \
        '{"Artist": {"S": "No One You Know"}, "SongTitle": {"S": "Call Me Today"}, "AlbumTitle": {"S": "Somewhat Famous"}}' \
    --return-consumed-capacity TOTAL 
```

return:

```json
{
    "ConsumedCapacity": {
        "TableName": "Music",
        "CapacityUnits": 1.0
    }
}
```

![Create item](assets/week-1/Create%20item.png)

List tables

```sh
aws dynamodb list-tables --endpoint-url http://localhost:8000
```

return

```json
{
    "TableNames": [
        "Music"
    ]
}
```

![List tables](assets/week-1/List%20tables.png)

Get Records

```sh
aws dynamodb scan --table-name Music --query "Items" --endpoint-url http://localhost:8000
```

return

```json
[
    {
        "Artist": {
            "S": "No One You Know"
        },
        "SongTitle": {
            "S": "Call Me Today"
        },
        "AlbumTitle": {
            "S": "Somewhat Famous"
        }
    }
]
```

![Get records](assets/week-1/Get%20records.png)

References:
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html 
https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Tools.CLI.html

## Run Postgres Container and ensure it works

For running a PostgreSQL image, I've added a new service to docker compose file,

```yml
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

Also, as this image references a volume, it should be added as well (in this case using a named volume mapping):

```yml
volumes:
  db:
    driver: local
```

![Run postgres](assets/week-1/Run%20postgres.png)

For insuring that the new image is working, a Postgresql driver must be installed, so I've used a Gitpod task to install it when environment starts.

.gitpod.yml
```yml
```

Then start compose file so Postgresql service is running, and check it:

```sh
psql -h localhost -U postgres
```

![Psql](assets/week-1/Psql.png)

```
\d
```

Did not find any relations.

![Postgres d](assets/week-1/Postgres%20d.png)

```
\t
```

Tuples only is on.

![Postgres t](assets/week-1/Postgres%20t.png)

```
\dl
```

      Large objects
 ID | Owner | Description 
----+-------+-------------
(0 rows)

![Postgres dl](assets/week-1/Postgres%20dl.png)

```
\l
```

 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 | 
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres

![Postgres l](assets/week-1/Postgres%20l.png)

```
\q
```
To exit

These commands indicate that Postgresql is up and running.

![Postgres](assets/week-1/Postgres.png)

Another way to check it can be by installing a VSCode extension to browse the service. That extension is called PostgreSQL, and it can be added also to .gitpod.yaml file.

```yml
vscode:
  extensions:
    - 42Crunch.vscode-openapi
    - cweijan.vscode-postgresql-client2
```

![Postgres extensions](assets/week-1/Postgres%20extensions.png)

![Postgres extensions 2](assets/week-1/Postgres%20extensions%202.png)

## Homework Challenges 

### Push and tag a image to DockerHub (they have a free tier)

I have windows 10 system with WSL2 and docker already installed:

```sh
 developer@N059D5BBC  aws-bootcamp-cruddur-2023  bash  no config    docker --version
Docker version 20.10.23, build 7155243
```

So, first step is to try to build the image locally:

```sh
docker build -t  backend-flask ./backend-flask
Sending build context to Docker daemon  36.35kB
Step 1/8 : FROM python:3.10-slim-buster
 ---> b5d627f77479
Step 2/8 : WORKDIR /backend-flask
 ---> Using cache
 ---> dec27c3e2b93
Step 3/8 : COPY requirements.txt requirements.txt
 ---> Using cache
 ---> da85902f7ee8
Step 4/8 : RUN pip3 install -r requirements.txt
 ---> Using cache
 ---> c8ea9985ae09
Step 5/8 : COPY . .
 ---> Using cache
 ---> 7207314c405a
Step 6/8 : ENV FLASK_ENV=development
 ---> Using cache
 ---> 8a94974b5f9e
Step 7/8 : EXPOSE ${PORT}
 ---> Using cache
 ---> e0946a0add56
Step 8/8 : CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]
 ---> Using cache
 ---> bd8e097c7bfb
Successfully built bd8e097c7bfb
Successfully tagged backend-flask:latest
```
![Build backend image](assets/week-1/Build%20backend%20image.png)

Then I can verify if the image is available:

```sh
docker images | grep flask
```
![Docker images backend](assets/week-1/Docker%20images%20backend.png)

Then I'll log into Docker Hub and search for instruction to push a custom image to the repository:

- Create a Docker ID ==> done = alvarezdaniel
- Create a repository ==> done = alvarezdaniel/aws-bootcamp (https://hub.docker.com/repository/docker/alvarezdaniel/aws-bootcamp/general)

![Docker repository](assets/week-1/Docker%20repository.png)

To be able to push the built image to the new created repository, it must be rebuilt using a different repository:

```sh
docker build -t  alvarezdaniel/aws-bootcamp/backend-flask:1.0.0 ./backend-flask
```

Then again, docker images can be called in order to check built image:

```sh
docker images | grep back
```

![Docker images backend 2](assets/week-1/Docker%20images%20backend%202.png)

Finally, the image can be pushed to Docker Hub

```sh
docker push alvarezdaniel/aws-bootcamp/backend-flask:1.0.0
```

![Docker push failed](assets/week-1/Docker%20push%20failed.png)

Searching for that error throwed that authentication is missing with Docker Hub:

```sh
docker login -u MY_USER -p MY_PASSWORD
```

![Docker login](assets/week-1/Docker%20login.png)

I've found that after doing that, docker push still was failing, so I kept investigating, and discovered that repository name has to be the same as the image name locally, so I've run this command to create a new image locally:

```sh
docker build -t  alvarezdaniel/backend-flask:1.0.0 ./backend-flask
```

![Docker build 2](assets/week-1/Docker%20build%202.png)

And after that, I could successfully pushed the image to Docker Hub:

```sh
docker push alvarezdaniel/backend-flask:1.0.0
```

![Docker push](assets/week-1/Docker%20push.png)

Then I was able to see the pushed image in the online repository:

![Docker repository 2](assets/week-1/Docker%20repository%202.png)

