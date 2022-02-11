# Introducing [Lambda Powertools for python](https://awslabs.github.io/aws-lambda-powertools-python/latest/)

This repo contains documentation for a live coding workshop for the AWS Programminng and Tools Meetup in Melbourne.  The workshop will start with the SAM Cli "Hello World" example API project.  Throughout the labs we will introduce each of the AWS Lambda Powertools Core utilities to showcase how simple they are to use and adopt for all your projects, and how powerful they are at bringing you closer to the Well Architected Serverless Lens.

This lab assumes zero knowledge and is a great starting point if you are curious about Serverless development with Python and SAM Cli.  By the end of the lab you should be familiar with the following:

- Creating a project with SAM Cli
- Building a Serverless Project
- Deploying a Serverless project with SAM Cli
- Installing aws-lambda-powertools (for python)
- Linking the Global AWS Lambda Powertools Layer into your project

The labs presented here will shocwase the core Lambda Powertools utilities: Logger, Tracer, Metrics and Event Source Data Classes.  With these 4 simple additions to your projects you will supercharge your lambda skills and best practice!

## Lab 1: Cloud9 IDE Setup

This lab will walk through Cloud9 IDE setup and introduce the project and deploy the core "Hello World" API as a starting point.

1. Create Cloud9 IDE environment from the console    
	- Create a standard Amazon Linux 2 Environment
	- bootup into the IDE
2. Create a SAM cli Python REST API project
	- use the sam wizard: sam init
		- Choose: 1 AWS Quick Start Templates 
		- Choose: 1 Zip artefact
		- Runtime: 10 - Python 3.7
		- name the project: sam-app
		- Template Selection: Hello World Example
3. Open terminal: cd sam-app
4. Build the Project: sam build
5. Deploy the Project: sam deploy --guided
	- Note: Must use guided deploy since this will also bootstrap the sam cli deployment process
	- step through the wizard
	- **note**: if asked about rollback support select 'N' since this will stop future changes in other labs.

## Lab 2: Logging, Tracing and Metrics

In this lab we will introduce each of the core utilities one at a time.  But first we will install the AWS Lambda Powertools and complete the IDE configuration.  The Cloud9 IDE default environment includes Python 3.7 by default and SAM Cli v1.33 so this lab assumes these as a basis and I have adjusted the lab steps to accomodate this environment.

### Part 1: Install Lambda Powertools

```bash
$ pip install aws-lambda-powertools

```

This installs the Lambda Powertools to a local IDE folder: /home/ec2-user/.local/lib/python3.7/site-packages

Before we continue we need to update the IDE PYTHONPATH so Type hinting works.  Update the PYTHONPATH to /home/ec2-user/.local/lib/python3.7/site-packages

Now we should have type hints working :-)

#### template.yaml (add Powertools Layer):

```yaml
Transform: AWS::Serverless-2016-10-31
Description: >
  sam-app

  Sample SAM Template for sam-app

# More info about Globals: https://github.com/awslabs/serverless-application-model/blob/master/docs/globals.rst
Globals:
  Function:
    Timeout: 3

Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function # More info about Function Resource: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction
    Properties:
      Layers:
        - !Sub arn:aws:lambda:${AWS::Region}:017000801446:layer:AWSLambdaPowertoolsPython:9
      CodeUri: hello_world/
      Handler: app.lambda_handler
      Runtime: python3.7
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get

```         

### Part 2: [Logger](https://awslabs.github.io/aws-lambda-powertools-python/latest/core/logger/)

Look at the JSON structured opionated logging - Just add the decorator as a starting point - click the title link to view the documentation.

#### hello_world/app.py (add logger):

**Important Note:** We have added `log_event=True` for demonstration purposes - but really you need to think hard on using this flag since some events in real world systems will transport Personally Identifiable Information (PII) so be wary!

```python
import json
from aws_lambda_powertools import Logger
from aws_lambda_powertools.logging import correlation_paths

logger = Logger()

@logger.inject_lambda_context(correlation_id_path=correlation_paths.API_GATEWAY_REST, log_event=True)
def lambda_handler(event, context):
    
    return {
        "statusCode": 200,
        "body": json.dumps({
            "message": "hello world"
        }),
    }
```

### Part 3: [Tracer](https://awslabs.github.io/aws-lambda-powertools-python/latest/core/tracer/)

A look at the Out of the Box Tracer.

#### hello_world/app.py (add Tracer):
```python
import json
from aws_lambda_powertools import Logger
from aws_lambda_powertools.logging import correlation_paths
from aws_lambda_powertools import Tracer

logger = Logger()
tracer = Tracer()

@logger.inject_lambda_context(correlation_id_path=correlation_paths.API_GATEWAY_REST, log_event=True)
@tracer.capture_lambda_handler()
def lambda_handler(event, context):
    
    return {
        "statusCode": 200,
        "body": json.dumps({
            "message": "hello world"
        }),
    }


```

#### template.yaml (activate XRAY):

```yaml
Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function # More info about Function Resource: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md#awsserverlessfunction
    Properties:
      Tracing: Active
      CodeUri: hello_world/
      Handler: app.lambda_handler
      Runtime: python3.7
      Events:
        HelloWorld:
          Type: Api 
          Properties:
            Path: /hello
            Method: get
      Environment:
        Variables:
          POWERTOOLS_SERVICE_NAME: aws_meetup
```

#### Lets add XRay [Annotations](https://awslabs.github.io/aws-lambda-powertools-python/latest/core/tracer/#annotations-metadata):


