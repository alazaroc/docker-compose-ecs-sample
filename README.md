# Docker Compose ECS Sample

This repository has been forked from this one: <https://github.com/anshrma/docker-compose-ecs-sample>
> The difference is that in this repository we will use Amazon Elastic Container Registry (ECR) instead of Docker Hub to store the images.

The original AWS workshop with all the steps is here: <https://docker.awsworkshop.io/21_build_images/10_sample_application_overview.html>

## Main commands

### Run `docker compose` locally

Build image locally:

``` code
docker compose build
```

Show images built:

``` code
docker images
```

Run locally

``` code
docker compose up -d
```

Access to the application

``` code
curl http://localhost:3000
curl http://localhost:3000/add/2/name2
curl http://localhost:3000/add/3/name3
curl http://localhost:3000/add/4/name4
```

Retrieve logs:

``` code
docker compose logs
```

List volumes created:

``` code
docker volume ls
```

Inspect volumes:

``` code
docker volume inspect docker-compose-ecs-sample_db-data
```

List the networks created:

``` code
docker network ls
```

Stop the application:

``` code
docker compose down
```

### Run `docker compose` command to deploy on Amazon ECS

Create ECR repositories:

``` code
aws ecr create-repository --repository-name example-app-backend --image-scanning-configuration scanOnPush=true --region {REGION}
aws ecr create-repository --repository-name example-app-frontend --image-scanning-configuration scanOnPush=true --region {REGION}
```

Login to ECR:

``` code
aws ecr get-login-password --region {REGION}| docker login --username AWS --password-stdin {ACCOUNT_ID}.dkr.ecr.{REGION}.amazonaws.com
```

Push images to ECR:

``` code
docker tag ${IMAGE_ID_FROM_DOCKER_IMAGES} ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/example-app-backend:latest
docker push ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/example-app-backend:latest

docker tag ${IMAGE_ID_FROM_DOCKER_IMAGES} ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/example-app-frontend:latest
docker push ${ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/example-app-frontend:latest
```

Create a new ECS context and use it:

``` code
docker context create ecs my-ecs-context-name
docker context use my-ecs-context-name
```

Deploy docker compose application on ECS:

``` code
ACCOUNT_ID=${xxxxx} REGION=${xxxxx} docker compose -f docker-compose.yaml -f  docker-compose.prod.migrate.yaml up
```

### Create CI/CD pipeline

You have to create first the following Secret in the Amazon Secrets Manager service. Here you have how to create the GitHub OAuth token: <https://docker.awsworkshop.io/41_codepipeline/10_setup_secretsmanager_github.html>

Create `github-oauth-token` in SecretsManager:

``` code
aws secretsmanager create-secret --name github-oauth-token --description "Secret for GitHub" --secret-string "{"GITHUB_ACCESS_TOKEN":"insert your GitHub OAuth token"}"
```

Create a CloudFormation stack to automatize the deployment of the solution each time you commit code to your GitHub repository using Amazon `CodePipeline` and Amazon `CodeBuild`:

``` code
aws cloudformation create-stack --stack-name docker-compose-code-pipeline --template-body file://operations/code-pipeline-cloudformation.yaml --capabilities CAPABILITY_NAMED_IAM --enable-termination-protection --region {REGION}
```
