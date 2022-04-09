# 作業フロー

https://aws.amazon.com/jp/getting-started/hands-on/break-monolith-app-microservices-ecs-docker-ec2/

## モノリスのコンテナ化

```sh
cd 2-containerized/services/api
aws ecr get-login-password --region [region] | docker login --username AWS --password-stdin [aws-account-id].dkr.ecr.[region].amazonaws.com
docker build -t api .
docker tag api:latest [aws-account-id].dkr.ecr.[region].amazonaws.com/api:v1
docker push [aws-account-id].dkr.ecr.[region].amazonaws.com/api:v1
```

## モノリスのデプロイ

## モノリスの分割

```sh
cd ../../../3-microservices/services
aws ecr get-login-password --region [region] | docker login --username AWS --password-stdin [aws-account-id].dkr.ecr.[region].amazonaws.com
docker build -t users ./users
docker tag users:latest [aws-account-id].dkr.ecr.[region].amazonaws.com/users:v1
docker push [aws-account-id].dkr.ecr.[region].amazonaws.com/users:v1

docker build -t threads ./threads
docker tag threads:latest [aws-account-id].dkr.ecr.[region].amazonaws.com/threads:v1
docker push [aws-account-id].dkr.ecr.[region].amazonaws.com/threads:v1

docker build -t posts ./posts
docker tag posts:latest [aws-account-id].dkr.ecr.[region].amazonaws.com/posts:v1
docker push [aws-account-id].dkr.ecr.[region].amazonaws.com/posts:v1
```

## マイクロサービスのデプロイ

### ECSでタスク定義

タスク定義は以下の通り、users/threads についても同様。

```json
{
    "requiresCompatibilities": [
        "EC2"
    ],
    "inferenceAccelerators": [],
    "containerDefinitions": [
        {
            "name": "posts",
            "image": "[aws-account-id].dkr.ecr.[region].amazonaws.com/posts:v1",
            "memoryReservation": "256",
            "cpu": "256",
            "essential": true,
            "portMappings": [
                {
                    "hostPort": "0",
                    "containerPort": "3000",
                    "protocol": "tcp"
                }
            ]
        }
    ],
    "volumes": [],
    "networkMode": "bridge",
    "placementConstraints": [],
    "family": "posts"
}
```

### CLIからターゲットグループを作成

posts, threads, users, drop-traffic について作成

```sh
aws elbv2 create-target-group --region [region] --name [service-name] --protocol HTTP --port 80 --vpc-id [vpc-attribute] --healthy-threshold-count 2 --unhealthy-threshold-count 2 --health-check-timeout-seconds 5 --health-check-interval-seconds 6
```
