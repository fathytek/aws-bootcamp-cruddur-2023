# Week 2 — Distributed Tracing

## Observability

- Decreased Alert Fatigue for Security Operation Teams
- Visibility of end2end of Logs, metrics & tracing
- Troubleshoot and resolve things quickly without costing too much money
- Understand application health
- Accelerate collaboration between teams
- Reduce overall operational cost
- Increase customer satisfaction

### Difference between Monitoring and Observability

There are many differentiating factors between monitoring and Observability
Let's take a look to some of them
for the availability, monitoring show you if the system is available where observability give you how long is up
For the performance, monitoring show you if the system is healthy or not, where observability illustrates what the system is doing
Fault Management and Recovery are importants keys, monitoring helps you to know  when and where the issue did occur and if the system is backup, meanwhile observality give the reason why it occur and what you can do to prevent the issue from reoccuring

### Observability in AWS 

Observability is based on three pillars : Logs, Metrcis & Tracing

![image](https://user-images.githubusercontent.com/32872009/222726404-7ba809ec-8d32-4480-8a78-a74736d618dd.png)

All the solutions provided by AWS for the observalibility: you can choose the needed tool to accomplish your objective

![image](https://user-images.githubusercontent.com/32872009/222726976-8edc6de0-484b-429b-96d4-751e05baa32b.png)


## HoneyComb

1. Install Packages
Install these packages to instrument a Flask app with OpenTelemetry:

```
opentelemetry-api
opentelemetry-sdk
opentelemetry-exporter-otlp-proto-http
opentelemetry-instrumentation-flask
opentelemetry-instrumentation-requests

```
2. Initialize
Add these lines to your existing Flask app initialization file app.py (or similar). These updates will create and initialize a tracer and Flask instrumentation to send data to Honeycomb

```
# app.py updates
    
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor

# Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)

# Initialize automatic instrumentation with Flask
app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```

3. Configuration and Run

From your honeycomb account copy your API KEY and export in your environment variable :

```sh
export HONEYCOMB_API_KEY=""
export HONEYCOMB_SERVICE_NAME="Cruddur"
gp env HONEYCOMB_API_KEY=""
gp env HONEYCOMB_SERVICE_NAME="Cruddur"
```

in your docker-composer.yaml  we need to add the following input 

```yml
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
OTEL_SERVICE_NAME: "${HONEYCOMB_SERVICE_NAME}"
```

## AWS X-RAY

### Instrument AWS X-Ray for Flask

I designate the region closed to me 

```sh
export AWS_REGION="eu-west-3"
gp env AWS_REGION="eu-west-3"
```

### Installation of aws-xray-sdk

for this step we need to install the aws-xray-sdk , so we should to add it in our requirements.txt of backend-flask repo as follow

```py
aws-xray-sdk
```

Install the needed dependencies under backend-flask

```sh
pip install -r requirements.txt
```

Instrument app.py with the needed instructions in backend-flask repo

Add to `app.py`

```py
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='backend-flask', dynamic_naming=xray_url)
XRayMiddleware(app, xray_recorder)
```


#### Add AWS XRay Resouce file

Add `aws/json/xray.json`

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
### AWS CLI commands for AWS X-RAY

```sh
export FLASK_ADDRESS="https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
aws xray create-group \
   --group-name "Cruddur" \
   --filter-expression "service(\"$FLASK_ADDRESS\") {fault OR error}"
```

```sh
aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
```
### More documentation for AWS X-RAY

[Install X-ray Daemon](https://docs.aws.amazon.com/xray/latest/devguide/xray-daemon.html)

[Github aws-xray-daemon](https://github.com/aws/aws-xray-daemon)

```sh
 wget https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-3.x.deb
 sudo dpkg -i **.deb
 ```

### Update docker-compose.yml file

We need to add the following variables

```yml
      AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
      AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```

and insert the xray deamon too

```yml
  xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "eu-west-3"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp
```
[X-Ray Docker Compose example](https://github.com/marjamis/xray/blob/master/docker-compose.yml)

## Rollbar

https://rollbar.com/

Create a new project in Rollbar called `Cruddur`

Add to `requirements.txt`


```
blinker
rollbar
```

Install dependencies

```sh
pip install -r requirements.txt
```

from your the onboard page retrieve the value of your token and set it below


```sh
export ROLLBAR_ACCESS_TOKEN=""
gp env ROLLBAR_ACCESS_TOKEN=""
```

Add to backend-flask for `docker-compose.yml`

```yml
ROLLBAR_ACCESS_TOKEN: "${ROLLBAR_ACCESS_TOKEN}"
```

```py

## Rollbar init code. You'll need the following to use Rollbar with Flask.
## This requires the 'blinker' package to be installed

import os
import rollbar
import rollbar.contrib.flask
from flask import got_request_exception

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

## Simple flask app

@app.route('/')
def hello():
    print "in hello"
    x = None
    x[5]
    return "Hello World!"

```


