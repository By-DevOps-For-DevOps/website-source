---
title: AWS CodeBuild
date: 2016-12-21 14:45:38
tags: AWS Developer tools
---
#### The following steps can be used to prepare the Github to be added to the codebuild.

1. The codebuild looks for `buildspec.yml` in root of the repository. It contains the actions that are to be performed by the   Codebuild. We can add `buildspec.yml` to the github repository.
  ```
version: 0.1

phases:
  build:
    commands:
      - git ls-remote https://${GITHUB_TOKEN}@github.com/microservices-today/nginx-docker.git HEAD | awk '{ print $1;}' > master.commit
      - docker build -t $ECR_REPO:$(cat ./master.commit) .
      - sed -i "s@ECS_CLUSTER_NAME@${ECS_CLUSTER_NAME}@g" ecs/service.yaml
      - sed -i "s@TAG@$(cat ./master.commit)@g" ecs/service.yaml
      - sed -i "s@DOCKER_IMAGE_URI@$ECR_REPO:$(cat ./master.commit)@g" ecs/service.yaml
      
  post_build:
    commands:
      - docker images
      - export AWS_DEFAULT_REGION=$AWS_REGION
      - $(aws ecr get-login --region $AWS_REGION)
      - echo $ECR_REPO:$(cat ./master.commit)
      - docker push $ECR_REPO:$(cat ./master.commit)

artifacts:
  files:
    - task-definition.json
    - service-definition.json
    - ecs/service.yaml
    - master.commit
  ```
  Replace the `https://github.com/***********.git` from the buildspec.yml with current github URL.

2. Add the `ecs/service.yaml`

```
AWSTemplateFormatVersion: '2010-09-09'
Parameters:
  Tag:
    Type: String
    Description: Tag of the Docker Image.
    Default: TAG
  ECSClusterName:
    Type: String
    Description: Name of an existing ECS Cluster.
    Default: ECS_CLUSTER_NAME

Resources:
  Nginxtaskdefinition:
    Type: AWS::ECS::TaskDefinition
    Properties:
      Family: Nginx-ng
      ContainerDefinitions:
      - Name: nginx
        Cpu: '10'
        Essential: 'true'
        Image:
          "Fn::Sub":
            - '${AccountId}.dkr.ecr.${Region}.amazonaws.com/nginx-docker:TAG'
            - { AccountId: AWS::AccountId, Region: AWS::Region }
        Memory: '128'
        PortMappings:
        - ContainerPort: 80
          HostPort: 80
      Volumes:
      - Name: my-vol
  NginxService:
    Type: AWS::ECS::Service
    Properties:
      Cluster: ECS_CLUSTER_NAME
      DesiredCount: '1'
      TaskDefinition:
        Ref: Nginxtaskdefinition
Outputs:
  ecsservice:
    Value:
      Ref: NginxService
```
