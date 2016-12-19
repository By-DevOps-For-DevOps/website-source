### CI/DC pipline for ECS with AWS Developer tools
We can use the AWS developer tools to Automate the app deployment to ECS clustor. This can help us to achieve a serveless architecture 
compared to the usual Jenkins automations.

### The Components that we use.
1.AWS ECS Cluster 
2.AWS CodeBuild
3.AWS Lambda
4.AWS CodePipeline.
5.AWS ECR

### AWS ECS Cluster
We can use the cloudformation template https://github.com/microservices-today/IaC-ecs to create an ECS cluster.
### AWS ECR
The docker images that are deployed to ECS Cluster is stored in the ECR repository. We need to create a repository for 
each of the custom application that we are hosting on ECS. You can follow the steps to create a new Repository for `nginx` 
[Steps](https://ap-southeast-1.console.aws.amazon.com/ecs/home?region=ap-southeast-1#/repositories/create/new).
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


