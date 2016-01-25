# AWS Lambda Reference Architecture:  IoT Backend

The Internet of Things (IoT) Backend reference architecture ([diagram](https://s3.amazonaws.com/awslambda-reference-architectures/iot-backend/lambda-refarch-iotbackend.pdf)) demonstrates how to use AWS Lambda in conjunction with Amazon Kinesis, Amazon DynamoDB, Amazon Simple Storage Service (Amazon S3), and Amazon CloudWatch to build a serverless system for ingesting and processing sensor data. By leveraging these services, you can build cost-efficient applications that can meet the massive scale required for processing the data generated by huge deployments of connected devices.

This repository contains sample code for all the Lambda functions depicted in this [diagram](https://s3.amazonaws.com/awslambda-reference-architectures/iot-backend/lambda-refarch-iotbackend.pdf) as well as a AWS CloudFormation template for creating the functions and related resources. There is also a simple webpage that you can run locally to publish sample events and query the data from DynamodDB.

## Running the Example

The entire example system can be deployed in us-east-1 using the provided CloudFormation template and an S3 bucket. If you would like to deploy the template to a different region, you must copy the Lambda deployment packages under the `iot-backend` prefix in the `awslambda-reference-architectures` bucket to a new S3 bucket in your target region. You can then provide this new bucket as a parameter when launching the template.

Choose **Launch Stack** to launch the template in the us-east-1 region in your account:

[![Launch Lambda IoT Backend into North Virginia with CloudFormation](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/images/cloudformation-launch-stack-button.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=lambda-iot-backend&templateURL=https://s3.amazonaws.com/awslambda-reference-architectures/iot-backend/iot-backend.template)

## Testing the Example

You can use the [test webpage](testpage.html) to test the system as follows:

1. Save the [testpage.html](testpage.html) file to your local system.
1. Open the downloaded file with a text editor and fill in the configuration values using the outputs of the CloudFormation stack launched in the previous section.
1. Open your edited copy of testpage.html using a web browser of your choice.

After you launch the test page, you can simulate the submission of sensor data from multiple devices, as well as query the DynamoDB table for the historical data of a given device. In addition to using the test page to query DynamoDB, you can check the CloudWatch metrics published under the `Sensor` namespace.

## Cleaning Up the Example Resources

To remove all resources created by this example, do the following:

1. Delete all objects from the `ArchiveBucket` created by the CloudFormation stack.
1. Delete the CloudFormation stack.
1. Delete all CloudWatch log groups for each of the Lambda functions in the stack.

## CloudFormation Template Resources

The following sections explain all of the resources created by the CloudFormation template provided with this example.

### Lambda functions

- **ApiFunction** - A Lambda function that provides a simple API for querying the sensor data stored in the `SensorDataTable`.

- **DdbCloudWatchEventProcessorFunction** - A Lambda function that processes events from `EventStream` and persists them to both the `SensorDataTable` and a custom CloudWatch metric under the `Sensor` namespace.

- **EventArchiverFunction** - A Lambda function that processes events from `EventStream` and archives the raw data in `ArchiveBucket`.


### Function roles

- **ApiExecutionRole** - An AWS Identity and Access Management (IAM) role assumed by the `ApiFunction`. This role provides logging permissions and access to query `SensorDataTable`. It also enables the function to call `GetFunction` in order to read configuration data from the function's description.

- **DdbCloudWatchProcessorRole** - An IAM role assumed by the `DdbCloudWatchEventProcessorFunction`. This role provides permissions for logging, writing items to `SensorDataTable`, and publishing custom CloudWatch metrics. It also enables the function to call `GetFunction` in order to read configuration data from the function's description.

- **EventArchiverRole** - An IAM role assumed by the `EventArchiverRole`. This role provides logging permissions and access to put objects to `ArchiveBucket`. It also enables the function to call `GetFunction` in order to read configuration data from the function's description.

### Event source mappings

- **DdbCloudWatchProcessorSourceMapping** - An event source mapping that enables `DdbCloudWatchEventProcessorFunction` to process records from `EventStream`.

- **EventArchiverSourceMapping** - An event source mapping that enables `EventArchiverFunction` to process records from `EventStream`.

### IAM Users and Policies

- **TestClientUser** - An IAM user used by the test webpage.

- **TestClientPolicy** - An IAM policy attached to `TestClientUser` that grants access to put records on the `EventStream` and invoke the `ApiFunction`.

- **TestClientKeys** - Access keys that enable the test webpage to sign API requests in order to simulate device events and query the `SensorDataTable`.


### Other Resources

- **EventStream** - An Amazon Kinesis stream to receive the raw sensor data.

- **SensorDataTable** - A DynamoDB table to store the processed sensor data.

- **ArchiveBucket** - An S3 Bucket for archiving the raw sensor data.


## License

This reference architecture sample is licensed under Apache 2.0.