```python
import json
from aws_lambda_powertools import Logger
from aws_lambda_powertools.logging import correlation_paths
from aws_lambda_powertools import Tracer

logger = Logger()
tracer = Tracer()

@logger.inject_lambda_context(correlation_id_path=correlation_paths.API_GATEWAY_REST, log_event=True)
@tracer.capture_lambda_handler()
def lambda_handler(event, context):
    tracer.put_annotation(key="HelloMeetup", value="SUCCESS") 
    return {
        "statusCode": 200,
        "body": json.dumps({
            "message": "hello world"
        }),
    }

```

### Part 4: [Custom Metrics](https://awslabs.github.io/aws-lambda-powertools-python/latest/core/metrics/#creating-metrics)

Custom metrics are overlooked when it comes to observability - abscence of metrics can indicate a problem that may need to be looked at - I am guilty of not using Custom metrics enough!

```python
import json
from aws_lambda_powertools import Logger
from aws_lambda_powertools.logging import correlation_paths
from aws_lambda_powertools import Tracer
from aws_lambda_powertools import Metrics
from aws_lambda_powertools.metrics import MetricUnit
from aws_lambda_powertools.utilities.data_classes import event_source, APIGatewayProxyEvent



logger = Logger()
tracer = Tracer()
metrics = Metrics(namespace="AWSMeetup", service="HelloWorld")

@metrics.log_metrics(capture_cold_start_metric=True)
@logger.inject_lambda_context(correlation_id_path=correlation_paths.API_GATEWAY_REST, log_event=True)
@tracer.capture_lambda_handler()
@event_source(data_class=APIGatewayProxyEvent)
def lambda_handler(event: APIGatewayProxyEvent, context):
    metrics.add_metric(name="HelloWorld", unit=MetricUnit.Count, value=1)
    tracer.put_annotation(key="HelloMeetup", value="SUCCESS") 
    return {
        "statusCode": 200,
        "body": json.dumps({
            "message": "hello world"
        }),
    }

```

## Lab 3: [Event Source Data Classes](https://awslabs.github.io/aws-lambda-powertools-python/latest/utilities/data_classes/#utilizing-the-data-classes)

Event Source data classes make working with Lambda events really simple.  Lets see how they can work for you to simplify and improve your developer experience.

### Lets add the API Gateway Proxy Data Class:

This is the quick way to use Powertools Data Classes

```python
import json
from aws_lambda_powertools import Logger
from aws_lambda_powertools.logging import correlation_paths
from aws_lambda_powertools import Tracer
from aws_lambda_powertools import Metrics
from aws_lambda_powertools.metrics import MetricUnit
from aws_lambda_powertools.utilities.data_classes import APIGatewayProxyEvent


logger = Logger()
tracer = Tracer()
metrics = Metrics(namespace="AWSMeetup", service="HelloWorld")

@metrics.log_metrics(capture_cold_start_metric=True)
@logger.inject_lambda_context(correlation_id_path=correlation_paths.API_GATEWAY_REST, log_event=True)
@tracer.capture_lambda_handler()
def lambda_handler(event, context):
    event = APIGatewayProxyEvent(event)
    
    metrics.add_metric(name="HelloWorld", unit=MetricUnit.Count, value=1)
    tracer.put_annotation(key="HelloMeetup", value="SUCCESS") 
    return {
        "statusCode": 200,
        "body": json.dumps({
            "message": "hello world"
        }),
    }
```

### Using the Event Source Decorator

With the event source decorator you can have proper typed parameters for your Lambda function.  This makes working with Lambda events super-simple since the IDE will start to offer type hints and work for you.

```python
import json
from aws_lambda_powertools import Logger
from aws_lambda_powertools.logging import correlation_paths
from aws_lambda_powertools import Tracer
from aws_lambda_powertools import Metrics
from aws_lambda_powertools.metrics import MetricUnit
from aws_lambda_powertools.utilities.data_classes import event_source, APIGatewayProxyEvent

logger = Logger()
tracer = Tracer()
metrics = Metrics()

@metrics.log_metrics(capture_cold_start_metric=True)
@logger.inject_lambda_context(correlation_id_path=correlation_paths.API_GATEWAY_REST, log_event=True)
@tracer.capture_lambda_handler()
@event_source(data_class=APIGatewayProxyEvent)
def lambda_handler(event: APIGatewayProxyEvent, context):
    metrics.add_metric(name="HelloWorld", unit=MetricUnit.Count, value=1)
    tracer.put_annotation(key="HelloMeetup", value="SUCCESS") 
    return {
        "statusCode": 200,
        "body": json.dumps({
            "message": "hello world"
        }),
    }
```

## Lab Extensions: 

### Event handlers: [REST API](https://awslabs.github.io/aws-lambda-powertools-python/latest/core/event_handler/api_gateway/#api-gateway-decorator)

In this lab we will introduce the REST Api Event Handler (ApiGatewayResolver) and see how it makes creating an API simple and easy.  We will explore the type hinting capability and showcase the core API utilities which make creating APIs simple and easy to work with.

This component tackles REST APIS for Amazon API Gateway REST/HTTP APIs and Application Loader Balancer (ALB).

If you are familiar with Flask then this will make you feel right at home.


### [Batch Processing](https://awslabs.github.io/aws-lambda-powertools-python/latest/utilities/batch/)

The Batch Utilities are awesome and make SQS batch processing super simple and robust!  I do hope we can get to this one to showcase the best practice in processing from an SQS queue!
