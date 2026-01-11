# AWS CodePipeline Setup Guide
## Step-by-Step Manual Setup for CloudFormation Deployment

This guide provides comprehensive instructions for setting up AWS CodePipeline manually to deploy CloudFormation templates with proper naming conventions.

---

## Table of Contents
1. [Naming Convention Standard](#naming-convention-standard)
2. [Prerequisites](#prerequisites)
3. [Setup Steps](#setup-steps)
4. [Parameter Configuration](#parameter-configuration)
5. [Deployment Instructions](#deployment-instructions)
6. [Troubleshooting](#troubleshooting)

---

## Naming Convention Standard

All resources follow the standardized naming pattern:

```
hq-<Workstream>-<Env>-<AppName>-<ResourceType>-<Purpose>
```

**Examples:**
- VPC: `hq-Workstream-dev-MyApp-VPC-Main`
- S3 Bucket: `hq-workstream-dev-myapp-s3-artifacts-123456789012`
- ECS Cluster: `hq-Workstream-dev-MyApp-ECS-Cluster`
- Lambda Function: `hq-Workstream-dev-MyApp-Lambda-Hello`
- CodePipeline: `hq-Workstream-dev-MyApp-Pipeline-CFNDeploy`

**Parameter Definitions:**
- `hq`: Headquarters prefix (constant)
- `Workstream`: Business workstream identifier (e.g., Finance, HR, IT, Engineering)
- `Env`: Environment (dev, qa, uat, prod)
- `AppName`: Application name (e.g., MyApp, CustomerPortal)
- `ResourceType`: AWS resource type (VPC, S3, ECS, Lambda, etc.)
- `Purpose`: Resource purpose (Main, Artifacts, Cluster, etc.)

---

## Prerequisites

Before starting, ensure you have:

### 1. AWS Account Setup
- AWS account with appropriate permissions
- IAM user with Administrator or PowerUser access
- AWS CLI installed and configured

### 2. GitHub Repository
- GitHub repository with your code
- GitHub Personal Access Token (PAT) with `repo` scope
  - Go to GitHub Settings → Developer settings → Personal access tokens
  - Generate new token with `repo` scope
  - Save the token securely

### 3. S3 Buckets
You need **TWO** S3 buckets:

#### a) CFN Package Bucket
For CloudFormation package artifacts (nested templates):
```bash
aws s3 mb s3://hq-workstream-dev-myapp-s3-cfn-package --region us-east-1
```

#### b) Pipeline Artifacts Bucket
For CodePipeline artifacts:
```bash
aws s3 mb s3://hq-workstream-dev-myapp-s3-pipeline-artifacts --region us-east-1
```

**Note:** Replace `dev` with your environment (qa, uat, prod) and update the workstream/app names accordingly.

---

## Setup Steps

### Step 1: Prepare Your Repository

1. **Clone this repository** (if you haven't already):
   ```bash
   git clone <your-repo-url>
   cd AWSCodePipelineTest
   ```

2. **Review the project structure**:
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
   └── buildspec.yml                   # CodeBuild build specification
   ```

### Step 2: Update Pipeline Parameters

Edit the pipeline parameter file for your environment:

**File:** `aws-cfn-codepipeline/pipeline-params/dev-pipeline.json`

```json
{
  "Parameters": {
    "Env": "dev",
    "Workstream": "YourWorkstream",
    "AppName": "YourAppName",
    "GitHubOwner": "your-github-username",
    "GitHubRepo": "AWSCodePipelineTest",
    "GitHubBranch": "main",
    "GitHubOAuthToken": "ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxx",
    "CFNPackageBucket": "hq-yourworkstream-dev-yourapp-s3-cfn-package",
    "PipelineArtifactBucket": "hq-yourworkstream-dev-yourapp-s3-pipeline-artifacts"
  }
}
```

**Update these values:**
- `Workstream`: Your business workstream (e.g., "Engineering", "Finance")
- `AppName`: Your application name (e.g., "MyApp")
- `GitHubOwner`: Your GitHub username or organization
- `GitHubRepo`: Your repository name
- `GitHubBranch`: Branch to track (usually `main` or `develop`)
- `GitHubOAuthToken`: Your GitHub Personal Access Token
- `CFNPackageBucket`: The S3 bucket you created for CFN packages
- `PipelineArtifactBucket`: The S3 bucket you created for pipeline artifacts

### Step 3: Update Infrastructure Parameters

Edit the infrastructure parameter file for your environment:

**File:** `aws-cfn-codepipeline/params/dev.json`

```json
{
  "Parameters": {
    "Env": "dev",
    "Workstream": "YourWorkstream",
    "ProjectName": "YourAppName",
    "VpcCidr": "10.10.0.0/16",
    "PublicSubnet1Cidr": "10.10.10.0/24",
    "PublicSubnet2Cidr": "10.10.20.0/24",
    "PrivateSubnet1Cidr": "10.10.110.0/24",
    "PrivateSubnet2Cidr": "10.10.120.0/24"
  }
}
```

**Update these values:**
- `Workstream`: Same as pipeline parameters
- `ProjectName`: Same as `AppName` in pipeline parameters
- Adjust CIDR blocks if needed (ensure they don't conflict with existing VPCs)

---

## Parameter Configuration

### Environment-Specific CIDR Blocks

Each environment should have unique CIDR blocks to avoid conflicts:

| Environment | VPC CIDR      | Public Subnet 1 | Public Subnet 2 | Private Subnet 1 | Private Subnet 2 |
|-------------|---------------|-----------------|-----------------|------------------|------------------|
| dev         | 10.10.0.0/16  | 10.10.10.0/24   | 10.10.20.0/24   | 10.10.110.0/24   | 10.10.120.0/24   |
| qa          | 10.20.0.0/16  | 10.20.10.0/24   | 10.20.20.0/24   | 10.20.110.0/24   | 10.20.120.0/24   |
| uat         | 10.30.0.0/16  | 10.30.10.0/24   | 10.30.20.0/24   | 10.30.110.0/24   | 10.30.120.0/24   |
| prod        | 10.40.0.0/16  | 10.40.10.0/24   | 10.40.20.0/24   | 10.40.110.0/24   | 10.40.120.0/24   |

---

## Deployment Instructions

### Option 1: Deploy CodePipeline via AWS Console

#### Step 1: Create the CodePipeline Stack

1. **Navigate to CloudFormation Console**:
   - Go to AWS Console → CloudFormation
   - Click "Create stack" → "With new resources (standard)"

2. **Upload Template**:
   - Choose "Upload a template file"
   - Upload: `aws-cfn-codepipeline/templates/pipeline/codepipeline.yml`
   - Click "Next"

3. **Specify Stack Details**:
   - Stack name: `hq-YourWorkstream-dev-YourApp-Stack-Pipeline`
   - Parameters: Enter values from your `dev-pipeline.json` file
   - Click "Next"

4. **Configure Stack Options**:
   - Tags (optional):
     - Key: `Workstream`, Value: `YourWorkstream`
     - Key: `Env`, Value: `dev`
     - Key: `AppName`, Value: `YourAppName`
   - Click "Next"

5. **Review and Create**:
   - Check "I acknowledge that AWS CloudFormation might create IAM resources with custom names"
   - Click "Create stack"

6. **Monitor Creation**:
   - Wait for stack status to become `CREATE_COMPLETE` (typically 5-10 minutes)
   - Check the "Events" tab for progress
   - Check the "Outputs" tab for the CodePipeline URL

#### Step 2: Verify CodePipeline

1. **Access CodePipeline**:
   - Go to AWS Console → CodePipeline
   - You should see: `hq-YourWorkstream-dev-YourApp-Pipeline-CFNDeploy`

2. **Check Pipeline Stages**:
   - **Source**: Should connect to your GitHub repository
   - **Build**: Should use CodeBuild to package templates
   - **Deploy**: Should deploy via CloudFormation

3. **First Execution**:
   - The pipeline may start automatically (if GitHub webhook is configured)
   - Or click "Release change" to start manually

### Option 2: Deploy CodePipeline via AWS CLI

```bash
# Navigate to the project directory
cd aws-cfn-codepipeline

# Deploy the CodePipeline stack
aws cloudformation create-stack \
  --stack-name hq-YourWorkstream-dev-YourApp-Stack-Pipeline \
  --template-body file://templates/pipeline/codepipeline.yml \
  --parameters file://pipeline-params/dev-pipeline.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1

# Monitor stack creation
aws cloudformation wait stack-create-complete \
  --stack-name hq-YourWorkstream-dev-YourApp-Stack-Pipeline \
  --region us-east-1

# Get stack outputs
aws cloudformation describe-stacks \
  --stack-name hq-YourWorkstream-dev-YourApp-Stack-Pipeline \
  --region us-east-1 \
  --query 'Stacks[0].Outputs'
```

---

## Manual Deployment Without CodePipeline

If you want to deploy the infrastructure directly without CodePipeline:

### Step 1: Package the Template

```bash
cd aws-cfn-codepipeline

# Package the nested stack template
aws cloudformation package \
  --template-file templates/root.yml \
  --s3-bucket hq-yourworkstream-dev-yourapp-s3-cfn-package \
  --output-template-file packaged-root.yml \
  --region us-east-1
```

### Step 2: Deploy the Infrastructure

```bash
# Deploy the infrastructure stack
aws cloudformation create-stack \
  --stack-name hq-YourWorkstream-dev-YourApp-Stack-Infrastructure \
  --template-body file://packaged-root.yml \
  --parameters file://params/dev.json \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1

# Monitor deployment
aws cloudformation wait stack-create-complete \
  --stack-name hq-YourWorkstream-dev-YourApp-Stack-Infrastructure \
  --region us-east-1
```

---

## Pipeline Workflow

Once deployed, the CodePipeline follows this workflow:

### 1. Source Stage
- **Trigger**: Git push to the specified branch
- **Action**: GitHub webhook triggers pipeline
- **Output**: Source code artifact

### 2. Build Stage
- **Input**: Source code artifact
- **CodeBuild Actions**:
  1. Install dependencies (cfn-lint)
  2. Lint CloudFormation templates
  3. Package nested templates to S3
  4. Output packaged template
- **Output**: `packaged-root.yml` and parameter files

### 3. Deploy Stage

#### 3a. CreateChangeSet Action
- **Input**: Build output artifacts
- **Action**: Create CloudFormation change set
- **Configuration**:
  - Stack name: `hq-YourWorkstream-dev-YourApp-Stack-Infrastructure`
  - Template: `packaged-root.yml`
  - Parameters: `params/dev.json`
  - Capabilities: `CAPABILITY_NAMED_IAM`

#### 3b. ExecuteChangeSet Action
- **Input**: Change set from previous action
- **Action**: Execute the change set
- **Result**: Infrastructure deployed/updated

---

## Resource Naming Examples

After deployment, you'll see resources with these naming patterns:

### Network Resources
- VPC: `hq-Workstream-dev-MyApp-VPC-Main`
- Internet Gateway: `hq-Workstream-dev-MyApp-IGW-Main`
- NAT Gateway: `hq-Workstream-dev-MyApp-NAT-Main`
- Public Subnet A: `hq-Workstream-dev-MyApp-Subnet-Public-A`
- Private Subnet A: `hq-Workstream-dev-MyApp-Subnet-Private-A`
- Public Route Table: `hq-Workstream-dev-MyApp-RT-Public`

### Security
- ALB Security Group: `hq-Workstream-dev-MyApp-SG-ALB`
- ECS Security Group: `hq-Workstream-dev-MyApp-SG-ECS`

### Storage
- Artifacts S3 Bucket: `hq-workstream-dev-myapp-s3-artifacts-123456789012`

### Compute & Load Balancing
- Application Load Balancer: `hq-Workstream-dev-MyApp-ALB`
- Target Group: `hq-Workstream-dev-MyApp-TG`
- ECS Cluster: `hq-Workstream-dev-MyApp-ECS-Cluster`
- ECS Service: `hq-Workstream-dev-MyApp-Service`
- Lambda Function: `hq-Workstream-dev-MyApp-Lambda-Hello`

### IAM Roles
- ECS Task Execution Role: `hq-Workstream-dev-MyApp-Role-ECSTaskExecution`
- Lambda Role: `hq-Workstream-dev-MyApp-Role-Lambda`
- CodePipeline Role: `hq-Workstream-dev-MyApp-Role-CodePipeline`
- CodeBuild Role: `hq-Workstream-dev-MyApp-Role-CodeBuild`

---

## Troubleshooting

### Common Issues and Solutions

#### 1. CodePipeline Source Stage Fails

**Problem:** GitHub connection fails

**Solutions:**
- Verify GitHub token is valid and has `repo` scope
- Check GitHub repository owner and name are correct
- Ensure branch name matches (case-sensitive)
- Verify webhook is created in GitHub repository settings

#### 2. Build Stage Fails - S3 Access Denied

**Problem:** CodeBuild cannot access S3 bucket

**Solutions:**
- Verify CFN package bucket name is correct
- Check bucket exists in the same region
- Ensure CodeBuild service role has S3 permissions
- Verify bucket policy allows CodeBuild to write

**Fix bucket permissions:**
```bash
aws s3api put-bucket-policy \
  --bucket hq-yourworkstream-dev-yourapp-s3-cfn-package \
  --policy file://bucket-policy.json
```

#### 3. Deploy Stage Fails - Insufficient Permissions

**Problem:** CloudFormation cannot create resources

**Solutions:**
- Check CloudFormation service role has necessary permissions
- Verify `CAPABILITY_NAMED_IAM` is enabled
- Review CloudFormation stack events for specific errors

#### 4. Resource Name Conflicts

**Problem:** Resource already exists with the same name

**Solutions:**
- Delete existing resources or update parameters
- Change `Workstream` or `AppName` parameters
- Use different environment names

#### 5. VPC CIDR Conflicts

**Problem:** VPC CIDR overlaps with existing VPC

**Solutions:**
- Update CIDR blocks in parameter files
- Use unique CIDR ranges for each environment
- Check existing VPCs: `aws ec2 describe-vpcs`

### Debugging Commands

**Check stack status:**
```bash
aws cloudformation describe-stacks \
  --stack-name hq-YourWorkstream-dev-YourApp-Stack-Pipeline \
  --query 'Stacks[0].StackStatus'
```

**View stack events:**
```bash
aws cloudformation describe-stack-events \
  --stack-name hq-YourWorkstream-dev-YourApp-Stack-Infrastructure \
  --max-items 20
```

**Check CodeBuild logs:**
```bash
aws codebuild batch-get-builds \
  --ids <build-id> \
  --query 'builds[0].logs'
```

**List pipeline executions:**
```bash
aws codepipeline list-pipeline-executions \
  --pipeline-name hq-YourWorkstream-dev-YourApp-Pipeline-CFNDeploy
```

---

## Clean Up

To delete all resources:

### 1. Delete Infrastructure Stack
```bash
aws cloudformation delete-stack \
  --stack-name hq-YourWorkstream-dev-YourApp-Stack-Infrastructure
```

### 2. Delete Pipeline Stack
```bash
aws cloudformation delete-stack \
  --stack-name hq-YourWorkstream-dev-YourApp-Stack-Pipeline
```

### 3. Empty and Delete S3 Buckets
```bash
# Empty CFN package bucket
aws s3 rm s3://hq-yourworkstream-dev-yourapp-s3-cfn-package --recursive

# Empty pipeline artifacts bucket
aws s3 rm s3://hq-yourworkstream-dev-yourapp-s3-pipeline-artifacts --recursive

# Delete buckets
aws s3 rb s3://hq-yourworkstream-dev-yourapp-s3-cfn-package
aws s3 rb s3://hq-yourworkstream-dev-yourapp-s3-pipeline-artifacts
```

---

## Multi-Environment Deployment

To deploy to multiple environments (QA, UAT, PROD):

### 1. Create Environment-Specific S3 Buckets
```bash
# QA
aws s3 mb s3://hq-workstream-qa-myapp-s3-cfn-package
aws s3 mb s3://hq-workstream-qa-myapp-s3-pipeline-artifacts

# UAT
aws s3 mb s3://hq-workstream-uat-myapp-s3-cfn-package
aws s3 mb s3://hq-workstream-uat-myapp-s3-pipeline-artifacts

# PROD
aws s3 mb s3://hq-workstream-prod-myapp-s3-cfn-package
aws s3 mb s3://hq-workstream-prod-myapp-s3-pipeline-artifacts
```

### 2. Update Parameter Files
Edit `qa-pipeline.json`, `uat-pipeline.json`, `prod-pipeline.json` with appropriate values

### 3. Deploy Pipelines
```bash
# QA Pipeline
aws cloudformation create-stack \
  --stack-name hq-YourWorkstream-qa-YourApp-Stack-Pipeline \
  --template-body file://templates/pipeline/codepipeline.yml \
  --parameters file://pipeline-params/qa-pipeline.json \
  --capabilities CAPABILITY_NAMED_IAM

# UAT Pipeline
aws cloudformation create-stack \
  --stack-name hq-YourWorkstream-uat-YourApp-Stack-Pipeline \
  --template-body file://templates/pipeline/codepipeline.yml \
  --parameters file://pipeline-params/uat-pipeline.json \
  --capabilities CAPABILITY_NAMED_IAM

# PROD Pipeline
aws cloudformation create-stack \
  --stack-name hq-YourWorkstream-prod-YourApp-Stack-Pipeline \
  --template-body file://templates/pipeline/codepipeline.yml \
  --parameters file://pipeline-params/prod-pipeline.json \
  --capabilities CAPABILITY_NAMED_IAM
```

---

## Best Practices

1. **Version Control**: Always commit parameter changes to Git
2. **Secrets Management**: Use AWS Secrets Manager for sensitive data (GitHub tokens)
3. **Tagging**: Maintain consistent tagging across all resources
4. **Monitoring**: Set up CloudWatch alarms for pipeline failures
5. **Testing**: Test in dev/qa before deploying to production
6. **Backup**: Enable versioning on S3 buckets
7. **Documentation**: Keep this guide updated with your specific configurations

---

## Support and Contact

For issues or questions:
1. Check the [Troubleshooting](#troubleshooting) section
2. Review CloudFormation stack events
3. Check CodeBuild and CodePipeline logs
4. Contact your DevOps team

---

## Appendix: AWS Console URLs

- **CloudFormation**: https://console.aws.amazon.com/cloudformation/
- **CodePipeline**: https://console.aws.amazon.com/codesuite/codepipeline/
- **CodeBuild**: https://console.aws.amazon.com/codesuite/codebuild/
- **S3**: https://s3.console.aws.amazon.com/s3/
- **IAM**: https://console.aws.amazon.com/iam/
- **VPC**: https://console.aws.amazon.com/vpc/
- **EC2**: https://console.aws.amazon.com/ec2/
- **ECS**: https://console.aws.amazon.com/ecs/
- **Lambda**: https://console.aws.amazon.com/lambda/

---

**Last Updated:** 2026-01-11
**Version:** 1.0
