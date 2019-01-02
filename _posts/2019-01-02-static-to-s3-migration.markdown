---
layout: post
title: "Static website migration to S3 + Lambda"
date: 2019-01-02 15:38:22 +0200
categories: aws serverless
---

Recently in my work we had a task to migrate website from classical Linux, Apache, PHP stack to something which won't require 'servers'. By severs in this case I mean EC2 instances. The main reason for doing that was to speed up deployment process.

### The old setup
Pages templates are made in jade. Jade is template language which evoled to PUG now. there is build task in gulp which generates pure html. Each language has different folder. The structure inside each folder is the same. 

![](/assets/img/to-s3-migration/1.png)

So if we put those folders on S3, enable static web hosting will it work out of the box?

![](/assets/img/to-s3-migration/2.png)

In this particular case the answer is no. The reason for that is PHP job handled on the server side. By that I mean:
  * handling contact forms
  * localization (l10n) & internalization (i18n)
  * handling routing & redirections

In this blog post we skip handling contact forms (just for your curiosity the setup for them is API Gateway + Lamda).

### Architecture overview

![](/assets/img/to-s3-migration/3.png)

Because every request will pass through CloudFront we can use Lambda@Egde to handle routing and personalize content by country. 


### First iteration

Cloudfront distribution provides 4 hooks where we can trigger lambda function in each request. In this case we started with viewer request.

![](/assets/img/to-s3-migration/5.png)

It seemed as a reasonable choice beacuse:
* We have **uri** so we can properlu redirect
* **accept-language** header can be **whitlisted** and it will be availible to serve proper content
* According to URI, we can **getObject** from S3

Looks pretty simple,  **what could possibly go wrong**? 
I fired test enviroment, went to the cloudfront url to see this
![](/assets/img/to-s3-migration/error1.png)

Cloudfront logs showed this error.

![](/assets/img/to-s3-migration/error2.png)

The problem with this thinking was fact, that that request is **before** cloudfront caches content. Because of that, there is a limit of thebody size (~40kb). 

### Second iteration

I have used two lambdas on two triggers.

![](/assets/img/to-s3-migration/viewer.png)

* We have uri, we can properly redirect (302)
* We have **accept-language** headers, therefore we can serve proper content

![](/assets/img/to-s3-migration/origin.png)

* **getObject** from S3

This worked as expected. 

### Third iteration

The next task is to make this code reusable between different enviroments (CloudFront + S3 buckets). One reusable set of two lambdas. So in this config we have two variables:
* domain connected to CloudFront
* S3 bucket with assets

We would like to differ between buckets based on the associated domain. How to do it?

![](/assets/img/to-s3-migration/viewer.png)

* We can serve propper content based on **host** header
  * e.g host: test12.example.com -> S3 bucket called test12-example-com

![](/assets/img/to-s3-migration/origin.png)

* the **host** header in origin request is equal to **bucket-name**.s3.amazonaws.com
  * to avoid some dirty string operations we need better solition

#### Custom header to the rescue!

![](/assets/img/to-s3-migration/host.png)

Thanks to that every request will have this custom header and we can differ between buckets without dirty string hacks. My last advice will be:

#### Don't forget about proper permissions

At the beggining of work I have lost a lot of time fighting with this view.

![](/assets/img/to-s3-migration/503.png)

I was stuck because my IAM role had propper permission specified in docs.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "lambda:GetFunction",
                "lambda:EnableReplication*",
                "iam:CreateServiceLinkedRole"
            ],
            "Resource": "*"
        }
    ]
}
```

The last piece beffore happy end is to create a **trust relationship**. Because lambda functions connected to Cloudfront are actually called Lambda@Edge. Apart from cool name they have different let's say basic enviroment.

![](/assets/img/to-s3-migration/iam1.png)

You need to add **edgelambda.amazonaws.com** there. So in the end your trust relationshipt should looks like this:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    },
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "edgelambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```


---
*resources*
* [StackOverflow disscusion](https://stackoverflow.com/questions/31354559/using-node-js-require-vs-es6-import-export)
* [ESM & commonJS module differences](http://voidcanvas.com/import-vs-require/)










