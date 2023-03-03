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


Observability is based on three pillars : Logs, Metrcis & Tracing

### Observability in AWS 

image.png


image.png

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


