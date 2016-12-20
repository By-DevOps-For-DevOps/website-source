### CI/DC pipline for ECS with AWS Developer tools
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
each of the custom application that we are hosting on ECS. You can follow the steps to create a new ECR Repository.
### AWS CodeBuild
We build the docker image and push to the ECR with the latest tag using AWS Codebuild. The `buildspec.yml` guides the 
CodeBuild on the actions that it has to perform. A sample spec file is shown below.
```
version: 0.1

phases:
  build:
    commands:
      - docker build -t $ECR_REPO:latest .
      - sed -i "s@DOCKER_URI@${ECR_REPO}:latest@g" task-definition.json
      - sed -i "s@ECS_CLUSTER_NAME@${ECS_CLUSTER_NAME}@g" service-definition.json
      - sed -i "s@LOADBALANCER_NAME@${LOADBALANCER_NAME}@g" service-definition.json
  post_build:
    commands:
      - $(aws ecr get-login --region $AWS_REGION)
      - docker push $ECR_REPO:latest
artifacts:
  files:
    - task-definition.json
    - service-definition.json
```
The variables in the commands can be replaced by using Environment Variables in the CodeBuild.

##### Task Definition

```
{
    "family": "nginx-prototype",
    "containerDefinitions": [
        {
            "environment": [],
            "name": "nginx",
            "image": "DOCKER_URI",
            "cpu": 10,
            "memory": 500,
            "portMappings": [
                {
                    "containerPort": 80,
                    "hostPort": 80
                }
            ]
        }
    ]
}
```

##### Service Definition

```
{
  "serviceName": "nginx-prototype",
  "taskDefinition": "nginx-prototype",
  "cluster": "ECS_CLUSTER_NAME",
  "desiredCount":2,
  "loadBalancers": [
     {
    "containerName": "nginx-prototype", 
    "containerPort": 80, 
    "loadBalancerName": "LOADBALANCER_NAME"
   }
  ],
  "role": "ECS_SERVICE_ROLE",
}
```
The Variable mapping are as shown below.

- ECR_REPO : ECR repo URI.
- ECS_CLUSTER_NAME : ECS Cluster name.
- LOADBALANCER_NAME : The loadbalancer name for the application. 
- ECS_SERVICE_ROLE : The role should be attached to the loadbalancer.

The Output of the CodeBuild will be as given below: 

1. Build Docker images
2. Push AWS ECR
3. Modify Task Definition.
4. Modify Service Definition.

### AWS Lambda

The AWS Lambda can be used as a part of our pipeline for updating task and service definition.

1. Update Task Definition 

The sample code below can be used to update the service definition.
```
'use strict';
exports.handler = (event, context, callback) => {
    //Configuring AWS 
    var AWS = require('aws-sdk');
    
    var fs = require('fs');
    var ecs = new AWS.ECS();
    var s3 = new AWS.S3({
        maxRetries: 10,
        signatureVersion: "v4"
    });
    var codepipeline = new AWS.CodePipeline();
    //Child process
    const exec = require('child_process').exec;
    //expecting params from input artifacts
    var params = {
        Bucket: event['CodePipeline.job'].data.inputArtifacts[0].location.s3Location.bucketName,
        Key: event['CodePipeline.job'].data.inputArtifacts[0].location.s3Location.objectKey
    };
    //Getting pipeline ID
    var jobId = event["CodePipeline.job"].id;
    const unzipCommand = "rm -rf /tmp/artifacts && mkdir -p /tmp/artifacts && unzip /tmp/artifact.zip -d /tmp/artifacts"
    // Notify AWS CodePipeline of a successful job
    AWS.config.update({region: 'ap-southeast-1'});
    var putJobSuccess = function(message) {
        console.log("Success" + message);
        var params = {
            jobId: jobId
        };
        console.log(params)
        codepipeline.putJobSuccessResult(params, function(err, data) {
            if(err) {
                console.log("Unable to update pipeline" + err);
                context.fail(err);      
            } else {
                context.succeed(message);      
            }
        });
    };
    
    // Notify AWS CodePipeline of a failed job
    var putJobFailure = function(message) {
        console.log("Failure" + message);
        var params = {
            jobId: jobId,
            failureDetails: {
                message: JSON.stringify(message),
                type: 'JobFailed',
                externalExecutionId: context.invokeid
            }
        };
        codepipeline.putJobFailureResult(params, function(err, data) {
            context.fail(message);      
        });
    };
    //We can start with pulling the artifacts
    s3.getObject(params, function(err, data) {
    if (err) putJobFailure(err);
    else {
        //writing the artifacts to the /tmp/ , we have access to only this directory
        fs.writeFile("/tmp/artifact.zip", data.Body, function(err) {
            if (err) putJobFailure(err);
            else {
                const child = exec(unzipCommand, (error) => {
                    var taskDefinition = require('/tmp/artifacts/task-definition.json');
                    var serviceDefinition = require('/tmp/artifacts/service-definition.json');
                    ecs.registerTaskDefinition(taskDefinition, function(err, data) {
                        if (err) putJobFailure(err);
                        else putJobSuccess("Successfully created task defnition");
                    });
                });
                //print the output of child process
                child.stdout.on('data', console.log);
                child.stderr.on('data', console.error);
                callback(null, 'Process complete!');
            }
        });
        }

    });
};
```
This lamda function will create a new task definition or a new revision of the existing task definition.

