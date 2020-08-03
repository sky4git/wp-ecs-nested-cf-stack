![Build Status](https://codebuild.ap-southeast-2.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoiZWRSR3BTODNTVldxSkYrZ3FDaTJBeWYzNjc4anNwbWJ3SWV4d2xYR25EMHJFY3VOdmlDYi9EbE54TC85MDVjMDNKcnl3b3JLUEFGRWtyMkNZZ25UNDBzPSIsIml2UGFyYW1ldGVyU3BlYyI6InQ2T1ZkenNNWE9BR3pzb2EiLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master)


# Wordpress Nested CloudFormation Stack

## Step 1
### Create CodeBuild project
In the [wp-ecs-nested-cf-stack](wp-ecs-nested-cf-stack/) directory there are multiple YAML (*CloudFormation Templates*) & JSON (*CloudFormation Configuration*) files.

**[codebuild-stack.yml](wp-ecs-nested-cf-stack/codebuild-stack.yml):** is the CloudFormation template to create CodeBuild project which will be used to run buildpsec.yml file to create other stacks once setup.

Rename **codebuild-params-sample.json** to **codebuild-params.json** and fill with the appropriate values.


### Run commands
```bash
aws cloudformation validate-template --template-body file://codebuild-stack.yml    

aws cloudformation create-stack --stack-name WPECSCodeBuild --template-body file://codebuild-stack.yml --parameters file://codebuild-params.json --capabilities CAPABILITY_NAMED_IAM

aws cloudformation describe-stacks --stack-name WPECSCodeBuild

# Create and see changes from change set but no deploy
aws cloudformation deploy --stack-name WPECSCodeBuild --template-file codebuild-stack.yml --capabilities CAPABILITY_NAMED_IAM --no-execute-changeset 
# Exceute a change set you have created from above command 
aws cloudformation execute-change-set --change-set-name [ARN]
# Delete the changeset you have created from deploy command 
aws cloudformation delete-change-set --change-set-name [ARN]

# Create and deploy change set without seeing the changes 
aws cloudformation deploy --stack-name WPECSCodeBuild --template-file codebuild-stack.yml --capabilities CAPABILITY_NAMED_IAM 
```
Check in AWS console for events output.

Copy S3BucketName from the outputs.

## step 2

### Create Apache and Php docker images
Create Apache and PHP Docker/ECR images from the following GitHub repo. 
**[https://github.com/sky4git/docker](https://github.com/sky4git/docker)**

## Step 3:

### Create base WPECSStack Stack
In the [wp-ecs-nested-cf-stack](wp-ecs-nested-cf-stack/) directory there are multiple YAML (*CloudFormation Templates*) & JSON (*CloudFormation Configuration*) files.

**[vpc-stack.yml](wp-ecs-nested-cf-stack/vpc-stack.yml):** is the CloudFormation template to create the base VPC, Subnets, NAT Gateways, etc which will be used.
**[security-stack.yml](wp-ecs-nested-cf-stack/security-stack.yml):** is the CloudFormation template to create the SecurityGroups.
**[ecs-stack.yml](wp-ecs-nested-cf-stack/ecs-stack.yml):** is the CloudFormation template to create the ECS cluster.
**[rds-stack.yml](wp-ecs-nested-cf-stack/rds-stack.yml):** is the CloudFormation template to create the RDS cluster.
**[config-params.json](wp-ecs-nested-cf-stack/cofig-params.json):** is the parameters file which contains the parameter values for the CFN template. 

To create a new enviroment, just update ***config-params.json*** file with the new values and push the changes in github.

To change the environment type to you will need to update Enviroment variable in your CodeBuild project.
You can either update it from Console or you can update it from CLI command line. For example,

```bash
aws codebuild update-project --name "CodebuidProjectName" --environment "type=LINUX_CONTAINER,image=aws/codebuild/standard:2.0,computeType=BUILD_GENERAL1_SMALL,environmentVariables=[{name=TEMPLATE_BUCKET,value=bucket_name_here,type=PLAINTEXT},{name=TEMPLATE_PREFIX,value=prod,type=PLAINTEXT}],imagePullCredentialsType=CODEBUILD"
```
Please replace *CodebuidProjectName* with your actual project name and *bucket_name_here* with your S3 bucket name.

# Reference architecture
Please note that not all components are in the template. such as codepipeline, sqs, waf and cloudfront distribution at this stage.
![Reference architecture](/WordPress-on-Cloud-Simple.jpg "Reference architecture")