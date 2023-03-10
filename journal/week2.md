# Week 2 — Distributed Tracing

## Introduction

Andrew Brown hosts AWS Ontario Virtual User Group

Sponsors:
- Adrian Cantrill = Free docker fundamentals courses
- WeCloudData = free videos

AWS user groups around the world

## Week 2 instructors

- Andrew Brown [@andrewbrown](https://twitter.com/andrewbrown)
- Shala Warner [@GiftedLane](https://twitter.com/GiftedLane)
- Jessica Joy Kerr [@jessitron](https://twitter.com/jessitron)

## New week

- Program = instructions (in the past, programs log history of what happened in text files, now they use distributed tracing)

## Class Summary

- Instrument our backend flask application to use Open Telemetry (OTEL) with Honeycomb.io as the provider
- Run queries to explore traces within Honeycomb.io
- Instrument AWS X-Ray into backend flask application
- Configure and provision X-Ray daemon within docker-compose and send data back to X-Ray API
- Observe X-Ray traces within the AWS Console
- Integrate Rollbar for Error Logging
- Trigger an error an observe an error with Rollbar
- Install WatchTower and write a custom logger to send application log data to CloudWatch Log group

## Todo Checklist

### Watch Week 2 Live-Stream Video

- https://www.youtube.com/watch?v=2GD9xCzRId4&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=30

### Watch Chirag Week 2 - Spending Considerations (Coming Soon)

AWS Bootcamp Week 2 - Honeycomb, Rollbar, AWS X-Ray and AWS Cloudwatch Logs pricing considerations

https://www.youtube.com/watch?v=2W3KeqCjtDY&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=38

In this week's video, let's understand the free tier for honeycomb io, Rollbar, Cloudwatch logs, and AWS Xray.

Links from the video:

- Honeycomb pricing: https://www.honeycomb.io/pricing
- Honeycomb usage: https://ui.honeycomb.io/teams/alvarez.daniel/usage
- Rollbar pricing: https://rollbar.com/pricing/
- AWS X-Ray pricing: https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all&all-free-tier.q=x%2Bray&all-free-tier.q_operator=AND
- AWS CloudWatch pricing: https://aws.amazon.com/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all&all-free-tier.q=cloudwatch&all-free-tier.q_operator=AND
- AWS X-Ray pricing: https://aws.amazon.com/xray/pricing/
- AWS CloudWatch pricing: https://aws.amazon.com/cloudwatch/pricing/

### Watch Ashish's Week 2 - Observability Security Considerations

Observability vs Monitoring Explained in AWS

https://www.youtube.com/watch?v=bOf4ITxAcXc&list=PLBfufR7vyJJ7k25byhRXJldB5AiwgNnWv&index=31

Security observability explained with Best practices for SOC teams, DevOps Teams in this week's Cloud Bootcamp with security and speed in mind

Links from the video:
 - Amazon CloudWatch releasing Services - https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/aws-services-cloudwatch-metrics.html

### Instrument Honeycomb with OTEL

- Implement tracing using honeycomb: https://www.honeycomb.io/
- Each row in honeycomb is a span (single unit of work, has a duration)
- Traces = what happened and when
- Useful for detecting places to optimize
- Modern Standard
- Instrumentation

#### Instrument backend app

- Create honeycomb account and set it up
- Create honeycomb environment called "bootcamp"

![](assets/week-2/01-honeycomb-environment.png)

- Copy bootcamp environment api key for setting it as an environment variable

![](assets/week-2/02-honeycomb-apikey.png)

- Add api key to environment and gitpod

![](assets/week-2/03-set-honeycomb-env.png)

- Add requirements to pip for adding honeycomb support

```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```

- Install requirements

```sh
pip install -r requirements.txt
```

```
Collecting flask
  Downloading Flask-2.2.3-py3-none-any.whl (101 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 101.8/101.8 kB 3.0 MB/s eta 0:00:00
Collecting flask-cors
  Downloading Flask_Cors-3.0.10-py2.py3-none-any.whl (14 kB)
Collecting opentelemetry-api
  Downloading opentelemetry_api-1.16.0-py3-none-any.whl (57 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 57.3/57.3 kB 2.1 MB/s eta 0:00:00
Collecting opentelemetry-sdk
  Downloading opentelemetry_sdk-1.16.0-py3-none-any.whl (94 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 94.6/94.6 kB 4.4 MB/s eta 0:00:00
Collecting opentelemetry-exporter-otlp-proto-http
  Downloading opentelemetry_exporter_otlp_proto_http-1.16.0-py3-none-any.whl (21 kB)
Collecting opentelemetry-instrumentation-flask
  Downloading opentelemetry_instrumentation_flask-0.37b0-py3-none-any.whl (13 kB)
Collecting opentelemetry-instrumentation-requests
  Downloading opentelemetry_instrumentation_requests-0.37b0-py3-none-any.whl (11 kB)
Collecting itsdangerous>=2.0
  Downloading itsdangerous-2.1.2-py3-none-any.whl (15 kB)
Collecting click>=8.0
  Downloading click-8.1.3-py3-none-any.whl (96 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 96.6/96.6 kB 4.0 MB/s eta 0:00:00
Requirement already satisfied: Jinja2>=3.0 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from flask->-r requirements.txt (line 1)) (3.1.2)
Collecting Werkzeug>=2.2.2
  Downloading Werkzeug-2.2.3-py3-none-any.whl (233 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 233.6/233.6 kB 9.2 MB/s eta 0:00:00
Requirement already satisfied: importlib-metadata>=3.6.0 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from flask->-r requirements.txt (line 1)) (6.0.0)
Requirement already satisfied: Six in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from flask-cors->-r requirements.txt (line 2)) (1.16.0)
Requirement already satisfied: setuptools>=16.0 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from opentelemetry-api->-r requirements.txt (line 4)) (65.6.3)
Collecting deprecated>=1.2.6
  Downloading Deprecated-1.2.13-py2.py3-none-any.whl (9.6 kB)
Requirement already satisfied: typing-extensions>=3.7.4 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from opentelemetry-sdk->-r requirements.txt (line 5)) (4.4.0)
Collecting opentelemetry-semantic-conventions==0.37b0
  Downloading opentelemetry_semantic_conventions-0.37b0-py3-none-any.whl (26 kB)
Collecting opentelemetry-proto==1.16.0
  Downloading opentelemetry_proto-1.16.0-py3-none-any.whl (52 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 52.6/52.6 kB 2.4 MB/s eta 0:00:00
Collecting googleapis-common-protos~=1.52
  Downloading googleapis_common_protos-1.58.0-py2.py3-none-any.whl (223 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 223.0/223.0 kB 10.0 MB/s eta 0:00:00
Requirement already satisfied: requests~=2.7 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from opentelemetry-exporter-otlp-proto-http->-r requirements.txt (line 6)) (2.28.1)
Collecting backoff<3.0.0,>=1.10.0
  Downloading backoff-2.2.1-py3-none-any.whl (15 kB)
Collecting protobuf<5.0,>=3.19
  Downloading protobuf-4.22.0-cp37-abi3-manylinux2014_x86_64.whl (302 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 302.4/302.4 kB 14.3 MB/s eta 0:00:00
Collecting opentelemetry-util-http==0.37b0
  Downloading opentelemetry_util_http-0.37b0-py3-none-any.whl (6.7 kB)
Collecting opentelemetry-instrumentation-wsgi==0.37b0
  Downloading opentelemetry_instrumentation_wsgi-0.37b0-py3-none-any.whl (12 kB)
Collecting opentelemetry-instrumentation==0.37b0
  Downloading opentelemetry_instrumentation-0.37b0-py3-none-any.whl (24 kB)
Requirement already satisfied: wrapt<2.0.0,>=1.0.0 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from opentelemetry-instrumentation==0.37b0->opentelemetry-instrumentation-flask->-r requirements.txt (line 7)) (1.14.1)
Requirement already satisfied: zipp>=0.5 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from importlib-metadata>=3.6.0->flask->-r requirements.txt (line 1)) (3.11.0)
Requirement already satisfied: MarkupSafe>=2.0 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from Jinja2>=3.0->flask->-r requirements.txt (line 1)) (2.1.1)
Requirement already satisfied: charset-normalizer<3,>=2 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from requests~=2.7->opentelemetry-exporter-otlp-proto-http->-r requirements.txt (line 6)) (2.1.1)
Requirement already satisfied: urllib3<1.27,>=1.21.1 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from requests~=2.7->opentelemetry-exporter-otlp-proto-http->-r requirements.txt (line 6)) (1.26.13)
Requirement already satisfied: certifi>=2017.4.17 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from requests~=2.7->opentelemetry-exporter-otlp-proto-http->-r requirements.txt (line 6)) (2022.12.7)
Requirement already satisfied: idna<4,>=2.5 in /home/gitpod/.pyenv/versions/3.8.16/lib/python3.8/site-packages (from requests~=2.7->opentelemetry-exporter-otlp-proto-http->-r requirements.txt (line 6)) (3.4)
Installing collected packages: Werkzeug, protobuf, opentelemetry-util-http, opentelemetry-semantic-conventions, itsdangerous, deprecated, click, backoff, opentelemetry-proto, opentelemetry-api, googleapis-common-protos, flask, opentelemetry-sdk, opentelemetry-instrumentation, flask-cors, opentelemetry-instrumentation-wsgi, opentelemetry-instrumentation-requests, opentelemetry-exporter-otlp-proto-http, opentelemetry-instrumentation-flask
Successfully installed Werkzeug-2.2.3 backoff-2.2.1 click-8.1.3 deprecated-1.2.13 flask-2.2.3 flask-cors-3.0.10 googleapis-common-protos-1.58.0 itsdangerous-2.1.2 opentelemetry-api-1.16.0 opentelemetry-exporter-otlp-proto-http-1.16.0 opentelemetry-instrumentation-0.37b0 opentelemetry-instrumentation-flask-0.37b0 opentelemetry-instrumentation-requests-0.37b0 opentelemetry-instrumentation-wsgi-0.37b0 opentelemetry-proto-1.16.0 opentelemetry-sdk-1.16.0 opentelemetry-semantic-conventions-0.37b0 opentelemetry-util-http-0.37b0 protobuf-4.22.0

[notice] A new release of pip available: 22.3.1 -> 23.0.1
[notice] To update, run: pip install --upgrade pip
```

- Add honeycomb imports to app.py

```py
# HoneyComb imports
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
```

- Add honeycomb initialization to app.py

```py
# HoneyComb initialization: initialize tracer and an exporter for sending data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)
```

- Add automatic instrumentation (after app initialization)

```py
# HoneyComb automatic instrumentation for flask
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```

- Add environment variables to compose file

```yaml
services:
  backend-flask:
    environment:
      OTEL_SERVICE_NAME: 'backend-flask'
      OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
      OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
```

![](assets/week-2/04-honeycomb-env-gitpod.png)

- OTEL_SERVICE_NAME: Sets the value of the service.name resource attribute (https://opentelemetry.io/docs/concepts/sdk-configuration/otlp-exporter-configuration/#otel_exporter_otlp_headers)
- OTEL_EXPORTER_OTLP_ENDPOINT: A base endpoint URL for any signal type, with an optionally-specified port number. Helpful for when you’re sending more than one signal to the same endpoint and want one environment variable to control the endpoint (https://opentelemetry.io/docs/concepts/sdk-configuration/otlp-exporter-configuration/#otel_exporter_otlp_headers)
- OTEL_EXPORTER_OTLP_HEADERS: A list of headers to apply to all outgoing data (traces, metrics, and logs) (https://opentelemetry.io/docs/concepts/sdk-configuration/otlp-exporter-configuration/#otel_exporter_otlp_headers)

- Run compose file to test it

```sh
docker compose  -f "docker-compose.yml" up -d
```

- Browse activities endpoint so as to generate tracing activity: 
https://4567-alvarezdani-awsbootcamp-ismmdqr3qis.ws-us88.gitpod.io/api/activities/home

```
[
  {
    "created_at": "2023-02-25T17:23:30.969884+00:00",
    "expires_at": "2023-03-04T17:23:30.969884+00:00",
    "handle": "Andrew Brown",
    "likes_count": 5,
    "message": "Cloud is fun!",
    "replies": [
      {
        "created_at": "2023-02-25T17:23:30.969884+00:00",
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
    "created_at": "2023-02-20T17:23:30.969884+00:00",
    "expires_at": "2023-03-08T17:23:30.969884+00:00",
    "handle": "Worf",
    "likes": 0,
    "message": "I am out of prune juice",
    "replies": [],
    "uuid": "66e12864-8c26-4c3a-9658-95a10f8fea67"
  },
  {
    "created_at": "2023-02-27T16:23:30.969884+00:00",
    "expires_at": "2023-02-28T05:23:30.969884+00:00",
    "handle": "Garek",
    "likes": 0,
    "message": "My dear doctor, I am just simple tailor",
    "replies": [],
    "uuid": "248959df-3079-4947-b847-9e0892d1bab4"
  }
]
```

- Browse in Honeycomb Datasets page for bootcamp environment: https://ui.honeycomb.io/alvarez.daniel/environments/bootcamp/datasets

![](assets/week-2/05-honeycomb-dataset.png)

- Create a new query to check information in this dataset: https://ui.honeycomb.io/alvarez.daniel/environments/bootcamp/datasets/backend-flask

![](assets/week-2/06-honeycomb-query.png)

- And view the generated trace:

![](assets/week-2/07-honeycomb-trace.png)

- For required ports to be opened on workspace start, add this code to .gitpod.yml file:

```yml
ports:
  - name: frontend
    port: 3000
    onOpen: open-browser
    visibility: public
  - name: backend
    port: 4567
    visibility: public
  - name: xray-daemon
    port: 2000
    visibility: public
```

- If any issue while sending data to honeycomb, there is the possibility to add console tracing as well:

```py
# HoneyComb imports
from opentelemetry.sdk.trace.export import ConsoleSpanExporter, SimpleSpanProcessor

# HoneyComb: show this in the logs within the backend-flask app (STDOUT)
simple_processor = SimpleSpanProcessor(ConsoleSpanExporter())
provider.add_span_processor(simple_processor)
```

- Also, spans and attributes can be added by modifying home_activities.py

```py
from datetime import datetime, timedelta, timezone
from opentelemetry import trace

tracer = trace.get_tracer("home.activities")

class HomeActivities:
  def run():
    with tracer.start_as_current_span("home-activites-mock-data"):
      span = trace.get_current_span()
      now = datetime.now(timezone.utc).astimezone()
      span.set_attribute("app.now", now.isoformat())
      
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
      
      span.set_attribute("app.result_length", len(results))
      
      return results
```

- When opening activities endpoint: https://4567-alvarezdani-awsbootcamp-rsrf37dsuxv.ws-us88.gitpod.io/api/activities/home, and executing honeycomb query, now this is the result:

![](assets/week-2/08-honeycomb-trace-multiple-spans.png)

- And the new span contains the defined attributes:

![](assets/week-2/09-honeycomb-trace-attributes.png)

- Honeycomb is useful for implementing different visualizations that can give a visual indication of application behavior, such as average response time grouped by http route:

![](assets/week-2/10-honeycomb-query.png)

- Useful Honeycomb links:

1. Direct Honeycomb.io Login to bypass the main website = https://ui.honeycomb.io/
2. Honeycomb.io Docs on Python OpenTelemetry = https://docs.honeycomb.io/getting-data-in/opentelemetry/python/
3. Honeycomb.io Docs on Getting Data In – Best Practices = https://docs.honeycomb.io/getting-data-in/data-best-practices/
4. Honeycomb.io Glitch.me page – What Honeycomb.io team is this? = https://honeycomb-whoami.glitch.me/

### Instrument AWS X-Ray

AWS X-Ray info: https://aws.amazon.com/xray/

AWS X-Ray provides a complete view of requests as they travel through your application and filters visual data across payloads, functions, traces, services, APIs, and more with no-code and low-code motions.

![](assets/week-2/11-xray.png)

What are the best practises for setting up x-ray daemon? https://stackoverflow.com/questions/54236375/what-are-the-best-practises-for-setting-up-x-ray-daemon

To x-ray to work a daemon must be installed and running for collecting and sending traces in batches to x-ray api.

![](assets/week-2/12-xray-best-practices.png)

The first thing to do for add x-ray instrumentation is adding python module to 'requirements.txt' file:

```
aws-xray-sdk
```

Notes 

> AWS Documentation - SDKs and Toolkits: https://docs.aws.amazon.com/index.html#sdks

> According to Andrew, Ruby docs are the best documented ones

> AWS SDK for Python (Boto3) https://aws.amazon.com/sdk-for-python/

> API Reference: https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/index.html

> XRay: https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/xray.html

> boto3 Github: https://github.com/boto/boto3

> OpenTelemetry Python with AWS X-Ray: https://github.com/aws/aws-xray-sdk-python

Then the new requirement module has to be installed:

```sh
pip install -r requirements.txt
```

Info about how to add xray middleware to flask app: https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-python-middleware.html#xray-sdk-python-adding-middleware-flask

Add imports to `app.py`

```py
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware
```

Add also x-ray initialization to `app.py` 

```py
xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='backend-flask', dynamic_naming=xray_url)
```

> https://docs.aws.amazon.com/xray-sdk-for-python/latest/reference/configurations.html#segment-dynamic-naming

And then x-ray middleware (after app initialization)
```py
XRayMiddleware(app, xray_recorder)
```

AWS resources must be created in order to capture traces in AWS X-RAY (sampling rule and xray group)

Create `xray.json` file with this content:

```json
{
  "SamplingRule": {
      "RuleName": "Cruddur",
      "ResourceARN": "*",
      "Priority": 9000,
      "FixedRate": 0.1,
      "ReservoirSize": 5,
      "ServiceName": "backend-flask",
      "ServiceType": "*",
      "Host": "*",
      "HTTPMethod": "*",
      "URLPath": "*",
      "Version": 1
  }
}
```

Execute this aws cli command to create an xray group:

```sh
FLASK_ADDRESS="https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
aws xray create-group \
   --group-name "Cruddur" \
   --filter-expression "service(\"backend-flask\")"
```

Result

```
>    --group-name "Cruddur" \
>    --filter-expression "service(\"backend-flask\")"
{
    "Group": {
        "GroupName": "Cruddur",
        "GroupARN": "arn:aws:xray:ca-central-1:XXXXXXXXXX:group/Cruddur/CGV4VDVHTFM5BTJMX2C6GOANIC46RLEZJ7FVIINO5ZB2TMWL2ZDA",
        "FilterExpression": "service(\"backend-flask\")",
        "InsightsConfiguration": {
            "InsightsEnabled": false,
            "NotificationsEnabled": false
        }
    }
}
```

And then this one to create the sampling rule:

```sh
aws xray create-sampling-rule --cli-input-json file://journal/aws/json/xray.json
```

Result

```
{
    "SamplingRuleRecord": {
        "SamplingRule": {
            "RuleName": "Cruddur",
            "RuleARN": "arn:aws:xray:ca-central-1:XXXXXXXXXX:sampling-rule/Cruddur",
            "ResourceARN": "*",
            "Priority": 9000,
            "FixedRate": 0.1,
            "ReservoirSize": 5,
            "ServiceName": "backend-flask",
            "ServiceType": "*",
            "Host": "*",
            "HTTPMethod": "*",
            "URLPath": "*",
            "Version": 1,
            "Attributes": {}
        },
        "CreatedAt": "2023-03-01T17:01:08+00:00",
        "ModifiedAt": "2023-03-01T17:01:08+00:00"
    }
}
```

We can check the AWS resources in AWS console (Cloudwatch):

X-ray Group

https://ca-central-1.console.aws.amazon.com/cloudwatch/home?region=ca-central-1#xray:settings/groups

![](assets/week-2/15-xray-group.png)

Sampling rule

https://ca-central-1.console.aws.amazon.com/cloudwatch/home?region=ca-central-1#xray:settings/sampling-rules

![](assets/week-2/16-xray-sampling-rules.png)

> Some info about AWS sampling rules and xray  groups:
> - https://docs.aws.amazon.com/xray/latest/devguide/xray-console-sampling.html
> - https://docs.aws.amazon.com/xray/latest/devguide/xray-api-sampling.html
> - https://aws.amazon.com/blogs/developer/deep-dive-into-aws-x-ray-groups-and-use-cases/

> Andrew mentioned why he implements AWS resources creation using api instead of using AWS console, and it's because AWS console UI changes constantly

> My opinion is that using api, resources creation can be documented and it can be automated using scripts

Next, install X-ray daemon

- Install X-ray Daemon https://docs.aws.amazon.com/xray/latest/devguide/xray-daemon.html
- Github aws-xray-daemon https://github.com/aws/aws-xray-daemon
- X-Ray Docker Compose example https://github.com/marjamis/xray/blob/master/docker-compose.yml

This script is used to install x-ray daemon, but we are not using it because we need it to run in a container

```sh
wget https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-3.x.deb
sudo dpkg -i **.deb
```

So, we are adding a new service to the `docker-compose.yml` file:

```yml
  xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "${AWS_DEFAULT_REGION}"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp
```

The image this container is using is `amazon/aws-xray-daemon` (managed by Amazon)
https://hub.docker.com/r/amazon/aws-xray-daemon/#!

The environment variables used in this container are already configured in Gitpod settings.

The variables that should be added to compose file are the ones used in backend-flask:

```yml
  backend-flask:
    environment:
      AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
      AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```

Now the application can be started, do some requests to backend and check the results:

By using this script, service data for the last 10 minutes can be checked:

```sh
EPOCH=$(date +%s)
aws xray get-service-graph --start-time $(($EPOCH-600)) --end-time $EPOCH
```

Result

```
{
    "Services": [
        {
            "ReferenceId": 0,
            "Name": "4567-alvarezdani-awsbootcamp-fzb9yochj8w.ws-us89.gitpod.io",
            "Names": [
                "4567-alvarezdani-awsbootcamp-fzb9yochj8w.ws-us89.gitpod.io"
            ],
            "Root": true,
            "State": "active",
            "StartTime": "2023-03-01T17:07:10+00:00",
            "EndTime": "2023-03-01T17:07:11+00:00",
            "Edges": [],
            "SummaryStatistics": {
                "OkCount": 0,
                "ErrorStatistics": {
                    "ThrottleCount": 0,
                    "OtherCount": 1,
                    "TotalCount": 1
                },
                "FaultStatistics": {
                    "OtherCount": 0,
                    "TotalCount": 0
                },
                "TotalCount": 1,
                "TotalResponseTime": 0.054
            },
            "DurationHistogram": [
                {
                    "Value": 0.054,
                    "Count": 1
                }
            ],
            "ResponseTimeHistogram": [
                {
                    "Value": 0.054,
                    "Count": 1
                }
            ]
        },
        {
            "ReferenceId": 1,
            "Name": "4567-alvarezdani-awsbootcamp-fzb9yochj8w.ws-us89.gitpod.io",
            "Names": [
                "4567-alvarezdani-awsbootcamp-fzb9yochj8w.ws-us89.gitpod.io"
            ],
            "Type": "client",
            "State": "unknown",
            "StartTime": "2023-03-01T17:07:10+00:00",
            "EndTime": "2023-03-01T17:07:11+00:00",
            "Edges": [
                {
                    "ReferenceId": 0,
                    "StartTime": "2023-03-01T17:07:10+00:00",
                    "EndTime": "2023-03-01T17:07:11+00:00",
                    "SummaryStatistics": {
                        "OkCount": 0,
                        "ErrorStatistics": {
                            "ThrottleCount": 0,
                            "OtherCount": 1,
                            "TotalCount": 1
                        },
                        "FaultStatistics": {
                            "OtherCount": 0,
                            "TotalCount": 0
                        },
                        "TotalCount": 1,
                        "TotalResponseTime": 0.054
                    },
                    "ResponseTimeHistogram": [
                        {
                            "Value": 0.054,
                            "Count": 1
                        }
                    ],
                    "Aliases": []
                }
            ]
        }
    ],
    "StartTime": "2023-03-01T17:07:10+00:00",
    "EndTime": "2023-03-01T17:07:10+00:00",
    "ContainsOldGroupVersions": false
}
```

Also, AWS console can be used to check it

https://ca-central-1.console.aws.amazon.com/cloudwatch/home?region=ca-central-1#xray:traces/query

![](assets/week-2/13-xray-traces.png)

![](assets/week-2/14-xray-trace.png)

Now let's check how to add custom segments/subsegments

> https://github.com/aws/aws-xray-sdk-python#start-a-custom-segmentsubsegment

Add custom subsegment in another endpoint (UserActivities `backend-flask\services\user_activities.py`)

```py
from datetime import datetime, timedelta, timezone

# XRAY import
from aws_xray_sdk.core import xray_recorder

class UserActivities:
  def run(user_handle):

    model = {
      'errors': None,
      'data': None
    }

    now = datetime.now(timezone.utc).astimezone()

    if user_handle == None or len(user_handle) < 1:
      model['errors'] = ['blank_user_handle']
    else:
      now = datetime.now()
      results = [{
        'uuid': '248959df-3079-4947-b847-9e0892d1bab4',
        'handle':  'Andrew Brown',
        'message': 'Cloud is fun!',
        'created_at': (now - timedelta(days=1)).isoformat(),
        'expires_at': (now + timedelta(days=31)).isoformat()
      }]
      model['data'] = results

    # XRAY subsegment
    subsegment = xray_recorder.begin_subsegment('user_activities_subsegment')
    dict = {
      "now": now.isoformat(),
      "results-size": len(model['data'])
    }
    subsegment.put_metadata('info', dict, 'namespace')
    subsegment.put_annotation('id', '12345')
    xray_recorder.end_subsegment()

    return model
```

> Info extracted from: https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-python-subsegments.html

Test again by executing the endpoint to generate traces: /api/activities/@[user]

https://4567-alvarezdani-awsbootcamp-vfl7q5gy1xq.ws-us89.gitpod.io/api/activities/@dd

Result

```json
[
  {
    "created_at": "2023-02-28T18:45:43.422017",
    "expires_at": "2023-04-01T18:45:43.422017",
    "handle": "Andrew Brown",
    "message": "Cloud is fun!",
    "uuid": "248959df-3079-4947-b847-9e0892d1bab4"
  }
]
```

Using AWS console (Cloudwatch), check the generated traces

![](assets/week-2/17-xray-traces-subsegments.png)

We can inspect the trace details, so in here we can see the subsegment, and the generated metadata and annotation

![](assets/week-2/18-xray-trace-subsegments.png)

![](assets/week-2/19-xray-annotations.png)

![](assets/week-2/20-xray-metadata.png)

Finally, for not paying X-RAY use in case it is sending too much data, disable x-ray middleware in `app.py`

```py
# X-RAY middleware
#XRayMiddleware(app, xray_recorder)
```

> AWS X-Ray pricing: https://aws.amazon.com/xray/pricing/

> aws-xray-sdk: Basic Usage: https://docs.aws.amazon.com/xray-sdk-for-python/latest/reference/basic.html

> Generating custom subsegments with the X-Ray SDK for Python: https://docs.aws.amazon.com/xray/latest/devguide/xray-sdk-python-subsegments.html

### Configure custom logger to send to CloudWatch Logs

For adding CloudWatch logs implementation to backend service, first of all we need to add required python module to `requirements.txt` file:

```txt
watchtower
```

> https://pypi.org/project/watchtower/

Then we need to install it:

```sh
pip install -r requirements.txt
```

Required imports must be added to `app.py` file

```py
import watchtower
import logging
from time import strftime
```

Then CloudWatch Logger must be initialized, again in `app.py` file

```py
LOGGER = logging.getLogger(__name__)
LOGGER.setLevel(logging.DEBUG)
console_handler = logging.StreamHandler()
cw_handler = watchtower.CloudWatchLogHandler(log_group='cruddur')
LOGGER.addHandler(console_handler)
LOGGER.addHandler(cw_handler)
```

Add code to handle errors to log them, also to `app.py` file

```py
@app.after_request
def after_request(response):
    timestamp = strftime('[%Y-%b-%d %H:%M]')
    LOGGER.error('%s %s %s %s %s %s', timestamp, request.remote_addr, request.method, request.scheme, request.full_path, response.status)
    return response
```

Then, in the service we need to log something, we add a call to the already configured logger

In this case, we'll add logging to `home_activities.py` file

```py
class HomeActivities:
  def run(logger):
    logger.info('Hello Cloudwatch! from  /api/activities/home')
```

And also pass the logger from `app.py`

```py
@app.route("/api/activities/home", methods=['GET'])
def data_home():
  data = HomeActivities.run(logger=LOGGER)
  return data, 200
```

Finally, we need to add the required env vars to docker compose, in backend-flask service

```yml
version: "3.8"
services:
  backend-flask:
    environment:
      AWS_DEFAULT_REGION: "${AWS_DEFAULT_REGION}"
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
```

For testing it, we start the compose file and then make a request browsing the endpoint /api/activities/home

https://4567-alvarezdani-awsbootcamp-4d0cyhye98n.ws-us89.gitpod.io/api/activities/home

For checking the created logs, get into AWS console, in CloudWatch, Logs, Log groups, and click on cruddur log group, and the generated log stream should be there

![](assets/week-2/21-cloudwatch-log-groups.png)

![](assets/week-2/22-cloudwatch-log-group-cruddur.png)

By clicking on one of them, details can be inspected

![](assets/week-2/23-cloudwatch-logs.png)

For not worrying about spending in AWS, logging can be left disabled

```py
class HomeActivities:
  def run(logger):
    #logger.info('Hello Cloudwatch! from  /api/activities/home')
```

### Integrate Rollbar and capture and error

To integrate Rollbar into the application, first of all, a Rollback account must be opened

https://app.rollbar.com/onboarding

![](assets/week-2/24-rollbar.png)

Search for flask SDK to obtain the instructions to integrate it

![](assets/week-2/25-rollbar-sdk.png)

Info about pyrollbar package:

https://github.com/rollbar/pyrollbar

First of all, we need to add the python modules to `requirements.txt`

```txt
blinker
rollbar
```

And install them

```sh
pip install -r requirements.txt
```

Then we need to generate a Rollback access token and add it as an environment variable to docker compose file

![](assets/week-2/26-rollbar-access-token.png)

```sh
export ROLLBAR_ACCESS_TOKEN="XXXXXX"
gp env ROLLBAR_ACCESS_TOKEN="XXXXXX"
```

![](assets/week-2/27-rollbar-env.png)

```yml
version: "3.8"
services:
  backend-flask:
    environment:
      ROLLBAR_ACCESS_TOKEN: "${ROLLBAR_ACCESS_TOKEN}"
```

For instrumenting the backend application, we need to add first the python imports to `app.py` file

```py
import os
import rollbar
import rollbar.contrib.flask
from flask import got_request_exception
```

Then we need to add Rollbar initialization, also in `app.py` file (after app declaration)

```py
rollbar_access_token = os.getenv('ROLLBAR_ACCESS_TOKEN')
@app.before_first_request
def init_rollbar():
    """init rollbar module"""
    rollbar.init(
        # access token
        rollbar_access_token,
        # environment name
        'production',
        # server root directory, makes tracebacks prettier
        root=os.path.dirname(os.path.realpath(__file__)),
        # flask already sets up logging
        allow_logging_basic_config=False)

    # send exceptions from `app` to rollbar, using flask's signal system.
    got_request_exception.connect(rollbar.contrib.flask.report_exception, app)
```

For testing Rollbar implementation, we are adding a new endpoint

```py
@app.route('/rollbar/test')
def rollbar_test():
    rollbar.report_message('Hello World!', 'warning')
    return "Hello World!"
``` 

So, by starting the application with docker compose up, we can test if it's working

![](assets/week-2/28-rollbar-compose.png)

We need to use the new endpoint defined to test it, so let's open it: /rollbar/test

https://4567-alvarezdani-awsbootcamp-yy4tw88k3ks.ws-us89.gitpod.io/rollbar/test

![](assets/week-2/29-rollbar-test.png)

And check the data in Rollbar (remember to check on all log levels to see the warning message)

https://app.rollbar.com/a/alvarez.daniel

![](assets/week-2/30-rollbar-items.png)

![](assets/week-2/31-rollbar-item1.png)

For an error to occur, we will change some code that should raise an error message in Rollbar

In `home_activities.py` introduce an error

```py
#return results
results
```

Open the request api: https://4567-alvarezdani-awsbootcamp-4n2xg6y3x4r.ws-us89.gitpod.io/api/activities/home

![](assets/week-2/32-rollbar-error.png)

The error is also shown in Rollbar

![](assets/week-2/33-rollbar-error.png)

![](assets/week-2/34-rollbar-item2.png)

> Rollbar flask example: https://github.com/rollbar/rollbar-flask-example/blob/master/hello.py

## Knowledge Challenges

### Security Quiz

What is one effect of implementing observability tools inside your web application?
- Decreased alert fatigue for security teams (X)
- Faster code deployments into production
- Increased operation costs
- Significantly more complex code base

Which of the following is NOT one of the 3 main pillars of observability?
- Metrics
- Traces
- Logs
- Analytics (X)

Which AWS service is responsible for logging backend API calls and other activity between the services inside your account?
- Cloud Watch (X)
- Cloud Trail
- Sessions Manager
- Cloud Advisor

What is the purpose of the Amazon Detective AWS Service?
- Automatically patch vulnerabilities in your AWS resources
- Collect and analyze AWS resource logs for security investigations (X)
- Provisioning AWS Security focused services in your account
- Prevent overspend on AWS security focused services

Which of the following is an example of a common instrumentation agent?
- AWS Cloud Trail Agent
- AWS Systems Manager
- AWS SQS
- AWS X-Ray Daemon (X)

### Pricing Quiz

How many monthly events are included in the Honeycomb free tier?
- 10M
- 20M (X)
- 5M
- 25M

How many monthly error events are included on the Rollbar free tier?
- 5000 (X)
- 500
- 10000
- 20000

How many AWS X-Ray traces are allowed per month on the free tier?
- 1000
- 10000
- 100000 (X)
- 25000

If you created 12 custom cloudwatch metrics on your AWS account, how many would you actually be billed for?
- 2
- 10
- 12 (X)
- 7

AWS Service billing for X-Ray tracing can vary region to region
- TRUE (X)
- FALSE

## Extra

### Configure Github Codespaces

Following Andrew's video about Github Codespaces, I've configured a workspace for using Codespaces.

First of all, I've added a devcontainer.json file with the required configuration for vscode extensions and theme.

![](assets/week-2/35-codespaces-devcontainer.png)

Also, I've configured required environment variables as Codespaces secrets

![](assets/week-2/36-codespaces-secrets.png)

I've ensured that environment variables are available by checking them with `env | grep AWS` command

![](assets/week-2/37-codespaces-env-vars.png)

Before starting compose file, I've also changed docker compose env vars, from GitPod to Codespaces ones:

![](assets/week-2/38-codespaces-compose-env.png)

Then, I've tested cruddur by starting compose file (before that I've run npn i in frontend folder)

When container are started, I've made backend port as public, for not getting CORS errors when using the application.

After all of this, I've opened cruddur main page and checked that everything is working as expected

https://alvarezdaniel-friendly-broccoli-6pxppjjv5hqj-3000.preview.app.github.dev/

![](assets/week-2/39-codespaces-cruddur.png)

Finally, for not consuming Codespaces credits, I've stopped the codespaces instance

![](assets/week-2/40-codespaces-stop.png)









