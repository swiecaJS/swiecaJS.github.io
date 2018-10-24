---
layout: post
title:  "DevOps Kitchen #1 - 24/7 online services on AWS EC2 spot instances "
date:   2018-10-24 21:18:22 +0200
categories: aws meetups ec2
---
## Facts
* pricing
  * spot instance price is cheaper by 50-80% On-Demand price
  * reserved instance is 28-64% cheaper 
* How AWS will notify you that your instance will be shut down?
  * pooling type
    * instance metadata (/spot/termination-time)
    * cli ```describe-spot-instance request```
  * event driven (!)
    * Cloudwatch Events -> instance action
* Spot instances market changed:
  * now the main reason for termination is ```instance-terminated-instance-terminated-```
  * price chart is more stable, but a bit higher (since April 2018)
  * billing type is **one second** - so no more first free hour if AWS shut down your instance


## How codewise operates
* On top of that is JVM service Spotty - they have a plan to release it as an open-source project.
* General idea is to have 2 - ASGs. One with On-Demands, another with spot instances, based on the same launch configuration.
  * if there is a need for a new instance it is launched as a spot instance 
* Persistence layer - **AWS Tags**
  
## Their main issues

#### AZ imbalance
* When 1AZ of spots is killed by AWS, AutoScalling receives task to create the same amount of On-Demand instances.
  * By design AutoScalling will try to evenly distribute those instances
  * Codewise has a lot of traffic, and their systems want to use internal AZ network as much as possible for cost/latency reduction
* Solution:
  * Client-side load balancing

#### AWS Limits
* They provision a lot of instances and often hit the max number of resources limit.

#### Interruption awareness
* They use polling meta-data for termination time - so it must be implemented in the AMI as a background service.
  
#### Scaling
* Scaling policies are copied to spot groups




---
*resources*
* [Meetup presentation - by Maciej Biela](https://drive.google.com/file/d/16NfCv6L_JevsHlr60P6gAQ0VhNiaYUDb/view)
* [Codewise open-source](https://github.com/codewise-oss/canaveral)
