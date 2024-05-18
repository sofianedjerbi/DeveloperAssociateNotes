# Metrics

## CloudWatch Custom Metrics

Use API call **PutMetricData**.  
You can segment metrics by instance id, environment name **= dimensions**...

Metrics have 2 resolutions with **StorageResolution** API:
- **Standard:** 1 minute
- **High Resolution:** 1/5/10/30 secs, higher cost.

You can post metrics up to **2 weeks in the past or 2 hours in the future**.

## CloudWatch Logs Metric Filter

Count **occurences in logs**. *(ex: find a specific IP)*  
Can **trigger alarms**. they are **not retroactive**. *(they only record data from their creation)*

## CloudWatch Synthetics Canary

Configurable **script that monitor APIs, URLs, Websites**...  
Reproduce what customers do programmatically to find issues before customers are impacted.  
Checks availability, latency, screenshots...

Integration with CloudWatch Alarms. Supports python / nodejs with headless google chrome.
Runs on a schedule.

Blueprints:

- **Heartbeat Monitor** – load URL, store screenshot, and an HTTP archive file.
- **API Canary** – test basic read and write functions of REST APIs.
- **Broken Link Checker** – check all links inside the URL.
- **Visual Monitoring** – compare a screenshot from a canary run with a baseline screenshot.
- **Canary Recorder** – used with CloudWatch Synthetics Recorder. Records your actions on a website and generates a script.
- **GUI Workflow Builder** – verifies actions can be taken on your webpage (e.g., test a login form).

## EventBridge Multi-Account Integration

Send events across acount **with event rules across** accounts that send events to **an event bus in a central account**.

## AWS X-Ray Precisions

Gives a **visual analysis** of applications.  
You can visualize **dependencies**, **service issues, bottlenecks**.

We can **trace a request and show its path**.

Should be **enabled in code**, it requires **a daemon**.

*Path: Code -> Daemon -> AWS X-Ray*

## X-Ray Vocabulary

- **Segments:** A measured part of the code. Each application/service will send them.
- **Subsegments:** For more details in your segment.
- **Trace:** Segments collected together for an end-to-end trace.
- **Sampling:** Decrease the number of requests sent to X-Ray, reduce cost.
- **Annotations:** Key-value pairs used to index traces and used with filters.
- **Metadata:** Key-value pairs, not indexed, not used for searching.

**The X-Ray daemon/agent has a config to send traces cross-account:**
- Ensure IAM permissions are correct - the agent will assume the role.
- Allows having a central account for all application tracing.

```js
const AWS = require('aws-sdk');
const AWSXRay = require('aws-xray-sdk');
const https = require('https');

AWSXRay.captureAWS(AWS);
const dynamoDB = new AWS.DynamoDB.DocumentClient();

// Main segment
AWSXRay.captureAsyncFunc('mainSegment', (mainSegment) => {
  // HTTP request subsegment
  const httpSubsegment = mainSegment.addNewSubsegment('httpsRequest');
  https.get('https://api.example.com/data', (res) => {
    res.on('end', () => httpSubsegment.close());
  }).on('error', (err) => {
    httpSubsegment.addError(err);
    httpSubsegment.close();
  });

  // DynamoDB operation subsegment
  const dbSubsegment = mainSegment.addNewSubsegment('dynamoDBQuery');
  dynamoDB.get({ TableName: 'YourTable', Key: { id: 'example' } }, (err, data) => {
    if (err) dbSubsegment.addError(err);
    dbSubsegment.close();
  });

  // Add annotations and metadata
  mainSegment.addAnnotation('key', 'value');
  mainSegment.addMetadata('key', { custom: 'data' });

  mainSegment.close();
});
```

## X-Ray Instrumentation in code

Used to measure performance **of some part of the code**.  

```js
const express = require('express');
const app = express();

const AWSXRay = require('aws-xray-sdk');
app.use(AWSXRay.express.openSegment('MyApp'));

app.get('/', (req, res) => {
    res.render('index');
});

app.use(AWSXRay.express.closeSegment());
```

## X-Ray Sampling Rules

The X-Ray SDK **samples requests** to monitor service performance.  

By default, **it always records the first request each second.**  
Beyond that, **it samples 5% of additional requests.**  
The **guaranteed one request per second** is called the *reservoir*.  

The 5% rate is how often it samples additional traffic beyond this *reservoir*.

You can create your own rules with the **reservoir** and **rate**.

## X-Ray APIs

### Write APIs used by the daemon

- **PutTraceSegments**: Uploads trace segments with timing and operational data.
- **PutTelemetryRecords**: Sends aggregated health statistics of applications.
- **GetSamplingRules**: Retrieves all sampling rules. *(to know what to send)*
- **GetSamplingTargets**: Requests targets for adjusting sampling rates.
- **GetSamplingStatisticSummaries**: Fetches summaries of sampling statistics.

### Read APIs

- **GetSamplingRules**: Retrieves rules that control data collection volume.
- **GetSamplingTargets**: Requests targets for adjusting sampling rates.
- **GetSamplingStatisticSummaries**: Fetches summaries of sampling statistics.
- **BatchGetTraces**: Retrieves a batch of trace data by trace IDs.
- **GetServiceGraph**: Generates a graphical representation of service components.
- **GetTraceGraph**: Provides a detailed graph of a single trace.
- **GetTraceSummaries**: Retrieves summaries of recent traces.
- **GetGroups**: Lists all defined groups in AWS X-Ray.
- **GetGroup**: Retrieves details about a specific group in AWS X-Ray.
- **GetTimeSeriesServiceStatistics**: Gets time-series data for service statistics.

## X-Ray with BeanStalk

You can run the daemon by setting **an option in the console or with a config file**.  
It requires the **correct IAM config**.

## X-Ray Integrations on ECS

- A single **X-Ray Daemon Container** in each EC2 instance. (EC2 Only)
- X-Ray container **as a side car**. (EC2 / Fargate)

You have to:
1. Map the container **port 2000 for udp**.
2. Set the env variable **AWS_XRAY_DAEMON_ADDRESS** to `xray-daemon:2000`.
3. Link the two containers *(network)*.

## AWS Distro for OpenTelemetry

AWS-supported version of **OpenTelemetry** (X-Ray but open source).  
Often used to send traces to multiple destinations.

## CloudTrial Insights

**Automatically detects and notifies** you of **unusual API activity**.**  
Helping to identify potential **security issues or operational problems**.

*Powered by AI.*

## Cloudwatch Detailed Monitoring

Enabled for collecting metric data **every 1-minute instead of 5**.

## CloudWatch Custom Metrics High-Resolution

Enabled to record data with **a minimum of 1-second granularity instead of 1-minute**.  
Alarms can be triggered as often as **10 seconds**.
