[AWS Codebuild Status](https://codebuild.ap-southeast-2.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoiT2h6T1ZCc1FxTlVMSGJENktZN21UcWFFZXpOcDdSU2ZySEJuSVZrMDkvbjF3YlJvcHBsb0JtTnB6Q2RMWGdCTURYSFd6MzdVTndhdk1sNmdJMXZJV0hNPSIsIml2UGFyYW1ldGVyU3BlYyI6IjB1YjlVRGRMeW5PNFp0UEQiLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master)

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

**[vpc-stack.yml](wp-ecs-nested-cf-stack/vpc-stack.yml):** is the CloudFormation template to create the base VPC, Subnets, NAT Gateways, etc which will be used.
**[security-stack.yml](wp-ecs-nested-cf-stack/security-stack.yml):** is the CloudFormation template to create the SecurityGroups.
**[config-params.json](wp-ecs-nested-cf-stack/cofig-params.json):** is the parameters file which contains the parameter values for the CFN template. 

To create a new enviroment, just update ***config-params.json*** file with the new values and push the changes in github.

To change the environment type to you will need to update Enviroment variable in your CodeBuild project.
You can either update it from Console or you can update it from CLI command line. For example,

```bash
aws codebuild update-project --name "CodebuidProjectName" --environment "type=LINUX_CONTAINER,image=aws/codebuild/standard:2.0,computeType=BUILD_GENERAL1_SMALL,environmentVariables=[{name=TEMPLATE_BUCKET,value=bucket_name_here,type=PLAINTEXT},{name=TEMPLATE_PREFIX,value=prod,type=PLAINTEXT}],imagePullCredentialsType=CODEBUILD"
```
Please replace *CodebuidProjectName* with your actual project name and *bucket_name_here* with your S3 bucket name.