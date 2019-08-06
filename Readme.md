# Wordpress Nested CloudFormation Stack

## Step 1:

### Create base VPC Stack
In the [wp-ecs-nested-cf-stack](wp-ecs-nested-cf-stack/) directory there are multiple YAML (*CloudFormation Templates*) & JSON (*CloudFormation Configuration*) files.

**[Vpc-stack.yml](wp-ecs-nested-cf-stack/Vpc-stack.yml):** is the CloudFormation template to create the base VPC, Subnets, NAT Gateways, etc which will be used.
**[vpc-params.json](wp-ecs-nested-cf-stack/Vpc-params.json):** is the parameters file which contains the parameter values for the CFN template. 

Go to `wp-ecs-nested-cf-stack` directory and execute the following AWS CLI command to create CloudFormation stack.

```bash
cd wp-ecs-nested-cf-stack
aws cloudformation create-stack --stack-name WPECSNested-BaseStack --template-body file://Vpc-stack.yml --parameters file://Vpc-params.json
```