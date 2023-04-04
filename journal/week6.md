# Week 6/7 — Serverless Containers


## Week 6/7 instructors

- Andrew Brown [@andrewbrown](https://twitter.com/andrewbrown)
- Shala Warner [@GiftedLane](https://twitter.com/GiftedLane) (Student Advocate)

## Class Summary

- Deploy an ECS Cluster using ECS Service Connect
- Deploy serverless containers using Fargate for the Backend and Frontend Application
- Route traffic to the frontend and backend on different subdomains using Application Load Balancer
- Securing our flask container
- Creating several bash utility scripts to easily work with serverless containers.

## Todo Checklist

### Watch ECS Security by Ashish

https://www.youtube.com/watch?v=zz2FQAk1I28&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=57

### Watch Fargate Technical Questions with Maish


### Provision ECS Cluster

https://www.youtube.com/watch?v=QIZx2NhdCMI&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=58


#### Fargate pricing information

AWS Fargate pricing - https://aws.amazon.com/fargate/pricing/

ChatGPT query:

```
using the information below, calculate the cost of running two containers with the lowest memory and compute on ECS Fargate

per vCPU per hour	$0.013368
per GB per hour	$0.0014595

project the cost for 10 hours, 20 hours, 40 hours, 730 hours

format it on a table
```

Response

```
Using the information provided, I can estimate the cost of running two containers with the lowest memory and compute on ECS Fargate.

Assuming the lowest memory and compute for a container is 0.25GB and 0.25 vCPU respectively, the cost for running one container per hour would be:

CPU cost: 0.25 vCPU x $0.013368 per vCPU per hour = $0.003342 per hour
Memory cost: 0.25 GB x $0.0014595 per GB per hour = $0.000364875 per hour
Total cost: $0.003342 + $0.000364875 = $0.003706875 per hour
Therefore, the cost of running two containers with the lowest memory and compute on ECS Fargate for one hour would be:

Total cost: 2 x $0.003706875 = $0.00741375 per hour
Keep in mind that this is just an estimate, and other factors such as the region and duration of the containers could impact the final cost.

Here's a table with the projected costs for running two containers with the lowest memory and compute on ECS Fargate for different durations:

Duration (hours)	Cost ($)
10	0.0741375
20	0.148275
40	0.29655
730	5.4052875
Again, these are just estimates and the actual costs may vary depending on various factors.
```

We are going to run it and then shut it down, so we are not going to consume too much credit

#### Week 6 instructions 

We are going to follow instructions from this page: https://github.com/omenking/aws-bootcamp-cruddur-2023/blob/week-6-fargate/journal/week6.md

#### Implement db test script

First, we are going to implement a bash script for testing db connection from the container

`backend-flask/bin/db/test`

```py
#!/usr/bin/env python3

import psycopg
import os
import sys

connection_url = os.getenv("CONNECTION_URL")

conn = None
try:
  print('attempting connection')
  conn = psycopg.connect(connection_url)
  print("Connection successful!")
except psycopg.Error as e:
  print("Unable to connect to the database:", e)
finally:
  conn.close()
```

> We need to set execute permissions on this script to be able to execute it

> Script uses CONNECTION_URL for local db, and it needs to be replaced with PROD_CONNECTION_URL for AWS RDS instance

Let's test it with the local DB container

```sh
./bin/db/test
```

```
attempting connection
Connection successful!
```

#### Implement health check endpoint

We are going to implement a health-check endpoint in flask app

`app.py`

```py
# Health-check endpoint
@app.route('/api/health-check')
def health_check():
  return {'success': True}, 200
```






