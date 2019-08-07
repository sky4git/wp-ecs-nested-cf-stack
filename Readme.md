# Wordpress Nested CloudFormation Stack

## Step 1
### Create CodeBuild project
In the [wp-ecs-nested-cf-stack](wp-ecs-nested-cf-stack/) directory there are multiple YAML (*CloudFormation Templates*) & JSON (*CloudFormation Configuration*) files.

**[codebuild-stack.yml](wp-ecs-nested-cf-stack/codebuild-stack.yml):** is the CloudFormation template to create CodeBuild project which will be used to run buildpsec.yml file to create other stacks once setup.

Rename **codebuild-params-sample.json** to **codebuild-params.json** and fill with the appropriate values.

### For non root CLI user
You will also need to add IAM policiy for on root CLI user. So create a IAM policy called **CFAPIAccess** and attached it to user or user group.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor0",
      "Effect": "Allow",
      "Action": [
        "codebuild:DeleteWebhook",
        "iam:DeletePolicy",
        "iam:CreateRole",
        "iam:DeleteRole",
        "codebuild:ListSourceCredentials",
        "iam:AttachRolePolicy",
        "codebuild:UpdateWebhook",
        "iam:PutRolePolicy",
        "codebuild:CreateWebhook",
        "codebuild:UpdateProject",
        "codebuild:CreateProject",
        "iam:CreatePolicy",
        "iam:PassRole",
        "iam:DeleteRolePolicy",
        "codebuild:DeleteProject",
        "codebuild:ListProjects",
        "codebuild:ListConnectedOAuthAccounts",
        "iam:PutGroupPolicy"
      ],
      "Resource": "*"
    }
  ]
}
```
Also assign AWS managed policy **AWSCloudFormationFullAccess** to user group or user.

### Run commands
```bash
aws cloudformation validate-template --template-body file://codebuild-stack.yml    

aws cloudformation create-stack --stack-name WPECSCodeBuild --template-body file://codebuild-stack.yml --parameters file://codebuild-params.json --capabilities CAPABILITY_NAMED_IAM

aws cloudformation describe-stacks --stack-name WPECSCodeBuild
```
Check in AWS console for events output.

Copy S3BucketName from the outputs.

## Step 2:

### Create base WPECSStack Stack
In the [wp-ecs-nested-cf-stack](wp-ecs-nested-cf-stack/) directory there are multiple YAML (*CloudFormation Templates*) & JSON (*CloudFormation Configuration*) files.

**[Vpc-stack.yml](wp-ecs-nested-cf-stack/vpc-stack.yml):** is the CloudFormation template to create the base VPC, Subnets, NAT Gateways, etc which will be used.
**[vpc-params.json](wp-ecs-nested-cf-stack/vpc-params.json):** is the parameters file which contains the parameter values for the CFN template. 

Go to `wp-ecs-nested-cf-stack` directory and execute the following AWS CLI command to create CloudFormation stack.

```bash
aws cloudformation validate-template --template-body file://WPECSStack.yml 

aws cloudformation create-stack --stack-name WPECSStack --template-body file://WPECSStack.yml --parameters file://WPECSStack-params.json --capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND
```