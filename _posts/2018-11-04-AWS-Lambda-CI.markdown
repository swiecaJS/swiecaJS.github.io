---
layout: post
title: "How I implemented Continous Delivery for AWS Lambda without a framework"
date: 2018-11-04 09:30:22 +0200
categories: AWS Serverless CD
---

### What AWS Lambda is - a short introduction

You can think of Lambda function as a just piece of code in specified runtime (e.g. Node.js 8.10) that perform some action (e.g HTTP request / AWS API call) when triggered.

![AWS Lambda flow](/assets/img/2018-11-04/1.JPG)

Moreover, Lambda on its own does not have access to any of your resources. But when invoked Lambda assumes the role with appropriate permissions.

![AWS Lambda flow](/assets/img/2018-11-04/2.JPG)

I definitely will write a more detailed post about what goes under the hood the AWS Lambda, but this definition should be just found to understand problems that we had.

### Overview of how we use it.

In my workplace, AWS Lambdas are mainly used for operational and monitoring purposes. We use them to clean up unused resources, perform backups, cross-account migrations and automation. In addition, a great majority of our Java services are publishing notifications to SNS topics. We have Lambda functions which are subscribed to those topics and there send us messages. In addition, some third-party services are controlled with serverless functions.

### What problems we had?

* Bad naming
* Roles with massive permissions
* No history of changes (Inline code edition vs GIT)
* Lack of CD pipeline

The first two problems were fixed by performing an IAM audit an creating proper guidelines. Actually, it can be complicated - when one role with a wide range of permissions is used by multiple lambdas - and you suspect that not all of them need it. 

As you remember Lambda when invoked assumes IAM Role - and this is the only thing that you will see in CloudTrail. Without having code locally and just ```Find in files``` AWS API call that you need it can be challenging.

Also, there is a question - **do we still need this IAM role**?
Thankfully AWS has a tool [Acess Advisor](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_access-advisor.html). You can peek when policy inside IAM role was used - and based on that make a decision. The sad part about it is at the moment of writing this post - you cannot access it by API. Only on AWS console. I hope it will change in future. But now if you are tired of doing it by yourself, you can use tools provided by [Netflix](https://medium.com/netflix-techblog/introducing-aardvark-and-repokid-53b081bf3a7e). They have created a tool that scrapes AWS console and deletes unaccessed roles.

To handle remaining problems there was a need for a pipeline and tool for local development.

### Options of tackling this problem

**AWS SAM** (Serverless Application Model) - an open-source framework based on CloudFormation that gives you CLI for handling Lambda operations - creating/updating/invoking-locally

**ServerLess** - another great framework - widely adopted in the industry. It can work with other cloud providers also. The only thing that bothers me is their [AWS credential setup video](https://www.youtube.com/watch?v=HSd9uYj2LJA). They instruct you to create **Full Administrator Access** for your developer machine. This is clearly a bad practice. Actually, they need access only to Lambda, IAM and CloudFormation.

Ok from those two - for our AWS SAM seems a better solution. But it turned out that I was told - that implementing SAM is *to much* for our purposes and if I want to implement pipeline I should use something *easier*.

## AWS CLI + Jenkins

I came up with below flow:
![AWS Lambda flow](/assets/img/2018-11-04/3.JPG)

Because our Java services had CI/CD on Jenkins I could use it to create a task 'Deploy to lambda'. But let's have a look of it step-by-step. 

### Local Development
It is worth noticing that we use Node.js. I chose to use a **jest** test runner for 'invoking' lambda locally. I have created a shell script that will create a propper directory at the beginning with some basic test 

![Directory tree](/assets/img/2018-11-04/tree.JPG)

Lambda has ```handler``` exported by default


```javascript
exports.handler = (event, context, callback) => {
 // TODO implement

 // example action
 var msg = event.Records[0].Sns.Message;
 const response = {
 statusCode: 200,
 body: msg.toUpperCase()
 };
 return response;
};
```

And we can ```import``` it in a unit test.

```javascript
const lambda = require('../index.js')
describe('Config is propper for deployment', () => {
 test('lambda exists', () => {
 expect(lambda).toBeTruthy();
 })

 test('index.handler is exposed properly', () => {
 expect(lambda.handler).toBeTruthy();
 })
})
```
I also added a sample SNS event to and prepare a basic test as an example

```javascript
const sampleSNSEvent = {
 "Records": [{
 "EventVersion": "1.0",
 "EventSubscriptionArn": "eventsubscriptionarn",
 "EventSource": "aws:sns",
 "Sns": {
 "SignatureVersion": "1",
 "Timestamp": "1970-01-01T00:00:00.000Z",
 "Signature": "EXAMPLE",
 "SigningCertUrl": "EXAMPLE",
 "MessageId": "95df01b4-ee98-5cb9-9903-4c221d41eb5e",
 "Message": "Hello from SNS!",
 "MessageAttributes": {
 "Test": {
 "Type": "String",
 "Value": "TestString"
 },
 "TestBinary": {
 "Type": "Binary",
 "Value": "TestBinary"
 }
 },
 "Type": "Notification",
 "UnsubscribeUrl": "EXAMPLE",
 "TopicArn": "topicarn",
 "Subject": "TestInvoke"
 }
 }]
}

describe('Lambda logic is propper', () => {

 const response = lambda.handler(sampleSNSEvent, null, null)
 const expetctedMessageBody = "HELLO FROM SNS!"

 test('handler returns response', () => {
 expect(response).toBeTruthy
 })

 test('messagebody is uppercase', () => {
 expect(response.body).toBe(expetctedMessageBody)
 })

 test('response status code is 200', () => {
 expect(response.statusCode).toBe(200);
 })

})
```

Now developer can ran ```npm test -t lambda-name``` to test it locally.

### Deployment

It's a simple process.

* ```zip -r``` - Create a zip file
* ```aws s3 cp``` - upload to s3
* ```aws lambda update-function-code``` - update function code

And that's it. You just need to provide to Jenkins ```your-lambda-name``` and those names should match.
I know this process does not include creating new lambdas, but my main goal was to *force* every developer to commit code change to the repository. To make sure that everybody will follow the rules - i removed ```update-lambda-function``` policy from developers IAM group :)

### Summary

Now we have a way for local development and deploying. Code changes will be visible in GIT repository and now we can use ```node_modules``` ! In the future, I would like to improve this process by migrating to AWS SAM, and instaling node on Jenkins to perform ```npm install && npm test``` - because at this moment we will have to keep node_modules folder in a repository (that's because AWS Lambda needs a packed zip file with all modules).

Anyway, it was a pleasure to implement it!

---
*resources*

* [Serverless framework](https://docs.aws.amazon.com/lambda/latest/dg/serverless_app.html)
* [AWS Lambda CLI](https://docs.aws.amazon.com/cli/latest/reference/lambda/)
* [Netflix way of handling unused IAM Roles](https://medium.com/netflix-techblog/introducing-aardvark-and-repokid-53b081bf3a7e).