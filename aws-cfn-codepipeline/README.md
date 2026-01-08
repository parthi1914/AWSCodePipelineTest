# Ready-to-use CloudFormation (Nested) + CodePipeline Buildspec

This repo contains minimal, maintainable CloudFormation nested stacks for:
- VPC (2 AZ public/private) + NAT
- Security Groups (ALB + ECS)
- ALB + Target Group + HTTP Listener
- ECS Fargate Cluster + Service behind ALB (default image nginx)
- S3 artifacts bucket (private)
- Lambda function (code pulled from S3)

## What you must do before deploy
1) Zip `lambda_src/app.py` as `hello.zip`
2) Upload to S3 path:
   s3://<myapp-ENV-artifacts-ACCOUNT-REGION>/lambda/hello.zip

## CodeBuild
Set env variable in CodeBuild:
- CFN_PACKAGE_BUCKET = <your packaging bucket>


##Env Setup
The Build stage outputs `packaged-root.yml` for CloudFormation deploy action.
