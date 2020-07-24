# aws-utility-scripts
A collection of AWS automations and commands that I use frequently which I have collected overtime. The majority of the automations have been shamelessly borrowed with modifications. Thanks to all the developers of the original works and to the AWS documentation. 

**Environment**
```
- AWS EC2/Amazon Linux
- AWS CLI
- Least privileged permissions
```

**Settings I consider a must for bash script debugging and development for the AWS CLI**
```
- set -o errexit  # abort on nonzero exitstatus
- set -o pipefail # don't hide errors within pipes
- For JSON parsing: sudo yum install jq
```
**An EC2 bootstrap script I found useful as a starting point**

https://github.com/awslabs/amazon-guardduty-tester/blob/master/bastion_bootstrap.sh

**Determine host OS**

```
function osrelease () {
    OS=`cat /etc/os-release | grep '^NAME=' |  tr -d \" | sed 's/\n//g' | sed 's/NAME=//g'`
    if [ "$OS" == "Ubuntu" ]; then
        echo "Ubuntu"
    elif [ "$OS" == "Amazon Linux" ]; then
        echo "AMZN"
    elif [ "$OS" == "CentOS Linux" ]; then
        echo "CentOS"
    else
        echo "Operating System Not Found"
    fi
    echo "${FUNCNAME[0]} Ended" >> /var/log/cfn-init.log
}

```
**Check if user has AWS credentials from EC2 by running command: aws sts get-caller-identity**

```aws sts get-caller-identity
{
    "Account": "123456789012",
    "UserId": "XKIAIV4QH8DVBZXDMKQE",
    "Arn": "arn:aws:iam::123456789012:user/app-developer01"
}
```

**Return the public IP address of an EC2 instance, if any**
```
curl https://checkip.amazonaws.com
```
**Create a key pair and pipe your private key directly to a local file.**

```
aws ec2 create-key-pair --key-name ThisKeyPair --query 'KeyMaterial' --output text > ThisKeyPair.pem
```

**Return AWS account ID and assign to variable**

```
- Method 1: AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query 'Account' --output text)
- Method 2: AWS_ACCOUNT_ID=$(curl -s http://169.254.169.254/latest/dynamic/instance-identity/document | jq -r .accountId)
```
Above example requires the jq package

**Using the EC2 metadata, determine AWS region and assign to var**

```MY_AWS_REGIONS=$(curl -s http://169.254.169.254/latest/meta-data/placement/availability-zone | sed 's/\(.*\)[a-z]/\1/')```

**Using the variable MY_AWS_REGIONS from previous command, we can test if we are in our supported regions and echo results**

```
SUPPORTED_REGIONS=("us-east-1" "us-east-2" )
if [[ ! " ${SUPPORTED_REGIONS[@]} " =~ " ${MY_AWS_REGIONS} " ]]; then
    /bin/echo -e "'$MY_AWS_REGIONS' is not a supported AWS Region." 
else
    /bin/echo -e "'$MY_AWS_REGIONS' is a supported AWS Region." 
fi
```

**Return a physical resource id from a CloudFormation stack and assign to a variable. In this example, return the name of an S3 bucket and assign to var**

```MY_BUCKET=$(aws cloudformation describe-stack-resource --stack-name my-stack --logical-resource-id bucket --query "StackResourceDetail.PhysicalResourceId" --output text)```

**Return AWS endpoint address of a service resource. In this example, return an ATS signed data endpoint and assign to var**

```MY_ENDPOINT_HOST=$(aws iot describe-endpoint --endpoint-type iot:Data-ATS | grep endpointAddress | cut -d'"' -f 4)```

**Return the IAM Role of a resource in a CloudFormation stack. The example below returns the role of a Lambda function and assign to var**

```MY_LAMBDA_ROLE=$(aws cloudformation describe-stack-resource --stack-name my-stack --logical-resource-id MyLambdaRole --query "StackResourceDetail.PhysicalResourceId" --output text)```

**Return the ARN of an IAM Role. In this example, build the ARN string and assign to var**

```MY_LAMBDA_ROLE_ARN=$(aws iam get-role --role-name $MY_LAMBDA_ROLE | grep Arn | cut -d'"' -f 4)```

**Filter list of EC2 instances to a specific instance type. In this example, filter on t3.xlarge instances and outputs only the InstanceId for all matches**

```
aws ec2 describe-instances --filters "Name=instance-type,Values=t3.xlarge" --query "Reservations[].Instances[].InstanceId"
```
**List EC2 instances that match the tag key=value ENV=DEV**
```
aws ec2 describe-instances --filters "Name=tag:ENV,Values=DEV"
```
**List EC2 instances with AMI ids ami-3333333 and ami-2222222**
```
aws ec2 describe-instances --filters "Name=image-id,Values=ami-3333333,ami-2222222"
```
**Basic S3 operations CLI commands**

```
// Delete local file
$ rm ./MyFile1.txt

// Attempt sync without --delete option
$ aws s3 sync . s3://my-bucket/path

// Sync with deletion - object is deleted from bucket
$ aws s3 sync . s3://my-bucket/path --delete

// Delete object from bucket
$ aws s3 rm s3://my-bucket/path/MySubdirectory/MyFile3.txt

// Sync with deletion - local file is deleted
$ aws s3 sync s3://my-bucket/path . --delete

// Sync with Infrequent Access storage class
$ aws s3 sync . s3://my-bucket/path --storage-class STANDARD_IA

// Sync but exclude files with extention txt
$ aws s3 sync . s3://my-bucket/path --exclude "*.txt"

// Sync but exclude files with extension txt unless the file names start with MyFile then include files
$ aws s3 sync . s3://my-bucket/path --exclude "*.txt" --include "MyFile*.txt"

// Sync but exclude files with extension txt unless the file names start with MyFile then include files except when there's only a signe char match
$ aws s3 sync . s3://my-bucket/path --exclude "*.txt" --include "MyFile*.txt" --exclude "MyFile?.txt"

// Copy MyFile.txt in current directory to s3://my-bucket/path
$ aws s3 cp MyFile.txt s3://my-bucket/path/

// Move all .jpg files in s3://my-bucket/path to ./MyDirectory
$ aws s3 mv s3://my-bucket/path ./MyDirectory --exclude "*" --include "*.jpg" --recursive

// List the contents of my-bucket
$ aws s3 ls s3://my-bucket

// List the contents of path in my-bucket
$ aws s3 ls s3://my-bucket/path/

// Delete s3://my-bucket/path/MyFile.txt
$ aws s3 rm s3://my-bucket/path/MyFile.txt

// Delete s3://my-bucket/path and all of its contents
$ aws s3 rm s3://my-bucket/path --recursive
```

