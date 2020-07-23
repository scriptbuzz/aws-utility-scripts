Environment: AWS CLI, Bash, EC2, Amazon Linux, and least privileged permissions.

**Task: These settings are a must for bash script debugging**

- set -o errexit  # abort on nonzero exitstatus
- set -o pipefail # don't hide errors within pipes
- For JSON parsing: sudo yum install jq

**TASK: Check if user has AWS credentials from EC2 by running command: aws sts get-caller-identity**

aws sts get-caller-identity
{
    "Account": "123456789012",
    "UserId": "XKIAIV4QH8DVBZXDMKQE",
    "Arn": "arn:aws:iam::123456789012:user/app-developer01"
}

**TASK: return AWS account ID and assign to variable AWS_ACCOUNT_ID**

- Method 1: AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query 'Account' --output text)
- Method 2: AWS_ACCOUNT_ID=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r .accountId)
Above example requires the jq package

**TASK: Using the CLI and Bash and EC2 metadata call, determine AWS region and assign to MY_AWS_REGIONS**

MY_AWS_REGIONS=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone | sed 's/\(.*\)[a-z]/\1/')

**TASK: Using the variable MY_AWS_REGIONS from previous command, we can test if we are in our supported regions and echo results**

SUPPORTED_REGIONS=("us-east-1" "us-east-2" )
if [[ ! " ${SUPPORTED_REGIONS[@]} " =~ " ${MY_AWS_REGIONS} " ]]; then
    /bin/echo -e "'$MY_AWS_REGIONS' is not a supported AWS Region." 
else
    /bin/echo -e "'$MY_AWS_REGIONS' is a supported AWS Region." 
fi


**TASK: Return a physical resource id from a CloudFormation stack and assign to a variable. In this example, return the name of an S3 bucket and assign to MY_BUCKET**

MY_BUCKET=$(aws cloudformation describe-stack-resource --stack-name my-stack --logical-resource-id bucket --query "StackResourceDetail.PhysicalResourceId" --output text)

**TASK: Return AWS endpoint address of a service resource. In this example, return an ATS signed data endpoint and assign to MY_ENDPOINT_HOST**

MY_ENDPOINT_HOST=$(aws iot describe-endpoint --endpoint-type iot:Data-ATS | grep endpointAddress | cut -d'"' -f 4)

**Task: Return the IAM Role of a resource in a CloudFormation stack. The example below returns the role of a Lambda function and assign to MY_LAMBDA_ROLE**

MY_LAMBDA_ROLE=$(aws cloudformation describe-stack-resource --stack-name my-stack --logical-resource-id MyLambdaRole --query "StackResourceDetail.PhysicalResourceId" --output text)

**Task: Return the ARN of an IAM Role. In this example, build the ARN string and assign to MY_LAMBDA_ROLE_ARN**

MY_LAMBDA_ROLE_ARN=$(aws iam get-role --role-name $MY_LAMBDA_ROLE | grep Arn | cut -d'"' -f 4)
