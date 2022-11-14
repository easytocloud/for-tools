# for-tools

A set of tools for use with AWS CLI to iterate over multiple AWS accounts and run AWS CLI commands in each

# Installation

Install the for-tools with brew:

```
brew tap easytocloud/tap
brew install aws-for-tools
```

# IAM Users, roles and profiles

The tool assumes that long-term credentials for IAM users are stored in ~/.aws/credentials.
Roles are profiles without an acces_key and secret_acces_key - best kept in ~/.aws/config - that refer to an IAM user in ~/.aws/credentials.

Essentially, the command prodivded as argument is run once for each {user, role or profile} by setting AWS_PROFILE and calling the command.

> TIP: When multiple commands need to run, create a script for it. AWS_PROFILE is exported and as such available in your script.

## for-iamusers

for-iamusers takes the users as defined in your ~/.aws/credentials file and iterates over all of them.

```
for-iamusers aws s3 ls
```

## for-iamroles

for-roles takes the roles as defined in your ~/.aws/config file and iterates over all roles that can be assumed by the current user in ${AWS_PROFILE}

for-roles fails if the current value of AWS_PROFILE already refers to a role.
```
for-iamroles aws s3 ls
```

Fun fact: the two for-iams can be nested to assume all roles for all users:
```
for-iamusers for-iamroles aws s3 ls
```

## for-profiles

for-profiles iterates over all profiles (users or roles) in your ~/.aws/config and ~/.aws/credentials files. Note: no effort is made to deduplicate.
```
for-profiles aws ec2 describe-instances
```

# AWS Organizations aware

> This only applies if your AWS accounts are organized in an AWS Organization.

Issuing the command from a profile in the main account in the Organization that has sufficient permissions, you can use
```
for-org_accts --role OrganizationAccountAccessRole aws iam list-users
```
where the role OrganizationAccountAccessRole is a role that can be assumed by the current profile in each member account of the organization.

The for-org_accts work slightly different than the profile-based for-commands above. For profile-based commands, AWS_PROFILE is set to the required values before calling the command.
As for-org_accts doesn't rely on any role-profiles being defined, it calls `aws sts assume-role` with the provided rolename in all accounts. 
The result of that is stored in environment variables that have a higher priority to the aws cli than AWS_PROFILE. 
```
AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN
```
AWS_PROFILE doesn't change throughout the sequence. 
Should you issue a script, use `aws sts get-caller-identity` to find the current 'profile'.