2. Update Service Definition

```
'use strict';
exports.handler = (event, context, callback) => {
    //Configuring AWS 
    var AWS = require('aws-sdk');
    
    var fs = require('fs');
    
    var codepipeline = new AWS.CodePipeline();
    //Child process
    const exec = require('child_process').exec;
    //expecting params from input artifacts
    var params = {
        Bucket: event['CodePipeline.job'].data.inputArtifacts[0].location.s3Location.bucketName,
        Key: event['CodePipeline.job'].data.inputArtifacts[0].location.s3Location.objectKey
    };
    //Getting pipeline ID
    var jobId = event["CodePipeline.job"].id;
    var codePipelineParams = {
        jobId: jobId
    };
    AWS.config.update({region: 'ap-southeast-1'});
    var ecs = new AWS.ECS();
    var s3 = new AWS.S3({
        maxRetries: 10,
        signatureVersion: "v4"
    });
    const unzipCommand = "rm -rf /tmp/artifacts && mkdir -p /tmp/artifacts && unzip /tmp/artifact.zip -d /tmp/artifacts"
    // Notify AWS CodePipeline of a successful job
    var putJobSuccess = function(message) {
        console.log("Success" + message);
        var params = {
            jobId: jobId
        };
        codepipeline.putJobSuccessResult(params, function(err, data) {
            if(err) {
                context.fail(err);      
            } else {
                context.succeed(message);      
            }
        });
    };
    
    // Notify AWS CodePipeline of a failed job
    var putJobFailure = function(message) {
        console.log("Failure" + message);
        var params = {
            jobId: jobId,
            failureDetails: {
                message: JSON.stringify(message),
                type: 'JobFailed',
                externalExecutionId: context.invokeid
            }
        };
        codepipeline.putJobFailureResult(params, function(err, data) {
            context.fail(message);      
        });
    };
    //We can start with pulling the artifacts
    s3.getObject(params, function(err, data) {
    if (err) putJobFailure(err);
    else {
        //writing the artifacts to the /tmp/ , we have access to only this directory
        fs.writeFile("/tmp/artifact.zip", data.Body, function(err) {
            if (err) putJobFailure(err);
            else {
                const child = exec(unzipCommand, (error) => {
                    var taskDefinition = require('/tmp/artifacts/task-definition.json');
                    var serviceDefinition = require('/tmp/artifacts/service-definition.json');
                    console.log(serviceDefinition);
                    ecs.createService(serviceDefinition, function(err, data) {
                        if (err) putJobFailure(err);
                        else putJobSuccess("Successfully launched the service");
                    });
                    callback(null, 'Process complete!');
                });
                child.stdout.on('data', console.log);
                child.stderr.on('data', console.error);
            }
        });
        }

    });
};

```

### AWS CodePipeline
The components are added to the CodePipeline in the order `Source>CodeBuild>Lambda` .
![image](https://blog.microservices.today/images/aws-dev-tools/code-pipeline.png)
