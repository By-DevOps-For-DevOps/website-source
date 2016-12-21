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
each of the custom application that we are hosting on ECS. You can follow these [steps](./ECR.md) to create a new ECR Repository.
### AWS CodeBuild
We build the docker image and push to the ECR with the latest tag using AWS Codebuild. You can follow [these](./AwsCodebuild.md) steps to create CodeBuild

### AWS Lambda

The AWS Lambda can be used as a part of our pipeline for updating task and service definition. You can follow [these](./Awslamda.md)

1. Update Task Definition 

The sample code below can be used to update the service definition.
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
        console.log(params)
        AWS.config.update({region: 'us-east-1'});
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
        AWS.config.update({region: 'us-east-1'});
        var params = {
            jobId: jobId,
            failureDetails: {
                message: JSON.stringify(message),
                type: 'JobFailed',
                externalExecutionId: context.invokeid
            }
        };
        AWS.config.update({region: 'us-east-1'});
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
        AWS.config.update({region: 'us-east-1'});
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
        AWS.config.update({region: 'us-east-1'});
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
                    var params = {
                        desiredCount: 2, 
                        service: serviceDefinition.serviceName,
                        cluster: serviceDefinition.cluster,
                    };
                    ecs.updateService(params, function(err, data) {
                        if (err) {
                            console.log(err);
                            console.log(params);
                            ecs.createService(serviceDefinition, function(err, data) {
                                if (err) putJobFailure(err);
                                else putJobSuccess("Successfully launched the service");
                            });
                            
                        } else putJobSuccess("Successfully launched the service");
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
