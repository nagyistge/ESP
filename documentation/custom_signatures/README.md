# Evident Security Platform (ESP) Custom Signatures Guide

## Introduction
Evident Security Platform (ESP) Signatures are the checks that run on top of the AWS API and Config data collected across your AWS accounts. There are two types of ESP Signatures, the default ones, built-in and maintained by Evident, and Custom Signatures maintained by you.  Custom signatures are enabled for ESP enterprise accounts and allow you to extend the functionality of ESP by enhancing, or even replacing, built-in signatures. 

The Custom Signatures DSL is an interpreted JavaScript language. The AWS-specific objects used follow the [AWS SDK for Ruby standards](http://docs.aws.amazon.com/sdkforruby/api/frames.html). ESP Custom signatures can be created here: https://esp.evident.io/control_panel/custom_signatures/new.

## Configuration Parameters
Every custom signature requires a `dsl.configure` function. It sets up parameters needed for the custom signature engine.

1. `c.module` is currently required to validate signatures and is used by built-in ESP signatures to define method names and classes dynamically during runtime. 
2. `c.identifier` is required for reports and being able to search for your specific custom signature. 
3. `c.description` is required for the descriptions on signatures within the metascrape console and to validate the signature.
4. `c.valid_regions` is not required. This is an array of regions that the signature should only run in. This allows you to isolate what regions to run the signature in. Ex [‘us_east_1’] meaning only run this signature in us-east-1.
5. `c.display_as` is not required. This was added for signatures when they are global, i.e. a signature is run in us-east-1, but the signature has no requirement on what region it runs in because the result is the same.
6. `c.deep_inspection` is not required. Deep inspection allows you to set extra data on the alert or alerts that the signature is returning. The Deep Inspection section below provides details about this parameter.
7. `c.unique_identifier` is not required. Unique identifier allows you to set data on a alert or alerts to make the signature more unique through data. This works in conjunction with deep inspection, and is explained more in the unique identifier section below.

### `perform` Function

Every custom signature requires a `perform` function. This is the entry point for the signature engine to call and pass in the live aws object. Please see the section below on the AWS object.

### `aws` Object in `perform`

This is a live AWS object that is a actually a ruby object passed into the javascript DSL during runtime. This object supports all the services ESP support as methods. These services are live API clients to AWS. Please see the section below for a list of each currently supported services.

### Services on the `aws` object

|Service|Method |
|:------|:------|
|Route53|route53|
|Glacier|glacier|
|ElasticBeanstalk|elbs|
|EC2|ec2|
|IAM|iam|
|SQS|sqs|
|SES|ses|
|RDS|rds|
|ElasticTranscoder|elt|
|ElasticLoadBalancing|elb|
|SimpleDB|sdb|
|CloudFormation|cfm|
|OpsWorks|ops|
|SNS|sns|
|STS|sts|
|EMR|emr|
|S3|s3|
|CloudSearch|cs|
|CloudWatch|cw|
|CloudFront|cf|
|CloudTrail|ct|
|StorageGateway|sg|
|AutoScaling|rs|
|Kinesis|ks|
|ElastiCache|ec|
|Route53Domains|route53_domains|
|CognitoSync|cognito_sync|
|CognitoIdentity|cognito|
|Lambda|lambda|
|ConfigService|config|
|CloudWatch|cw|
|CloudWatchLogs|cwl|
|CodeDeploy|cd|
|DirectConnect|dc|
|ImportExport|ie|
|DataPipeline|dp|
|ElastiCache|ec|

#### Example
```
aws.ec2()
```

All of these objects map to the ruby AWS SDK client objects. You should be able to do any of the actions that are defined in the documentation there. 

Please note that most arguments should be passed in as hashes. For example: For S3 to get an object it requires bucket: and key: as in `aws.s3().get_object({bucket: ‘bucket_name’, key: ‘bucket_key’})`. These arguments are converted into ruby objects in the Custom Signature engine.

### DSL Object

The `dsl` object in custom signatures is another ruby object passed into the JavaScript during runtime. This object gives access to setting deep inspection and unique identifier data as well as returning an alert of the signature. Currently supported methods are below.

* `dsl.set_data()` Used to set deep inspection and unique identifier data. _Please note_ that you must pass in either a hash of that data or the object required to take the data from. So if you set `:username` in deep inspection or unique identifier the object you pass to set data must respond to `:username` as a method or value in some way. This will add the data automatically to any alert you create. So if you are creating alerts within a loop you must call this method before each alert creation and it will add the data to each alert.
* `dsl.pass()` This will create a passing alert. Takes an optional hash for extra information in the alert. Example `dsl.pass({message: ‘MFA on root yay!’})`. _Please note_ you can pass whatever key value pairs you want to show up in the alert.
* `dsl.fail()` This will create a failing alert. Takes an optional hash for extra information in the alert. Example `dsl.fail({message: ‘No MFA on root :(’}).` _Please note_ you can pass whatever key value pairs you want to show up in the alert.
* `dsl.warn()` This will create a warning alert. Takes an optional hash for extra information in the alert. Example `dsl.warn({message: ‘Some warning’})`. _Please note_ you can pass whatever key value pairs you want to show up in the alert.
* `dsl.error()` This will create a error alert. Takes an optional hash for extra information in the alert. Example `dsl.error({error: ‘something bad happened’})`. _Please note_ you can pass whatever key value pairs you want to show up in the alert.

### Deep Inspection

Deep inspection data can be added to any alert by calling `dsl.set_data()` before each alert. The values defined in the configuration is what is pulled out of the objects that are passed into the `set_data` method call. Example `c.deep_inspection   = [‘users']`

Setting this information for your alert: `dsl.set_data(aws.iam.list_users());`

Notice that the object being passed into deep inspection is `list_users()`. The data returned from this object has either a field or method named users defined on it. So `deep_inspection` will pull that data from the object and add it to the alert automatically. Please note that this will be reset between reports.


Example for live `deep_inspection` response on an alert:

```
"deep_inspection" => [
  [0] {
    "users" => [
               
    [0] {
      "path" => "/",
      "user_name" => "joebazooka",
      "user_id" => "ABCDEF0123456789ABCDE",
      "arn" => "arn:aws:iam::012345678901:user/joebazooka",
      "create_date" => "2014-09-23T16:22:48.000Z",
      "password_last_used" => "2014-10-20T19:33:05.000Z"
    }
```
