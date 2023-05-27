# Deploy a Python/Flask/Postgresql app with Cloudformation (Cfn) and Cfn helper scripts.

Use this yaml template to deploy and provision a Flask app which uses a local postgres database instance.

## Requirements
* An AWS account
* AWS cli
* Git
* A key pair for use with an Amazon Elastic Compute Cloud (EC2)

## Cloudformation
Creates the EC2 instance and the components that supports the instance: VPC, routes, subnets, internet gateway, security groups, etc.

## Metadata - AWS::CloudFormation::Init key
cfn-init reads and create the resources in the metadata section: packages, files, services, etc.

## cfn-init
This helper script reads the metadata data section and processes the configuration in the following order:
* packages
* groups
* users
* sources
* files
* commands
* and finally services

```
# Ubuntu
/usr/local/bin/cfn-init -s ${AWS::StackName} -r WebServerInstance --region ${AWS::Region} -c setup
```

Use configset to specify the order in which the config keys should be processed.

## cfn-hup
This helper script detects changes in resource metadata and runs user-specified actions when a change is detected.

```
# Enable and start cfn-hup
systemctl start cfn-hup && systemctl enable cfn-hup  
```
## cfn-signal
Instructs Cloudformation that the stack creation has been completed and all the services are running. CloudFormation sets the status of the stack as CREATE_COMPLETE after it successfully creates all the resources.

```
/usr/local/bin/cfn-signal -e $? --stack ${AWS::StackId} --resource WebServerInstance --region ${AWS::Region}
```

## CreationPolicy
Use the cfn-signal script in conjunction with a CreationPolicy attribute or an Auto Scaling group with a WaitOnResourceSignals update policy. 

```
CreationPolicy:
      ResourceSignal:
        Count: 1
        Timeout: PT5M
```

Associate the CreationPolicy attribute with a resource to prevent its status from reaching create complete until CloudFormation receives a specified number of success signals or the timeout period is exceeded.

## Cloud init output

Check the /var/log/cloud-init-output.log for information about cfn-init and cnf-signal.

cloud-init-output.log
```
Successfully installed aws-cfn-bootstrap-2.0 chevron-0.14.0 docutils-0.20.1 lockfile-0.12.2 python-daemon-2.1.2
+ sudo ln -s /usr/local/init/ubuntu/cfn-hup /etc/init.d/cfn-hup
+ /usr/local/bin/cfn-init -s cf-init -r WebServerInstance --region us-east-2 -c setup
+ /usr/local/bin/cfn-signal -e 0 --stack arn:aws:cloudformation:us-east-2:187871168870:stack/cf-init/e94d7aa0-fb5e-11ed-bb58-062dee8e8c61 --resource WebServerInstance --region us-east-2
Cloud-init v. 23.1.2-0ubuntu0~20.04.1 running 'modules:final' at Fri, 26 May 2023 00:48:34 +0000. Up 21.59 seconds.
Cloud-init v. 23.1.2-0ubuntu0~20.04.1 finished at Fri, 26 May 2023 00:49:53 +0000. Datasource DataSourceEc2Local.  Up 100.86 seconds

```

## Usage
```
 aws cloudformation deploy --stack-name cf-init --template-file cf-init.yaml
```