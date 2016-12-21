---
title: CI/CD pipeline for ECS with AWS Developer tools
date: 2016-12-21 14:45:38
tags: AWS Developer tools
---
### CI/CD pipeline for ECS with AWS Developer tools
We can use the AWS developer tools to Automate the app deployment to ECS clustor. This can help us to achieve a serveless architecture 
compared to the usual Jenkins automations.

### The Components that we use.

1. AWS ECS Cluster 
2. AWS CodeBuild
3. AWS Lambda
4. AWS CodePipeline.
5. AWS ECR

### AWS ECS Cluster
We can use the cloudformation template https://github.com/microservices-today/IaC-ecs to create an ECS cluster.
### AWS ECR
The docker images to be deployed to ECS Cluster are stored in the ECR repository. We need to create a repository for 
each of the custom application that we are hosting on ECS. You can follow these [steps](../ECR) to create a new ECR Repository.
### AWS CodeBuild
We build the docker image and push to the ECR with the latest tag using AWS Codebuild. You can follow [these](../AwsCodebuild) steps to create CodeBuild

### AWS Lambda

The AWS Lambda can be used as a part of our pipeline for updating task and service definition. You can follow [these](../Awslamda)

### AWS CodePipeline
The components are added to the CodePipeline in to complete the CI/CD pipeline. You can follow [these](../AwsCodepipeline) steps to create Codepipeline.
