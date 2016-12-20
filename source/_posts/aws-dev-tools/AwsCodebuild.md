#### The following steps can be used to create a Codebuild which can build a docker image and push to ECR.

1. The codebuild looks for `buildspec.yml` in root of the repository. It contains the actions that are to be performed by the Codebuild. We can add `buildspec.yml` to the github repository.
```
version: 0.1

phases:
  build:
    commands:
      - export GITHUB_TOKEN=${GITHUB_TOKEN} && git ls-remote https://github.com/microservices-today/nginx-docker.git HEAD | awk '{ print $1;}' > master.commit
      - docker build -t $ECR_REPO:$(cat ./master.commit) .
      - sed -i "s@DOCKER_URI@${ECR_REPO}:$(cat ./master.commit)@g" task-definition.json
      - sed -i "s@ECS_CLUSTER_NAME@${ECS_CLUSTER_NAME}@g" service-definition.json
      - sed -i "s@TARGET_GROUP_ARN@${TARGET_GROUP_ARN}@g" service-definition.json
      - sed -i "s@ECS_SERVICE_ROLE@${ECS_SERVICE_ROLE}@g" service-definition.json
  post_build:
    commands:
      - $(aws ecr get-login --region $AWS_REGION)
      - docker push $ECR_REPO:$(cat ./master.commit)
artifacts:
  files:
    - task-definition.json
    - service-definition.json
    - master.commit
```
