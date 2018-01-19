# Datascience Autovision API

The datascience autovision API runs in a docker container and only needs access to s3.  As such, the build process will consist of building a docker image and pushing it to an ECR repository.  Presently, there is a repository named "datascience-autovision" in us-east-1.  It's URI is 082894742960.dkr.ecr.us-east-1.amazonaws.com/datascience-autovision

Deploying consists of creating a new task definition that references the new image, then deploying the new task.

A separate family of task definitions should be created for each environment.

Task definitions are registered using the aws cli via the following command:

`aws ecs register-task-definition --cli-input-json "file://./autovision-task.json"`

the contents of "autovision-task.json" should look like the following

```
{
  "containerDefinitions": [
    {
      "name": "autovision-container",
      "image": "082894742960.dkr.ecr.us-east-1.amazonaws.com/datascience-autovision:latest",
      "portMappings": [
        {
          "protocol": "tcp",
          "containerPort": 80
        }
      ],
      "essential": true,
      "environment": [
        {
          "name": "CAR_PRESENCE_PREDICTION_MODEL",
          "value" : "./models/car_presence_model.pb"
        },
        {
          "name": "VIEW_PREDICTION_MODEL",
          "value" : "./models/view_prediction_model.pb"
        },
        {
          "name": "CAR_PRESENCE_THRESHOLD",
          "value" : "0.6"
        },
        {
          "name": "VIEW_PREDICTION_THRESHOLD",
          "value" : "0.6"
        },
        {
          "name": "AWS_ACCESS_KEY_ID",
          "value": "<actual aws access key id>"
        },
        {
          "name": "AWS_SECRET_ACCESS_KEY",
          "value": "<actual aws secret access key>"
        }
      ],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "<env>-autovision",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "<env>-autovision"
        }
      }
    }
  ],
  "networkMode": "awsvpc",
  "family": "<env>-autovision",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "<CPU spec>",
  "memory": "<Memory spec>",
  "executionRoleArn": "<execution role ARN>",
  "taskRoleArn": "<task role ARN>"
}

```
