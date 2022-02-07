# Introducing [Lambda Powertools for python](https://awslabs.github.io/aws-lambda-powertools-python/latest/)

This repo contains documentation for a live coding workshop for the AWS Programminng and Tools Meetup in Melbourne.  The workshop will start with the SAM Cli "Hello World" example API project.  Throughout the labs we will introduce each of the AWS Lambda Powertools Core utilities to showcase how simple they are to use and adopt for all your projects, and how powerful they are at bringing you closer to the Well Architected Serverless Lens.

This lab assumes zero knowledge and is a great starting point if you are curious about Serverless development with Python and SAM Cli.  By the end of the lab you should be familiar with the following:

- Creating a project with SAM Cli
- Building a Serverless Project
- Deploying a Serverless project with SAM Cli
- Installing aws-lambda-powertools (for python)
- Linking the Global AWS Lambda Powertools Layer into your project

## Lab 1: Cloud9 IDE Setup

This lab will walk through Cloud9 IDE setup and introduce the project and deploy the core "Hello World" API as a starting point.

1. Create Cloud9 IDE environment from the console    
	- Create a standard Amazon Linux 2 Environment
	- bootup into the IDE
2. Create a SAM cli Python REST API project
	- use the sam wizard: sam init
		- Choose: 1 Quick Start Templates 
		- Choose: 1 Zip artefact
		- Runtime: 10 - Python 3.7
		- name the project: sam-app
3. Open terminal: cd sam-app
4. Build the Project: sam build
5. Deploy the Project: sam deploy --guided
	- Note: Must use guided deploy since this will also bootstrap the sam cli deployment process
	- step through the wizard
	- **note**: if asked about rollback support select 'N' since this will stop future changes in other labs.

## Lab 2: Logging, Tracing and Metrics

In this lab we will introduce each of the core utilities on at a time.  But first we will install the AWS Lambda Powertools and complete the IDE configuration.  The Cloud9 IDE default environment includes Python 3.7 by default and SAM Cli v1.33 so this lab assumes these as a basis and I have adjusted the lab steps to accomodate this environment.

### Part 1: Install Lambda Powertools

```bash
$ pip install aws-lambda-powertools

```

This installs the Lambda Powertools to a local IDE folder: /home/ec2-user/.local/lib/python3.7/site-packages

Before we continue we need to update the IDE PYTHONPATH so Type hinting works - I really struggled to get a good IDE experience Out of the Box here - maybe I could learn something I am a first time Cloud9 user.

### Part 2: [Logger](https://awslabs.github.io/aws-lambda-powertools-python/latest/core/logger/)

Look at the JSON structured opionated logging - Just add the decorator as a starting point.

### Part 3: [Tracer](https://awslabs.github.io/aws-lambda-powertools-python/latest/core/tracer/)

A look at the Out of the Box Tracer.

### Part 4: [Custom Metrics](https://awslabs.github.io/aws-lambda-powertools-python/latest/core/metrics/)

Custom metrics are overlooked when it comes to observability - abscence of metrics can indicate a problem that may need to be looked at - I am guilty of not using Custom metrics enough!

## Lab 3: Event handlers: REST API 

In this lab we will introduce the REST Api Event Handler (ApiGatewayResolver) and see how it makes creating an API simple and easy.  We will explore the type hinting capability and showcase the core API utilities which make creating APIs simple and easy to work with.

This component tackles REST APIS for Amazon API Gateway REST/HTTP APIs and Application Loader Balancer (ALB).

If you are familiar with Flask then this will make you feel right at home.


## Lab Extensions: 

### [Event Source Data Classes](https://awslabs.github.io/aws-lambda-powertools-python/latest/utilities/data_classes/)

Event Source data classes make working with Lambda events really simple.  Lets see how they can work for you to simplify and improve your developer experience.

### [Batch Processing](https://awslabs.github.io/aws-lambda-powertools-python/latest/utilities/batch/)

The Batch Utilities are awesome and make SQS batch processing super simple and robust!  I do hope we can get to this one to showcase the best practice in processing from an SQS queue!
