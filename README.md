# GitHub Actions with CloudFormation

## Motivation

Test CloudFormation with GitHub Actions.
The goal is to deploy

- VPC
- subnet

in AWS (because it is free!).


## Overview

1. Create an IAM user for management.
   - Set up authentication app for single sign-on.
   - Log in from aws cli.
   
2. Edit CloudFormation template file.

3. Test if CloudFormation works from aws cli.

4. Edit GitHub Action deployment file
   - set up a new GitHub repo. 

5. Create an IAM user for the deployment.

<!-- ------------------------------  -->

## Procedure

### Create an IAM user for management

It is highly recommended not to use root user for testing.
Do the following:

- Create AWS Organization and invite your account. It does not cost. 
- Create an admin group at IAM Identity Center
- Create a user at IAm Identity Center and add it to teh admin group
- Attach AdministratorRole to the group

To ease the experiment

- Download Google Authenticator to use for MFA (=multi- factor authentication)
- Set up single sign on.

Log in using single sign on.
```
aws configure sso
```

Check if it works.
```
aws s3 ls --profile XXXX
```

Set an environment variable.
```
export AWS_PROFILE=XXXX
```

Check if it works (without "--profile XXXX")
```
aws s3 ls
```

For the security remove AWS Access Key and AWS Secret Access Key from
root account


### Prepare CloudFormation template file

We will deploy VPC and Subnet.

```
WSTemplateFormatVersion: "2010-09-09"
Description:
  "Create a network to test CloudFormation and GitHub Actions"

  #--------------------------------------
Parameters:
  VPCName:
    Default: VPC-Glenn
    Type: String
  SubnetName:
    Default: Subnet-VPC-Glenn
    Type: String

Resources:
  #--------------------------------------
  # VPC
  #--------------------------------------
  GlennVPC:
    Type: "AWS::EC2::VPC"
    Properties:
      CidrBlock: 10.1.0.0/16
      Tags:
        - Key: Name
          Value: !Ref VPCName
  #--------------------------------------
  # Subnet
  #--------------------------------------
  GlennSubnet01:
    Type: "AWS::EC2::Subnet"
    Properties:
      VpcId:
        Ref: GlennVPC
      AvailabilityZone: eu-west-1a
      CidrBlock: 10.1.1.0/24
      Tags:
        - Key: Name
          Value: !Ref SubnetName
#--------------------------------------
# Outputs
#--------------------------------------
Outputs:
  VPCId:
    Value: !Ref GlennVPC
    Description: "The Vpc Id Created By This Stack."
  SubnetId:
    Value: !Ref GlennSubnet01
    Description: "The Subnet Id Created By This Stack."
#--------------------------------------
```

### Test CloudFormation template with aws cli

First we will test if the template file we wrote above
works from terminal.

Check if the syntax is correct.

```
aws cloudformation validate-template --template-body file://vpc-1.yml
{
    "Parameters": [
        {
            "ParameterKey": "SubnetName",
            "DefaultValue": "Subnet-VPC-Glenn",
            "NoEcho": false
        },
        {
            "ParameterKey": "VPCName",
            "DefaultValue": "VPC-Glenn",
            "NoEcho": false
        }
    ],
    "Description": "Create a network to test CloudFormation and GitHub Actions"
}

```
Looks all right.

Then create a stack
```
aws cloudformation create-stack --stack-name StackOfGlenn \
    --template-body file://vpc-1.yml
```

Check if it is there.

```
aws cloudformation describe-stacks
```

Check if it is there on AWS Management Console.

![stack-1](./images/stack-1.png)

Check if VPC and Subnet are there.

![vpc-subnet-1](./images/vpc-subnet-1.png)

All right.

Delete stack.
```
aws cloudformation delete-stack --stack-name StackOfGlenn
```
Check if the vpc is gone.

![vpc-2](./images/vpc-2.png)

All right.


### Prepare GitHub Actions workflow file. 

Create ```./.github/workflows/deploy.yml```.

Set up GitHub secrets here.

![github-secrets](./images/github-secrets.png)

Then commit the files. Check if GitHub Actions started and completed.

![github-secrets](./images/github-actions-completed.png)

All right.

Check if the resouces are created.

![vpc-3](./images/vpc-3.png)

All right. Remove all resources.

```
aws cloudformation delete-stack --stack-name StackOfGlenn
```

Make sure to ***Disable** GitHub Actions.

![disable-actions](./images/disable-actions.png)

Otherwise, when you update README.md in the future, you will create resources by accdent.

Push README.md once again and make sure no resource will be created. 


<!-- ------------------------------  -->

# END

<!-- ####################  -->
