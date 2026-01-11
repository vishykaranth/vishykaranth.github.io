# AWS Lambda - In-Depth Guide

## Part 1: Fundamentals, Architecture, and Core Concepts

---

## Table of Contents

1. [Introduction to AWS Lambda](#1-introduction-to-aws-lambda)
2. [How Lambda Works](#2-how-lambda-works)
3. [Lambda Architecture](#3-lambda-architecture)
4. [Event Sources and Triggers](#4-event-sources-and-triggers)
5. [Lambda Function Structure](#5-lambda-function-structure)
6. [Supported Runtimes](#6-supported-runtimes)
7. [Configuration and Limits](#7-configuration-and-limits)
8. [Execution Model](#8-execution-model)
9. [Use Cases](#9-use-cases)
10. [Creating Your First Lambda Function](#10-creating-your-first-lambda-function)

---

## 1. Introduction to AWS Lambda

### 1.1 What is AWS Lambda?

AWS Lambda is a **serverless compute service** that lets you run code without provisioning or managing servers. You simply upload your code, and Lambda handles everything required to run and scale your code with high availability.

### 1.2 Key Characteristics

- **Serverless**: No server management required
- **Event-Driven**: Executes code in response to events
- **Auto-Scaling**: Automatically scales from zero to thousands of concurrent executions
- **Pay-Per-Use**: Pay only for compute time consumed
- **Managed Service**: AWS handles infrastructure, patching, and monitoring
- **Multi-Language Support**: Supports multiple programming languages

### 1.3 Serverless vs Traditional Computing

#### Traditional Computing (EC2)
```
┌─────────────────────────────────────┐
│ You Manage:                         │
│ - Server provisioning               │
│ - OS installation & updates         │
│ - Security patches                  │
│ - Scaling configuration             │
│ - Monitoring setup                  │
│ - Load balancing                    │
│ - High availability                 │
└─────────────────────────────────────┘
```

#### Serverless Computing (Lambda)
```
┌─────────────────────────────────────┐
│ AWS Manages:                        │
│ - Server provisioning               │
│ - OS installation & updates         │
│ - Security patches                  │
│ - Auto-scaling                      │
│ - Monitoring                        │
│ - Load balancing                    │
│ - High availability                 │
│                                     │
│ You Focus On:                        │
│ - Writing code                       │
│ - Business logic                     │
└─────────────────────────────────────┘
```

### 1.4 Benefits of Lambda

1. **No Server Management**: Focus on code, not infrastructure
2. **Automatic Scaling**: Handles traffic spikes automatically
3. **Cost-Effective**: Pay only for compute time (100ms increments)
4. **High Availability**: Built-in fault tolerance and redundancy
5. **Fast Deployment**: Deploy code in seconds
6. **Event Integration**: Native integration with 200+ AWS services

---

## 2. How Lambda Works

### 2.1 Execution Flow

```
┌─────────────────────────────────────────────────────────┐
│ 1. Event Occurs                                         │
│    (e.g., file uploaded to S3, API request, etc.)      │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│ 2. Event Source Triggers Lambda                         │
│    (S3, API Gateway, EventBridge, etc.)                  │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│ 3. Lambda Service Receives Event                        │
│    - Validates event                                    │
│    - Checks function configuration                      │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│ 4. Lambda Execution Environment                        │
│    - Creates or reuses execution environment            │
│    - Loads function code                                │
│    - Initializes runtime                                │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│ 5. Execute Function                                     │
│    - Runs handler function                              │
│    - Processes event                                    │
│    - Returns response                                   │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│ 6. Response/Result                                      │
│    - Returns to event source                            │
│    - Logs execution details                             │
│    - Updates metrics                                    │
└─────────────────────────────────────────────────────────┘
```

### 2.2 Cold Start vs Warm Start

#### Cold Start
```
First Invocation or After Idle Period:
┌─────────────────────────────────────┐
│ 1. Create execution environment     │ ~100-1000ms
│ 2. Download function code            │
│ 3. Initialize runtime                │
│ 4. Execute initialization code       │
│ 5. Run handler function              │
└─────────────────────────────────────┘
Total: ~1-3 seconds (first time)
```

#### Warm Start
```
Subsequent Invocations (within ~15 minutes):
┌─────────────────────────────────────┐
│ 1. Reuse execution environment      │ ~1-10ms
│ 2. Run handler function              │
└─────────────────────────────────────┘
Total: ~10-100ms
```

### 2.3 Execution Environment Lifecycle

```
┌─────────────────────────────────────────────────────────┐
│ Execution Environment Created                            │
│ - Allocated compute resources                           │
│ - Runtime initialized                                   │
│ - Function code loaded                                  │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│ INIT Phase (Cold Start Only)                            │
│ - Runs code outside handler (global scope)              │
│ - Initialize connections, load libraries                │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│ INVOKE Phase                                             │
│ - Handler function executes                             │
│ - Processes event                                        │
│ - Returns response                                       │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│ Environment Reused (if within timeout)                   │
│ - Next invocation reuses same environment               │
│ - Skips INIT phase                                       │
└─────────────────────────────────────────────────────────┘
```

---

## 3. Lambda Architecture

### 3.1 High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    AWS Lambda Service                    │
├─────────────────────────────────────────────────────────┤
│                                                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   Region 1   │  │   Region 2   │  │   Region 3   │  │
│  │              │  │              │  │              │  │
│  │ ┌──────────┐ │  │ ┌──────────┐ │  │ ┌──────────┐ │  │
│  │ │Function  │ │  │ │Function  │ │  │ │Function  │ │  │
│  │ │Version 1 │ │  │ │Version 1 │ │  │ │Version 1 │ │  │
│  │ └──────────┘ │  │ └──────────┘ │  │ └──────────┘ │  │
│  │              │  │              │  │              │  │
│  │ ┌──────────┐ │  │ ┌──────────┐ │  │ ┌──────────┐ │  │
│  │ │Function  │ │  │ │Function  │ │  │ │Function  │ │  │
│  │ │Version 2 │ │  │ │Version 2 │ │  │ │Version 2 │ │  │
│  │ └──────────┘ │  │ └──────────┘ │  │ └──────────┘ │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │         Execution Environments Pool              │  │
│  │  (Auto-scaled, managed by AWS)                  │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
└─────────────────────────────────────────────────────────┘
         ▲                    ▲                    ▲
         │                    │                    │
    ┌────┴────┐          ┌────┴────┐          ┌────┴────┐
    │  S3     │          │  API    │          │ Event   │
    │ Events  │          │ Gateway │          │ Bridge  │
    └─────────┘          └─────────┘          └─────────┘
```

### 3.2 Lambda Function Components

```
Lambda Function
├── Function Code
│   ├── Handler function (entry point)
│   ├── Business logic
│   └── Dependencies/libraries
│
├── Configuration
│   ├── Runtime (Python, Node.js, Java, etc.)
│   ├── Memory (128MB - 10GB)
│   ├── Timeout (1s - 15 minutes)
│   ├── Environment variables
│   └── IAM role (permissions)
│
├── Event Source Mappings
│   ├── S3 bucket triggers
│   ├── DynamoDB streams
│   ├── Kinesis streams
│   └── SQS queues
│
└── Layers
    ├── Shared libraries
    ├── Custom runtimes
    └── Common code
```

### 3.3 Execution Environment Details

```
┌─────────────────────────────────────────────────────────┐
│ Execution Environment (Container)                       │
├─────────────────────────────────────────────────────────┤
│                                                          │
│ ┌────────────────────────────────────────────────────┐ │
│ │ Runtime Interface (Language Runtime)                │ │
│ │ - Python 3.9, Node.js 18, Java 17, etc.           │ │
│ └────────────────────────────────────────────────────┘ │
│                                                          │
│ ┌────────────────────────────────────────────────────┐ │
│ │ Function Code                                        │ │
│ │ - Your handler function                             │ │
│ │ - Dependencies                                       │ │
│ └────────────────────────────────────────────────────┘ │
│                                                          │
│ ┌────────────────────────────────────────────────────┐ │
│ │ Lambda Layers (Optional)                            │ │
│ │ - Shared libraries                                  │ │
│ │ - Common utilities                                  │ │
│ └────────────────────────────────────────────────────┘ │
│                                                          │
│ ┌────────────────────────────────────────────────────┐ │
│ │ Runtime API                                         │ │
│ │ - /runtime/invocation/next (get event)             │ │
│ │ - /runtime/invocation/response (send response)     │ │
│ │ - /runtime/invocation/error (report error)        │ │
│ └────────────────────────────────────────────────────┘ │
│                                                          │
│ ┌────────────────────────────────────────────────────┐ │
│ │ System Resources                                    │ │
│ │ - CPU (proportional to memory)                     │ │
│ │ - Memory (128MB - 10GB)                            │ │
│ │ - Ephemeral storage (/tmp: 512MB - 10GB)           │ │
│ │ - Network (VPC or internet)                        │ │
│ └────────────────────────────────────────────────────┘ │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

---

## 4. Event Sources and Triggers

### 4.1 Event Source Categories

#### Push Model (Event-Driven)
Services that push events to Lambda:
- **S3**: Object created, deleted, or modified
- **API Gateway**: HTTP/HTTPS requests
- **CloudFront**: Viewer request/response events
- **Cognito**: User pool triggers
- **EventBridge**: Scheduled events or custom events
- **SNS**: Topic notifications
- **SES**: Email events
- **CodeCommit**: Repository events
- **CloudFormation**: Stack events

#### Pull Model (Polling)
Lambda polls services for events:
- **SQS**: Queue messages
- **Kinesis**: Stream records
- **DynamoDB Streams**: Table change events
- **MSK (Kafka)**: Topic messages

### 4.2 Common Event Sources

#### S3 Events
```json
{
  "Records": [
    {
      "eventVersion": "2.1",
      "eventSource": "aws:s3",
      "eventName": "ObjectCreated:Put",
      "s3": {
        "bucket": {
          "name": "my-bucket"
        },
        "object": {
          "key": "path/to/file.jpg"
        }
      }
    }
  ]
}
```

#### API Gateway Events
```json
{
  "httpMethod": "GET",
  "path": "/users/123",
  "headers": {
    "Content-Type": "application/json"
  },
  "queryStringParameters": {
    "filter": "active"
  },
  "body": "{\"name\":\"John\"}",
  "requestContext": {
    "requestId": "abc123",
    "stage": "prod"
  }
}
```

#### DynamoDB Stream Events
```json
{
  "Records": [
    {
      "eventID": "1",
      "eventName": "INSERT",
      "dynamodb": {
        "Keys": {
          "id": {"S": "123"}
        },
        "NewImage": {
          "id": {"S": "123"},
          "name": {"S": "John"}
        }
      }
    }
  ]
}
```

#### SQS Events
```json
{
  "Records": [
    {
      "messageId": "abc123",
      "body": "{\"userId\":\"123\",\"action\":\"update\"}",
      "attributes": {
        "ApproximateReceiveCount": "1"
      }
    }
  ]
}
```

#### EventBridge (Scheduled) Events
```json
{
  "version": "0",
  "id": "event-id",
  "detail-type": "Scheduled Event",
  "source": "aws.events",
  "time": "2024-01-01T00:00:00Z",
  "region": "us-east-1",
  "detail": {}
}
```

---

## 5. Lambda Function Structure

### 5.1 Basic Function Structure

#### Python Example
```python
import json

# Global scope - runs once per cold start
# Good for: Initializing connections, loading data
database_connection = initialize_database()

def lambda_handler(event, context):
    """
    Handler function - runs on every invocation
    
    Args:
        event: Event data from trigger
        context: Runtime information
    
    Returns:
        Response object
    """
    # Extract data from event
    user_id = event.get('userId')
    
    # Business logic
    result = process_user(user_id)
    
    # Return response
    return {
        'statusCode': 200,
        'body': json.dumps({
            'message': 'Success',
            'data': result
        })
    }

def process_user(user_id):
    # Your business logic here
    return {'userId': user_id, 'processed': True}
```

#### Node.js Example
```javascript
// Global scope - runs once per cold start
const AWS = require('aws-sdk');
const s3 = new AWS.S3();

exports.handler = async (event, context) => {
    // Extract data from event
    const userId = event.userId;
    
    // Business logic
    const result = await processUser(userId);
    
    // Return response
    return {
        statusCode: 200,
        body: JSON.stringify({
            message: 'Success',
            data: result
        })
    };
};

async function processUser(userId) {
    // Your business logic here
    return { userId, processed: true };
}
```

#### Java Example
```java
import com.amazonaws.services.lambda.runtime.Context;
import com.amazonaws.services.lambda.runtime.RequestHandler;
import java.util.Map;

public class MyLambdaFunction implements RequestHandler<Map<String, Object>, Map<String, Object>> {
    
    // Global scope - runs once per cold start
    private DatabaseConnection dbConnection;
    
    public MyLambdaFunction() {
        // Initialize connections
        this.dbConnection = new DatabaseConnection();
    }
    
    @Override
    public Map<String, Object> handleRequest(Map<String, Object> event, Context context) {
        // Extract data from event
        String userId = (String) event.get("userId");
        
        // Business logic
        Map<String, Object> result = processUser(userId);
        
        // Return response
        return Map.of(
            "statusCode", 200,
            "body", Map.of(
                "message", "Success",
                "data", result
            )
        );
    }
    
    private Map<String, Object> processUser(String userId) {
        // Your business logic here
        return Map.of("userId", userId, "processed", true);
    }
}
```

### 5.2 Handler Function Signature

#### Python
```python
def lambda_handler(event, context):
    # event: Dict containing event data
    # context: LambdaContext object with runtime info
    pass
```

#### Node.js
```javascript
exports.handler = async (event, context) => {
    // event: Object containing event data
    // context: Context object with runtime info
};
```

#### Java
```java
public class Handler implements RequestHandler<InputType, OutputType> {
    public OutputType handleRequest(InputType input, Context context) {
        // input: Event data
        // context: Context object with runtime info
    }
}
```

### 5.3 Context Object

The `context` object provides runtime information:

```python
def lambda_handler(event, context):
    # Function information
    function_name = context.function_name
    function_version = context.function_version
    memory_limit = context.memory_limit_in_mb
    
    # Request information
    request_id = context.aws_request_id
    log_group_name = context.log_group_name
    log_stream_name = context.log_stream_name
    
    # Remaining time
    remaining_time = context.get_remaining_time_in_millis()
    
    return {'statusCode': 200}
```

---

## 6. Supported Runtimes

### 6.1 Current Supported Runtimes

| Runtime | Version | Use Case |
|---------|---------|----------|
| **Python** | 3.9, 3.10, 3.11, 3.12 | Data processing, APIs, automation |
| **Node.js** | 18.x, 20.x | Web APIs, real-time applications |
| **Java** | 8, 11, 17, 21 | Enterprise applications, microservices |
| **.NET** | 6, 8 | .NET applications, C# services |
| **Go** | 1.x | High-performance services, CLI tools |
| **Ruby** | 3.2 | Web applications, scripting |
| **Custom Runtime** | Any | Any language via Runtime API |

### 6.2 Runtime Selection Criteria

**Choose Python when**:
- Data processing and analytics
- Machine learning inference
- Scripting and automation
- Rapid prototyping

**Choose Node.js when**:
- Web APIs and REST services
- Real-time applications
- Event-driven architectures
- Full-stack JavaScript applications

**Choose Java when**:
- Enterprise applications
- Existing Java codebase
- Spring Boot applications
- High-performance requirements

**Choose Go when**:
- High-performance services
- Concurrent processing
- CLI tools
- Minimal cold start requirements

---

## 7. Configuration and Limits

### 7.1 Memory Configuration

```
Memory Range: 128 MB to 10,240 MB (10 GB)
CPU Allocation: Proportional to memory
  - 1,769 MB = 1 vCPU
  - Below 1,769 MB = Fractional vCPU
  - Above 1,769 MB = Multiple vCPUs
```

**Memory Guidelines**:
- **128-512 MB**: Simple functions, API responses
- **512 MB - 1 GB**: Database queries, moderate processing
- **1-3 GB**: Image processing, data transformation
- **3-10 GB**: Machine learning, large data processing

### 7.2 Timeout Configuration

```
Timeout Range: 1 second to 15 minutes (900 seconds)
Default: 3 seconds
```

**Timeout Guidelines**:
- **1-30 seconds**: API responses, quick processing
- **30-60 seconds**: Database operations, external API calls
- **1-5 minutes**: File processing, batch operations
- **5-15 minutes**: Long-running ETL, data processing

### 7.3 Lambda Limits

| Limit | Value | Notes |
|-------|-------|-------|
| **Function Memory** | 128 MB - 10 GB | Configurable |
| **Function Timeout** | 15 minutes | Maximum |
| **Deployment Package** | 50 MB (zipped) | Direct upload |
| **Deployment Package** | 250 MB (unzipped) | Direct upload |
| **Deployment Package** | 250 MB (zipped) | Via S3 |
| **Deployment Package** | 3 GB (unzipped) | Via S3 |
| **Ephemeral Storage** | 512 MB - 10 GB | /tmp directory |
| **Environment Variables** | 4 KB | Total size |
| **Concurrent Executions** | 1,000 (default) | Can be increased |
| **Invocation Payload** | 6 MB (synchronous) | Request/response |
| **Invocation Payload** | 256 KB (asynchronous) | Event payload |

### 7.4 Environment Variables

```python
import os

def lambda_handler(event, context):
    # Access environment variables
    database_url = os.environ['DATABASE_URL']
    api_key = os.environ.get('API_KEY', 'default')
    
    # Use in your code
    connect_to_database(database_url)
```

**Best Practices**:
- Store sensitive data in AWS Secrets Manager or Parameter Store
- Use environment variables for configuration
- Never commit secrets in code
- Use different values per environment (dev, staging, prod)

---

## 8. Execution Model

### 8.1 Concurrency Model

```
┌─────────────────────────────────────────────────────────┐
│ Lambda Function                                          │
├─────────────────────────────────────────────────────────┤
│                                                          │
│ Concurrent Executions:                                  │
│ ┌────────┐ ┌────────┐ ┌────────┐ ┌────────┐           │
│ │Exec 1  │ │Exec 2  │ │Exec 3  │ │Exec 4  │ ...        │
│ │        │ │        │ │        │ │        │           │
│ │Event 1 │ │Event 2 │ │Event 3 │ │Event 4 │           │
│ └────────┘ └────────┘ └────────┘ └────────┘           │
│                                                          │
│ Each execution is isolated:                              │
│ - Separate execution environment                        │
│ - Independent memory and CPU                            │
│ - No shared state                                        │
└─────────────────────────────────────────────────────────┘
```

### 8.2 Reserved Concurrency

```
Total Concurrency: 1,000 (default)
Reserved Concurrency: 100 (for Function A)
Available Concurrency: 900 (for other functions)

Function A: Maximum 100 concurrent executions
Other Functions: Share remaining 900
```

**Use Cases for Reserved Concurrency**:
- Critical functions that must always execute
- Functions that consume downstream resources
- Rate limiting for external APIs

### 8.3 Provisioned Concurrency

```
Provisioned Concurrency: 10
- Keeps 10 execution environments warm
- Eliminates cold starts for those environments
- Costs apply even when not in use
```

**Use Cases**:
- Low-latency requirements (< 100ms)
- Predictable traffic patterns
- Critical user-facing functions

---

## 9. Use Cases

### 9.1 Real-Time File Processing

**Scenario**: Process images uploaded to S3
```python
def lambda_handler(event, context):
    for record in event['Records']:
        bucket = record['s3']['bucket']['name']
        key = record['s3']['object']['key']
        
        # Download image
        image = download_from_s3(bucket, key)
        
        # Process (resize, compress, etc.)
        processed = process_image(image)
        
        # Upload processed image
        upload_to_s3(processed, bucket, f'processed/{key}')
```

### 9.2 API Backend

**Scenario**: RESTful API using API Gateway + Lambda
```python
def lambda_handler(event, context):
    method = event['httpMethod']
    path = event['path']
    
    if method == 'GET' and path == '/users':
        users = get_all_users()
        return {
            'statusCode': 200,
            'body': json.dumps(users)
        }
    elif method == 'POST' and path == '/users':
        user_data = json.loads(event['body'])
        user = create_user(user_data)
        return {
            'statusCode': 201,
            'body': json.dumps(user)
        }
```

### 9.3 Data Transformation

**Scenario**: Transform data from one format to another
```python
def lambda_handler(event, context):
    # Triggered by SQS message
    for record in event['Records']:
        data = json.loads(record['body'])
        
        # Transform data
        transformed = transform_data(data)
        
        # Send to destination
        send_to_destination(transformed)
```

### 9.4 Scheduled Tasks

**Scenario**: Run periodic tasks (cron jobs)
```python
def lambda_handler(event, context):
    # Triggered by EventBridge schedule
    # Runs every day at 2 AM
    
    # Generate daily report
    report = generate_daily_report()
    
    # Send email
    send_email_report(report)
```

### 9.5 Real-Time Stream Processing

**Scenario**: Process Kinesis stream data
```python
def lambda_handler(event, context):
    for record in event['Records']:
        data = json.loads(record['kinesis']['data'])
        
        # Process stream record
        result = process_stream_record(data)
        
        # Store results
        store_results(result)
```

---

## 10. Creating Your First Lambda Function

### 10.1 Using AWS Console

**Step 1: Create Function**
1. Go to Lambda Console
2. Click "Create function"
3. Choose "Author from scratch"
4. Enter function name
5. Select runtime (e.g., Python 3.11)
6. Click "Create function"

**Step 2: Write Code**
```python
def lambda_handler(event, context):
    return {
        'statusCode': 200,
        'body': 'Hello from Lambda!'
    }
```

**Step 3: Configure**
- Memory: 128 MB
- Timeout: 3 seconds
- Environment variables: (optional)

**Step 4: Test**
- Click "Test"
- Create test event
- Run test
- View results

### 10.2 Using AWS CLI

```bash
# Create deployment package
zip function.zip lambda_function.py

# Create function
aws lambda create-function \
    --function-name my-function \
    --runtime python3.11 \
    --role arn:aws:iam::123456789012:role/lambda-role \
    --handler lambda_function.lambda_handler \
    --zip-file fileb://function.zip

# Update function code
aws lambda update-function-code \
    --function-name my-function \
    --zip-file fileb://function.zip

# Invoke function
aws lambda invoke \
    --function-name my-function \
    --payload '{"key":"value"}' \
    response.json
```

### 10.3 Using Infrastructure as Code (Terraform)

```hcl
resource "aws_lambda_function" "my_function" {
  filename         = "function.zip"
  function_name    = "my-function"
  role            = aws_iam_role.lambda_role.arn
  handler         = "lambda_function.lambda_handler"
  source_code_hash = filebase64sha256("function.zip")
  runtime         = "python3.11"
  memory_size     = 128
  timeout         = 3

  environment {
    variables = {
      ENV = "production"
    }
  }
}
```

### 10.4 Using SAM (Serverless Application Model)

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Resources:
  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: lambda_function.lambda_handler
      Runtime: python3.11
      CodeUri: ./
      MemorySize: 128
      Timeout: 3
      Events:
        ApiEvent:
          Type: Api
          Properties:
            Path: /hello
            Method: get
```

---

## Summary of Part 1

**Key Takeaways**:
1. **Lambda is serverless**: No server management required
2. **Event-driven**: Executes in response to events from various sources
3. **Auto-scaling**: Handles traffic automatically
4. **Pay-per-use**: Cost-effective for variable workloads
5. **Multiple runtimes**: Supports Python, Node.js, Java, Go, .NET, Ruby
6. **Cold vs Warm starts**: First invocation slower, subsequent faster
7. **Configuration**: Memory (128MB-10GB), Timeout (15 min max)
8. **Use cases**: APIs, file processing, data transformation, scheduled tasks

**Next**: Part 2 will cover advanced topics including:
- Performance optimization
- Error handling and retries
- Monitoring and debugging
- Security best practices
- Integration patterns
- Cost optimization
- Best practices and anti-patterns

