# AWS CloudFormation CodePipeline Project

This project contains AWS CloudFormation templates for deploying a complete infrastructure stack with automated CI/CD pipeline using AWS CodePipeline.

## Architecture Overview

The infrastructure includes:
- **VPC** with public and private subnets across 2 AZs
- **Application Load Balancer (ALB)** in public subnets
- **ECS Fargate** cluster and service in private subnets
- **Lambda** functions
- **S3** buckets for artifacts
- **Security Groups** for network isolation
- **CodePipeline** for automated deployments

## Naming Convention

All resources follow the standardized pattern:
```
hq-<Workstream>-<Env>-<AppName>-<ResourceType>-<Purpose>
```

Examples:
- `hq-Workstream-dev-MyApp-VPC-Main`
- `hq-Workstream-dev-MyApp-ECS-Cluster`
- `hq-Workstream-dev-MyApp-Pipeline-CFNDeploy`

## Quick Start

### 1. Prerequisites
- AWS Account with appropriate permissions
- GitHub repository with Personal Access Token
- Two S3 buckets created:
  - CFN package bucket: `hq-<workstream>-<env>-<app>-s3-cfn-package`
  - Pipeline artifacts bucket: `hq-<workstream>-<env>-<app>-s3-pipeline-artifacts`

### 2. Configure Parameters

Edit parameter files in:
- `params/` - Infrastructure parameters
- `pipeline-params/` - CodePipeline parameters

### 3. Deploy CodePipeline

**Via AWS Console:**
1. Go to CloudFormation → Create Stack
2. Upload `templates/pipeline/codepipeline.yml`
3. Use parameters from `pipeline-params/dev-pipeline.json`
4. Create stack

**Via AWS CLI:**
```bash
aws cloudformation create-stack \
  --stack-name hq-Workstream-dev-MyApp-Stack-Pipeline \
  --template-body file://templates/pipeline/codepipeline.yml \
  --parameters file://pipeline-params/dev-pipeline.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1
```

### 4. Monitor Pipeline
- Go to AWS CodePipeline console
- Find: `hq-Workstream-dev-MyApp-Pipeline-CFNDeploy`
- Watch the pipeline execute: Source → Build → Deploy

## Project Structure

```
aws-cfn-codepipeline/
├── templates/
│   ├── root.yml                    # Main root stack
│   ├── pipeline/
│   │   └── codepipeline.yml        # CodePipeline infrastructure
│   ├── network/
│   │   └── vpc.yml                 # VPC and networking
│   ├── security/
│   │   └── sg.yml                  # Security groups
│   ├── s3/
│   │   └── buckets.yml             # S3 buckets
│   ├── alb/
│   │   └── alb.yml                 # Application Load Balancer
│   ├── ecs/
│   │   └── cluster-service.yml     # ECS Fargate
│   └── lambda/
│       └── functions.yml           # Lambda functions
├── params/
│   ├── dev.json                    # Development parameters
│   ├── qa.json                     # QA parameters
│   ├── uat.json                    # UAT parameters
│   └── prod.json                   # Production parameters
├── pipeline-params/
│   ├── dev-pipeline.json           # Pipeline dev parameters
│   ├── qa-pipeline.json            # Pipeline QA parameters
│   ├── uat-pipeline.json           # Pipeline UAT parameters
│   └── prod-pipeline.json          # Pipeline prod parameters
├── buildspec.yml                   # CodeBuild build specification
└── README.md                       # This file
```

## Environments

The project supports four environments:
- **dev** (Development) - VPC CIDR: 10.10.0.0/16
- **qa** (Quality Assurance) - VPC CIDR: 10.20.0.0/16
- **uat** (User Acceptance Testing) - VPC CIDR: 10.30.0.0/16
- **prod** (Production) - VPC CIDR: 10.40.0.0/16

## Pipeline Workflow

1. **Source Stage**
   - Triggered by GitHub webhook on push
   - Fetches source code from GitHub

2. **Build Stage**
   - Lints CloudFormation templates
   - Packages nested templates to S3
   - Outputs `packaged-root.yml`

3. **Deploy Stage**
   - Creates CloudFormation change set
   - Executes change set to deploy/update infrastructure

## Resources Created

### Network
- 1 VPC
- 2 Public Subnets (across 2 AZs)
- 2 Private Subnets (across 2 AZs)
- 1 Internet Gateway
- 1 NAT Gateway
- Route Tables

### Security
- ALB Security Group (allows HTTP from internet)
- ECS Security Group (allows traffic from ALB)

### Compute
- ECS Fargate Cluster
- ECS Service (1 task)
- Lambda Function

### Load Balancing
- Application Load Balancer
- Target Group
- HTTP Listener

### Storage
- S3 Bucket for application artifacts

### CI/CD
- CodePipeline
- CodeBuild Project
- GitHub Webhook

### IAM
- CodePipeline Service Role
- CodeBuild Service Role
- CloudFormation Service Role
- ECS Task Execution Role
- Lambda Execution Role

## Customization

### Update Workstream
Edit all parameter files and update:
```json
"Workstream": "YourWorkstream"
```

### Update Application Name
Edit all parameter files and update:
```json
"AppName": "YourAppName",
"ProjectName": "YourAppName"
```

### Modify VPC CIDR
Edit environment parameter files (`params/*.json`):
```json
"VpcCidr": "10.X.0.0/16"
```

## Documentation

See [CODEPIPELINE_SETUP_GUIDE.md](../CODEPIPELINE_SETUP_GUIDE.md) for:
- Detailed step-by-step setup instructions
- Troubleshooting guide
- Best practices
- Multi-environment deployment

## Clean Up

To delete all resources:

```bash
# Delete infrastructure stack
aws cloudformation delete-stack \
  --stack-name hq-Workstream-dev-MyApp-Stack-Infrastructure

# Delete pipeline stack
aws cloudformation delete-stack \
  --stack-name hq-Workstream-dev-MyApp-Stack-Pipeline

# Empty and delete S3 buckets
aws s3 rm s3://hq-workstream-dev-myapp-s3-cfn-package --recursive
aws s3 rb s3://hq-workstream-dev-myapp-s3-cfn-package

aws s3 rm s3://hq-workstream-dev-myapp-s3-pipeline-artifacts --recursive
aws s3 rb s3://hq-workstream-dev-myapp-s3-pipeline-artifacts
```

## Support

For issues or questions:
1. Check the troubleshooting section in CODEPIPELINE_SETUP_GUIDE.md
2. Review CloudFormation stack events
3. Check CodeBuild and CodePipeline logs
4. Contact your DevOps team

## License

This project is for internal use only.

---

**Version:** 1.0
**Last Updated:** 2026-01-11
