---
title: DCOS Disaster Recovery
date: 2016-11-03 17:07:05
tags: DCOS Disaster Recovery
---

### Marathon App Recovery

#### Taking Marathon Snapshots

[Marathon snapshot](https://github.com/microservices-today/IaC-marathon-snapshots) IaC is used to take snapshots of marathon applications. These snapshots are stored in a S3 bucket (Bucket name is specified while running NGP). This is performed by running a Lambda function which is periodically triggered by a CloudWatch metric. This app takes snapshots of marathon applications every hour. The IaC, also creates an AWS API gateway to trigger the Lambda function in order to restore the marathon snapshot.
