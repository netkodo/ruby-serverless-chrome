# ruby-serverless-chrome

Want to run serverless code that uses headeless Chrome, but want to use Ruby?

You're in luck!  I wanted to do that too, and I figured out how.  This setup uses the [`serverless-chrome`](https://github.com/adieuadieu/serverless-chrome) headless Chromium project, but it does not use Puppeteer and Node.js scripting code.  Instead, it uses the [Selenium Webdriver](https://github.com/SeleniumHQ/selenium/tree/master/rb) Ruby gem and Ruby scraper code.

## Overview

### SAM

The Serverless Application Model Command Line Interface (SAM CLI) is an extension of the AWS CLI that adds functionality for building and testing Lambda applications. It uses Docker to run your functions in an Amazon Linux environment that matches Lambda. It can also emulate your application's build environment and API.

To use the SAM CLI, you need the following tools.

* AWS CLI - [Install the AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-install.html) and [configure it with your AWS credentials].
* SAM CLI - [Install the SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)
* Ruby - [Install Ruby 2.5](https://www.ruby-lang.org/en/documentation/installation/)
* Docker - [Install Docker community edition](https://hub.docker.com/search/?type=edition&offering=community)

### This project

This project contains source code and supporting files for a serverless application that you can deploy with the SAM CLI. It includes the following files and folders.

- scraper - Code for the application's Lambda function.
- events - Invocation events that you can use to invoke the function.
- template.yaml - A template that defines the application's AWS resources.

The application uses several AWS resources, including Lambda functions and an API Gateway API. These resources are defined in the `template.yaml` file in this project. You can update the template to add AWS resources through the same deployment process that updates your application code.

## Getting started

### Fetch binary dependencies

You will need to fetch the Headless Chromium browser and the Chromedriver project.  Run this script to fetch those:

```bash
ruby-serverless-chrome$ script/fetch-dependencies.sh
```

That should fetch binaries for both into your `scraper/bin` folder, where your Lambda function code will be able to use them.

## Use the SAM CLI to build and test locally

Build your application with the `sam build` command.

```bash
ruby-serverless-chrome$ sam build --use-container
```

The SAM CLI installs dependencies defined in `scraper/Gemfile`, creates a deployment package, and saves it in the `.aws-sam/build` folder.

Test a single function by invoking it directly with a test event. An event is a JSON document that represents the input that the function receives from the event source. Test events are included in the `events` folder in this project.

Run functions locally and invoke them with the `sam local invoke` command.

```bash
ruby-serverless-chrome$ sam local invoke ScraperFunction --event events/event.json
```

The SAM CLI can also emulate your application's API. Use the `sam local start-api` to run the API locally on port 3000.

```bash
ruby-serverless-chrome$ sam local start-api
ruby-serverless-chrome$ curl http://localhost:3000/scrape
```

## Deploy the application

The SAM CLI uses an Amazon S3 bucket to store your application's deployment artifacts. If you don't have a bucket suitable for this purpose, create one. Replace `BUCKET_NAME` in the commands in this section with a unique bucket name.

```bash
ruby-serverless-chrome$ aws s3 mb s3://BUCKET_NAME
```

To prepare the application for deployment, use the `sam package` command.

```bash
ruby-serverless-chrome$ sam package \
    --output-template-file packaged.yaml \
    --s3-bucket BUCKET_NAME
```

The SAM CLI creates deployment packages, uploads them to the S3 bucket, and creates a new version of the template that refers to the artifacts in the bucket.

To deploy the application, use the `sam deploy` command.

```bash
ruby-serverless-chrome$ sam deploy \
    --template-file packaged.yaml \
    --stack-name ruby-serverless-chrome \
    --capabilities CAPABILITY_IAM
```

After deployment is complete you can run the following command to retrieve the API Gateway Endpoint URL:

```bash
ruby-serverless-chrome$ aws cloudformation describe-stacks \
    --stack-name ruby-serverless-chrome \
    --query 'Stacks[].Outputs[?OutputKey==`ScraperApi`]' \
    --output table
```

## Fetch, tail, and filter Lambda function logs

To simplify troubleshooting, SAM CLI has a command called `sam logs`. `sam logs` lets you fetch logs generated by your deployed Lambda function from the command line. In addition to printing the logs on the terminal, this command has several nifty features to help you quickly find the bug.

`NOTE`: This command works for all AWS Lambda functions; not just the ones you deploy using SAM.

```bash
ruby-serverless-chrome$ sam logs -n ScraperFunction --stack-name ruby-serverless-chrome --tail
```

You can find more information and examples about filtering Lambda function logs in the [SAM CLI Documentation](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-logging.html).

## Cleanup

To delete the sample application and the bucket that you created, use the AWS CLI.

```bash
ruby-serverless-chrome$ aws cloudformation delete-stack --stack-name ruby-serverless-chrome
ruby-serverless-chrome$ aws s3 rb s3://BUCKET_NAME
```

## Resources

See the [AWS SAM developer guide](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html) for an introduction to SAM specification, the SAM CLI, and serverless application concepts.
