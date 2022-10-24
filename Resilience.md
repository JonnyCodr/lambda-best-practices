# Resilience

## Fan out

|Service|Description|Price|Concurency|
|:---|:---|:---|:---|
|SNS| 1000 x 60s x 60m x 24hr x 30 days @ $0.50 per mill | $1296.00 |grows Linearly|
|SQS| 1000 x 60s x 60m x 24hr x 30 days @ $0.40 per mill | $1036.80 | grows O(n)|
|EventBridge| 1000 x 60s x 60m x 24hr x 30 days @ $1.00 per mill | $2592.00 |grows Linearly|
|Kinesis| 1000 x 60s x 60m x 24hr x 30 days @ $0.015 per mill | $47.09 |grows as in steps per shard|


## Controlled Concurency

if you want:
- maximum throughput
  - SNS/EventBridge
- percice control over concurency
  - Kinesis

### Api Gateway service proxies

#### when to use it
- User when yiou are concerned about cold start overhead or burst limits AND function does nothing but call AWS and return a response

what you loose:
- retry and exponential backoff
- contextual logging
- error handling
- fallbacks
- tracing
- chaos tools

### Load Testing

- test user flowsm not individual functions
- don't forget to load test the async parts of the system

### Provisioned Concurrency

### Handling RDS connections

Note: the default RDS configs are bad
- the max connection is too low
- idle connections are not closed.
- too many connections per "container"
- serverside idle timeout is too long

#### fixing these issues

- set "wait_timeout" and "interactive_timeout" to 10 mins (default is 8 hours)
- increase "max_connections" to 1000 (default is 151)
- set the client socket pool size to 1 (default is 50)
- use RDS Proxy when it becomes available

## Security

- use IAM roles for services
- use the principle of least privilege
- Avoidusing '*' in IAM policies

|Secrest Manager|Attribute|Parameter Store|
|:---:|---|:---:|
|✔|use KMS to encrypt secrets|✔|
|✔|encryprtion at rest and in flight|✔|
|✔|cost efficient|X|
|✔|scalable|✔|
|✔|fully manages|✔|
|X|built in rotation|✔|

### SSM Parameter Store
        cost: $0.05 per 10,000 requests


### Secrets Manager
        cost: $0.40 per secret per month

- use to pull in environment variables rather then setting them in a template
        https://therecord.media/another-set-of-malicious-npm-packages-caught-stealing-discord-tokens-environment-variables/

### Api Gateway

Note that the 10000 request limit is per stage, not per API it is account wide. So if you have 10 stages, you can make 100,000 requests per month.


## Resiliancy
BE MULIT-REGION

## Observability

### Alerts
Lambda
- create an alarm for concurrent executions, set to ~80% of provisioned concurrency
- Kenisis alarm for iterator age
- DeadLetter Queue
- Throttles
- error % in CloudWatch (can be coinfigured in the console)

API Gateway
- p90/p95/p99 latency never use average
- success reate % (can be configured in the console)
- 4xx rate %
- 5xx rate %

SQS
- message age

Step functions
- failed count`
- throttle count
- timedout 

### Logging
- use Labdas built in logging collection
  - logging libs tend to be too heavy for lambdas
  - Lambda ships all logs to cloudwatch
  - don't bother with implementing your own logging that dumps them to a 3rd party product
- use structured logging with JSON
- use the correct log level
- use log sampling when applicable
  
### Distributed Tracing

#### Using X-Ray