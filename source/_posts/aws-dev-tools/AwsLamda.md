---
title: AWS Lamda for Task definition updation
date: 2016-12-21 14:45:38
tags: AWS Developer tools
---
#### Follow these steps to create two lamda functions.

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
