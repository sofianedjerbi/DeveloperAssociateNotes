# API Gateway

## API Gateway Precisions

There is support for **websocket, versioning, different environments, security, API keys**...

## Deployment Stages

Changes need to be **deployed**.  
Changes are deployed to **stages**, each stage has its own configuration.  
Stages **can be rolled back**, a history of deployment is kept.

`https://api.example.com/<stage>` stage can be `v1` or `v2` as example.

### Stage Variables

**Environment variables** for API Gateway.  
They are passed to the consumer (lambda...).  

Can be used as a **pointer** to the right **lambda alias**.

## Canary Deployment

Choose the **% of the traffic** to redirect to the Canary channel.  

## Integration Types

- **MOCK:** Return a response without asking the backend.
- **HTTP / AWS:** We use **mapping templates** to change the request & the response.
- **AWS_PROXY:** Incoming request are passed **to AWS services** without any changes.
- **HTTP_PROXY:** Proxy the HTTP request.

### Mapping Templates

Templates use the **VTL** language, to filter output results.  
Content type can be either `application/json` or `application/xml`.

Can be used to convert JSON to XML, or query strings to JSON...

## Open API spec

Open API is used to define REST APIs as code.  
It is **compatible with API Gateway**.  

Can be used to **validate request format**.  
Create an Open API definition file and add a **AWS Extension parameter** to enable validation.

## Caching

Default **TTL** is 300s (from 0s to 3600s).  
Caches are defined **per stage**.  
Possible to override cache settings **per method**.  
Compatible with **encryption**, from 0.5GB to 237GB.  
Very expensive.

### Invalidation

- **Flush the entire cache**
- **Clients can invalidate the cache** with `header: Cache-Control: max-age=0` and IAM permissions.  

If no invalidation policy is given, then **any client can invalidate the cache**.

## Usage Plans & Keys

Used to create **paid APIs**, with **rate limits**, **API Keys** and **throttling**.  
Order for API Keys:
1. **Create and deploy** the API, configure the methods to require an API key.
2. Generate or import **API keys** for customers
3. Create the usage plan with **limits**
4. **Associate API stages and API keys with the usage plan**.

## Logging & Tracing

**CloudWatch Logs** contains information about **requests/response body**.  
Enable logging at **stage level**.  
Can override settings on a per API basis.

**XRay** can provide extra information about requests in API Gateway.

**CloudWatch Metrics** can provide **CacheHitCount** and **CacheMissCount**.  
Request **Count**, **IntegrationLatency** for backend latency and **Latency** for total latency.
There are also metrics for errors **4XXError** and **5XXError**

## Throttling

Throttle requests at **10000 rps** across **all API**. (soft limit that can be increased)  
We can set **stage or method level limits** or **per customer**.  

Once an API is overloaded, **all other APIs are going to be throttled**.

## Errors

### 4xx - Client errors
- **400:** Bad Request
- **403:** Access Denied, WAF filtered
- **429:** Quota exceeded, Throttle

### 5xx - Server errors
- **502:** Bad Gateway Exception, typically due to incompatible backend outputs or heavy loads.
- **503:** Service Unavailable Exception
- **504:** Integration Failure – Endpoint Request Timed-out, API Gateway timeout after 29 seconds.


## CORS

Must be enabled to **receive calls from another domain**.  
Pre-flight request must contain the following headers:
- **Access-Control-Allow-Methods**
- **Access-Control-Allow-Headers**
- **Access-Control-Allow-Origin**

## Security

First, **IAM permissions** are available for **access within AWS**, IAM credentials are in header.  
**Resource Policies** allow to setup **IP / cross acount access** at **resource-level**.  
**Cognito User Pools** can be used to get an access **token**.

### Lambda Authorizer

Used to verify **3rd party auth systems** (OAuth / JWT).  
Lambda authorizers are **custom AWS Lambda functions** that handle API Gateway request authentication.  
They verify tokens or other credentials against user-defined logic.  
Upon validation, they return IAM policies to manage access.

## HTTP API

All-proxy, supports **OIDX and OAuth 2.0**.  
Don't support **resource policies**.

## REST API

Don't suport **OIDC / OAuth 2.0**.

## WebSocket API

Two-way interactive communication. **Stateful**.  
Used in **real-time** applications: chat applications, multiplayer games...
The **server can send message to the client**.

URL: `wss://<name>.execute-api.us-west-1.amazonaws.com/<stage>`  
Callback extension is `/@connections/<commection id>` *(POST / GET / DELETE)*

### Routing

Incoming JSON messages are routed to different backend with a **route key table**.  
There is a **default** backend if no specific backend is found.

## Architecture

You can use API Gateway to setup **a single interace for all microservices**.  
We can create **API Endpoints for various resources**.  
We can **apply SSL certificates**.
And **gateway-level transformation rules**.
