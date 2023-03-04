# Week 2 — Distributed Tracing
It was a bit difficult to do this week's tasks because I had no prior knowledge of distributed tracing and I was not in the best state due to the elections held in my country - Nigeria. I managed to complete all the videos in the youtube playlist and watch additional videos to better understand distributed tracing.

## Required tasks
### Instrumenting HoneyComb with OTEL
I already created a honeycomb account in week0 following Shala's youtube video.
Honeycomb.io is a cloud-based observability platform designed for engineers and developers to understand and analyze complex systems and applications. It allows users to collect, explore, and visualize data from various sources, including logs, traces, and metrics, in real-time.

I carried out the following to instrument HoneyComb:
- I created a new environment in HoneyComb called `aws-bootcamp-cruddur`.  
An environment refers to a set of configurations and settings that define how your application or system behaves in different contexts. Environments are used to manage the various stages of your application's lifecycle, such as development, staging, and production, and to separate data and resources across these stages.
Each environment in Honeycomb can have its own datasets, API keys, integrations, and other settings that are specific to that environment.
- I copied the API key from my newly created environment and saved it in my gitpod environment as an environment variable
```bash
export HONEYCOMB_API_KEY="myapikey"
gp env HONEYCOMB_API_KEY="myapikey"
```
- In the `requirements.txt` file I added the following to install Open Telemetry packages to instrument a flask app
```bash
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```
And then I changed directory to `backend-flask` and installed them with:
```bash
pip install -r requirements.txt
```
- I added the following in my `app.py` file to create and initialize a tracer and Flask instrumentation to send data to Honeycomb
```bash    
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
- I added the following to configure OpenTelemetry to send events to Honeycomb using environment variables in the `docker-compose.yml` file
```bash
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
OTEL_SERVICE_NAME: "backend-flask"
```
${HONEYCOMB_API_KEY} references the API key environment variable that was exported earlier.
- I added spans and attributes. See it [here]()  

A span is a unit of work in a distributed system that represents a single operation or transaction. Spans capture information such as the start time, duration, and outcome of an operation, as well as any events or activities that occurred during the operation. By collecting spans, you can gain visibility into the flow of requests through your system and identify bottlenecks or issues that may be affecting performance.  
Attributes are key-value pairs that provide additional context for spans and events. Attributes can be used to capture metadata about the application, such as the version number, hostname, or user ID, as well as specific details about a span, such as the name of the function or the input parameters. By adding attributes to your data, you can better understand the context in which events occurred and make more informed decisions based on the data.
