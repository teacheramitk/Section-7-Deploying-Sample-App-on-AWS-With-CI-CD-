# Section-7-Deploying-Sample-App-on-AWS-With-CI-CD


# Create CodeBuild Project - Lab

**Step 1.Goto AWS Console>Services>S3>Create bucket**

Provide the following details:
- Give Bucket name - sample-node-app-amit
- Unblock all public access & acknowledge

**Step 2. Open sample-node-app-amit bucket>Create Folder**
- Folder name - devbuild
- Encryption - Disable

Click on Create Folder

**Step 3.Goto AWS Console>Developers Tools>CodeBuild>Build projects>Create build project**

Provide the following details:
- Project name - first-cd-project
- In Source Tab
  - Source provider - AWS CodeCommit
  - Repository - Sample-Node_App
  - Reference Type select “Branch” and Branch as “Master”

- In Environment Tab
  - Environment image - Managed image
  - Operating system - Amazon Linux 2
  - Runtime(s) - Standard
  - Image - aws/codebuild/amazonlinux2-x86_64-standard:3.0
  - Image version - Always use the latest image for this version
  - Service role - New service role
  
- In Buildspec section:
  - Build specifications - select “Use a buildspec file”

- Artifacts Section:
  - Type - Amazon S3
  - Bucket name - Sample-Node_App-amit
  - Path - devbuilds/
  - Artifacts packaging - Select Zip
  - Select “Disable artifacts encryption”

- Logs section
  - CloudWatch - Select Cloudwatch logs
  - Group name - “cb-project-s3”
  - Stream name - "cb-project-s3"

Click on Create Build Project

**Step 4.Project is successfully created**

Click on Start build

**Step 5.Provide Build configuration:**
- Project - cb-project-s3
- Build type - Single build
- Timeout - 0 Hours 5 Minutes

Click on Start Build


**Step 6. Check the status of all the following phases in phase details-**
```sh
SUBMITTED - Succeeded
QUEUED    - Succeeded
PROVISIONING - Succeeded
DOWNLOAD_SOURCE - Failed      YAML_FILE_ERROR:YAML file does not exist
FINALIZING      - Succeeded
COMPLETED       - Succeeded
```


**Step 7. Goto IAM>Roles>codebuild-first-cd-project-service-role**
- Click on Permissions
  - There are two policies 
  - See and compare both the policies

**Step 8. AWS Management Console>CloudWatch>CloudWatch>CloudWatch Logs>Log groups>first-cb-project**
- Click on current time Log stream
  - See the log events
  
**Step 9. AWS Management Console>CloudWatch> Goto Metrics and validate metrices**

# End of Lab




# AWS CodeBuild : Build Sample App from CodeCommit - Lab

**Step 1. Goto AWS Management Console>Services>Developers Tools>CodeBuild>first-cd-project>Start build**

Provide the following details:
- Build configuration
  - built type - single build
  - Timeout - 0 Hours 5 Minutes
Click on Start Build

**Step 2.Goto Phase details to see the ongoing build**
- Download source - build fails due to version

To remove this error version need to changed

**Step 3. Now Open Terminal in Visual Studio Code**
```sh
$ git status 
$ git add .
$ git commit -m "changed node version to 12"
$ git push
```
**Step 4. Goto AWS Management Console>Services>Developers Tools>CodeBuild>first-cd-project>Start Build**

Provide the following details:
- Build configuration 
  - Timeout - 0 Hours 5 Minutes
- Source version shows node version to 12
- Click on Start Build

**Step 5. See Build Status>Phase details**
- Monitor the build phases

**Step 6. Now Goto S3>sample-node-app-amit>devbuild/>first-cd-project**

**Step 7. Goto Developers Tools>CodeBuild>Build projects>first-cd-project>Edit>Artifacts**
- See that semantic versioning is checked
Click on Cancel & than click on Discard

**Step 8. Goto AWS Management Console>CloudWatch>CloudWatch>CloudWatch Logs>Log groups>first-cd-project**
- Click on latest time logstream
- see the log events

# End of Lab

# AWS CodeBuild : Build Sample App from S3- Lab

**Step 1.Open Terminal in Visual Studio Code**
- Zip the Sample-Node_App folder

```
$ zip -r Sample-Node_App.zip . -x "_MACOSX"**
```
**Step 2.Goto AWS Management Console>Services>S3>Buckets>teacheramitk>Add files**
- Select Sample-Node_App.zip and Open
- Click on Upload

**Step 3.Click on Sample-Node_App.zip**
- In Details Copy S3 URI

**Step 4. Goto AWS Management Console>Services>Developers Tools>CodeBuild>Build Projects>Create Build Project**

Provide the following details:
- Provide Project Configuration
  - project name - cb-project-s3

- In Source Tab
  - Source provider - Amazon S3
  - Bucket - teacheramitk
  - S3-object-key - Sample-Node_App.zip
  
- In Environment Tab
  - Environment image - Managed image
  - Operating system - Amazon Linux 2
  - Runtime(s) - Standard
  - Image - aws/codebuild/amazonlinux2-x86_64-standard:3.0
  - Image version - Always use the latest image for this version
  - Service role - Existing service role
  - Role ARN - arn:aws:iam::xxxxrole/sevice role/codebuild-first-cd-project-service-role
  
- In Buildspec section: 
  - Build specifications - select “Use a buildspec file”

- Artifacts Section:
  - Type - Amazon S3
  - Bucket name - Sample-Node_App-amit
  - Path - devbuild/
  - Artifacts packaging - Select Zip
  - Select “Disable artifacts encryption”
- Logs section
  - CloudWatch - Select Cloudwatch logs
  - Group name - “cb-project-s3”
  - Stream name - "cb-project-s3"  

Click on Create Build Project

**Step 5.Project is successfully created**
- Click on Start build

**Step 6.Provide Build configuration:**
  - Project - cb-project-s3
  - Build type - Single build  
  - Timeout - 0 Hours 5 Minutes

Click on Start Build
 
**Step 7. Check the status of all phases in phase details-**
```sh
SUBMITTED
QUEUED
PROVISIONING
DOWNLOAD_SOURCE
INSTALL
PRE_BUILD
BUILD
POST_BUILD
FINALIZING
COMPLETED
```
**Step 8.Goto AWS Management Console>CloudWatch>CloudWatch>CloudWatch Logs>Log groups>cb-project-s3**
- Click on latest Log stream
- See the log events

**Step 9. Now Goto S3>sample-node-app-amit>devbuild/>cb-project-s3**
- See the details

# End of Lab

# Create EC2 Instance for AWS CodeDeploy - Lab

# Follow this Link for installing code-deploy-agent:- https://docs.aws.amazon.com/codedeploy/latest/userguide/codedeploy-agent-operations-install-linux.html

**Step 1.AWS Console>All services>EC2>Launch Instance**

**Step 2.Select  Amazon Linux2 AMI**

**Step 3. Choose Instance type t2 micro**
- Click Next:Configure Instance

**Step 4.Leave everything default beside IAM Role and User Data Section**
- IAM Role - EC2S3FullAccess
- User Data>As Text:See below
```sh
#!/bin/bash
$ yum update -y
$ yum install ruby -y
$ yum install wget -y
$ yum install nmap-ncat -y
$ cd /home/ec2-user
$ wget https://aws-codedeploy-ap-south-1.s3.ap-south-1.amazonaws.com/latest/install
$ chmod +x ./install
$ ./install auto
$ service codedeploy-agent status
```
- Click on Next:Add Storage

**Step 5.Keep Default**
- Click on Next:Add Tags

**Step 6.Give key as Name and value as node-server**
- Click on Next:Configure Security Group

**Step 7.Configure Security Group:**
- Select Security Group with:
  - HTTP:80 Source-0.0.0.0/0
  - TCP:22  Source-0.0.0.0/0
  - Custom TCP Rule:3000 Source-0.0.0.0/0

- Now Click on Review and Launch


**Step 8.Click on Launch>Select existing Key pair**
- Now Click on Launch Instances

**Step 9.Goto EC2 Dashboard**
- Select node-server>Actions>Connect
- Type the following command

```
$ sudo service codedeploy-agent status
#You Will see this message "The AWS Codedeploy agent is running as PID xxxx"
```


# End of lab

# Deploying Sample App on Single EC2 : Part 1 - Lab

**Step 1.Goto AWS Console>Developers Tools>CodeDeploy>Applications>Create Application**

In Application configuration give the following details:
- Give Application name:- cd-app
- Compute platform - Ec2/On-premises

Click on Create application

**Step 2.Goto Deployment groups>Create deployment group**
- Give the name “ cd-app-dg” in the deployment group name section

**Step 3.Service Role-Create service role as following**
- Goto AWS Console>IAM>Roles>Create Role
  - Select AWS service>CodeDeploy>CodeDeploy

- Click on Next:Permissions>Next:Tags>Next:Review

- Give name - node-app-code-deploy-service-role

- Click on Create role

**Step 4.Goto Step 2 and give service role**
- Refresh and Select node-app-code-deploy-service-role

**Step 5.In Deployment type choose - In-Place**

**Step 6.Select Amazon Ec2 instances in Environment configuration**
- In tags select as following
  - key - Name 
  - value - node-server

See the 1 Matching instance for above tags

**Step 7. Agent configuration with AWS Systems Manager**
- Never

**Step 8. Deployment settings - CodeDeployDefault:AllAtOnce**

**Step 9.Load Balancer - deselect it**

Click on Create deployment group

**Step 10.Goto IAM>Roles>node-app-code-deploy-service-role>Permissions**
- See the Policy summary

**Step 11.Now Click on Create Deployment**
- See deployment settings 
- Revision type - My application is stored in S3 
- Revision location 
   - S3>Buckets>sample-node-app>devbuild/>first-cd-project
   - To make first-cd-project public 
     - goto Object actions>Make Public
   - Copy S3 URI 

**Step 12.Click on Create deployment**
- Deployment Failed - Due to No Appsec.yml file

**Step 13.Open terminal in Visual Studio Code**
Run the following commands:
```sh
$ cp ../appsec.yml .
$ git status
$ git add .
$ git commit -m "adding appspec.yml"
$ git push
```
**Step 14. Goto AWS Console>All services>Code Commit>Repositories>Sample-Node-App**
- See that the appsec.yml file has been copied

# End Of lab
     


# Deploying Sample App on Single EC2 : Part 2 - Lab

**Step 1.Goto S3>Buckets>sample-node-app-amit>Edit Bucket Versioning**
- Select Enable 
- Click on Save changes

**Step 2.Developers Tools>CodeBuild>Build projects>first-cd-project**
- Click on Start Build

**Step 3.In Build Configuration**
- Timeout - 0 Hours 5 Minutes 
- See the Source version comment "adding appsec.yml"

Click on Start Build

**Step 4. See Build Status in the following Phase details**
```sh
SUBMITTED       - Succeeded
QUEUED          - Succeeded
PROVISIONING    - Succeeded
DOWNLOAD_SOURCE - Succeeded
INSTALL         - Succeeded
PRE_BUILD       - Succeeded
BUILD           - Succeeded
POST_BUILD      - Succeeded
FINALIZING      - Succeeded
COMPLETED       - Succeeded
```

**Step 5. Now Goto S3>sample-node-app-amit>devbuild/>first-cd-project**
- See both the Versions
- To Make it public goto Object actions>Make Public
- Copy S3 URI

**Step 6.Goto AWS Console>Developers Tools>CodeDeploy>Applications>cd-app**
- Choose cd-app-sg
- Click on Create deployment

**Step 7.In Deployment settings**
- Revision type - My application is stored in S3 
- Revision location 
   - s3://sample-node-app/devbuild/first-cd-project
   - Revision file type - .zip

Click on Create Deployment
- Deployment failed - Error code ScriptMissing

**Step 8.Open terminal in Visual Studio Code**
```sh
$ cp -r ../deploy_scripts .
$ git status
$ git add .
$ git commit -m "added deployment scripts"
$ git push
```



# Deploying Sample App on Single EC2 : Part 3 - Lab

**Step 1.Goto AWS Console>All services>Code Commit>Repositories>Sample-Node-App**
- See deploy_scripts have been copied

**Step 2.Developers Tools>CodeBuild>Build projects>first-cd-project**
- Click on Retry Build

**Step 3. See Build Status in the following Phase details**
```sh
SUBMITTED       - Succeeded
QUEUED          - Succeeded
PROVISIONING    - Succeeded
DOWNLOAD_SOURCE - Succeeded
INSTALL         - Succeeded
PRE_BUILD       - Succeeded
BUILD           - Succeeded
POST_BUILD      - Succeeded
FINALIZING      - Succeeded
COMPLETED       - Succeeded
```
Build is done

**Step 4.Now Goto S3>sample-node-app-amit>devbuild/>first-cd-project**
- Now See there are 3 Versions
- Make latest public :Object actions>Make Public
- Copy S3 URI

**Step 5.Goto AWS Console>Developers Tools>CodeDeploy>Applications>cd-app**
- Choose cd-app-sg
- Click on Create deployment

**Step 6.In Deployment settings**
- Revision type - My application is stored in S3 
- Revision location 
   - s3://sample-node-app/devbuild/first-cd-project
   - Revision file type - .zip

Click on Create Deployment

**Step 7.See the Deployment status>Deployment lifecycle events**
```sh
ApplicationStop - Succeeded
DownloadBundle  - Succeeded
BeforeInstall   - Succeeded
Install         - Succeeded
AfterInstall    - Succeeded
ApplicationStart - Succeeded
Validateservice  - Succeeded
```
Deployment has been done successfully

**Step 8.Goto Ec2 Dashboard>copy Public Ip>Paste in Browser e.g 13.222.158.20:3000**
- See the Application Running

# End of Lab


# Validate CodeDeploy files/concepts on EC2 Server -Lab

**Step 1.Open Appsec.yml file in Visual Studio code**

**Step 2.Goto Ec2 dashboard>node-server>Actios>Connect>connect**

**Step 3.Goto https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-structure-hooks.html**

**Step 4.Goto Connected instance node-server**
- Run the following commands
```sh
$ cd /opt/codedeploy-agent/deployment-root
$ ls
$ cd deployment-instructions
$ ls
$ cat xxxxxxxxxxx_lastsuccessful_install
$ cd /opt/codedeploy-agent/deployment-root/xxxxx-xxxx-
$ ls
$ cd deployment-archive/
$ ls
$ cd /home/ec2-user
$ ls
```
# End of lab

# CodeDeploy Deployment on Auto Scaling - Lab

**Step1-AWS Console>All Services>EC2>Auto Scaling>Launch Configuration>Create Launch Configuration**

**Step2- Provide following details:**
- Give name as “lc-cd” 
- select AMI(Copy it from Ec2 Launch Instance)
- Select instance type t2.micro 
- IAM Instance profile - Ec2S3FullAccess
- In Advanced details>as text>Provide User data:
```sh
#!/bin/bash
$ yum update -y
$ yum install ruby -y
$ yum install wget -y
$ yum install nmap-ncat -y
$ cd /home/ec2-user
$ wget https://aws-codedeploy-ap-south-1.s3.ap-south-1.amazonaws.com/latest/install
$ chmod +x ./install
$ ./install auto
$ service codedeploy-agent status
```
- keep storage as default
- Select security group with All port open as of now 
- Choose existing key pair 

Click on Create launch configuration

**Step 3. Launch configuration is created**

**Step 4. Select launch configuration “lc-cd ”** 
- Click Actions>create auto scaling group

**Step 5. Give auto scaling group name - asg-cd**
- See launch-configuration and click on Next

**Step 6. Select default vpc and all three subnets**
- Click Next

**Step 7.Configure advanced options** 
- Select No Load Balancer 
- Select Health checks with ELB
  - Health check grace period - 180 sec

Click Next

**Step 8 Keep Group size as following**
- Desired - 1 
- Minimum - 1
- Maximum - 1

**Step 9 Select Scaling policies-None**
Click on Next

**Step 10. In Add Notifications Click Next**

**Step 11. In Add tags Click Next**

**Step 12. See Review and Click on Create Auto Scaling Group**

**Step 13. Goto EC2 dashboard and see the newly created instance by ASG**

**Step 14. AWS Console>Developers Tools>CodeDeploy>Applications>cd-app>Create Deployment Group**

**Step 15.Provide details:**
- Deployment group name - cd-app-asg
- Service role - use existing 
- Deployment Type - In-Place
- Environment configuration - Select Auto Scaling group asg-cd
- Deployment settings - CodeDeployDefault:AllAtOnce
- Load Balancer - Deselect the Load Balancer 
 
Click Create deployment group
	
**Step 16. Click on Create Deployment**
- keep Deployment group as it is
- Revision type - My application is stored in Amazon S3
- Revision location - Paste the S3 URI
- Revision file type - .zip

Click on Create Deployment

**Step 17.Monitor Deployment LifeCycle Events**
```sh
ApplicationStop
DownloadBundle
BeforeInstall
Install
AfterInstall
ApplicationStart
Validateservice
```
**Step 18. Copy the Instance Ip address and paste it in browser**
- See that the application is running

**Step 19.AWS Console>Developers>CodeDeploy>Applications>cd-app>cd-app-asg>Edit**
- After validating click on cancel and discard

**Step 20.Ec2>Auto Scaling group>Instance Management>Lifecycle hooks**
- Select codedeploy hook>Actions>Edit lifecycle hook

**Step 21.Ec2>Auto Scaling groups>asg-cd>Edit Group size**

Put the following value:
- Desired - 2
- Minimum - 2
- Maximum - 2

Click on Update

**Step 22.Ec2>Auto Scaling Group>asg-cd>Instances**
- See the lifecycle behaviour pending>pending-wait>pending-wait:proceed>Inservice

**Step 23.Click the new Instance and copy & paste the Public IP address in browser to see it running**
- e.g.-13.222.125.52:3000

# End of lab



# CodeDeploy in-place Deployment with ALB - Lab

**Step 1. Go to AWS Console>All Services>EC2>Load Balancing>Target Groups>Create Target Group**

**Step 2. Specify group details as following**

- Select Instances in target type
- Target group name - tg-cd
- Protocol-HTTP,Port-3000
- VPC-Default
- Protocol version
- Health Checks
  - Health check protocol- HTTP
  - Health check path- /

Now Click on Next

**Step 3. Register targets**
- Select the available instances
- Ports for the selected instances-3000
- Click on "Include as pending below"

Now Click on Create target group

**Step 4. "tg-cd" Target Group has been created**

**Step 5. Go to AWS Console>All Services>EC2>Load Balancing>Load Balancers>Create load balancer**

**Step 6. Click on Create in Application Load Balancer**

**Step 7. Configure Load Balancer as following**

In Basic Configuration
- Name- alb-cd
- Scheme- internet-facing
- Ip address type- ipv4
- Listeners-Protocol-HTTP,Port-80
- Availability Zones - VPC-Default - Availability Zones-Select all Zones

Click Next:Configure Security Settings

**Step 8.Configure Security Groups**

- Select security group

Click on Next:Configure Routing

**Step 9.Configure Routing**
- Target group- Select Existing target group Name - tg-cd

Click on Register Targets

**Step 10.Click on Next:Review>Click on Create>Click on Close**

**Step 11.EC2>Target groups >tg-cd>edit Attributes**
- change Deregistration delay - 60 seconds

**Step 12.Open Terminal in Visual Studio Code**
```sh
$ git status
$ git add .
$ git commit -a
$ git commit -m "changed index page's color"
$ git push
```
**Step 13.Developers Tools>CodeBuild>Build projects>first-cd-project**
- Click on Retry build

**Step 14.Copy the DNS of Load Balancer and paste it in browser**
- See that it ALB is sending traffic

**Step 15. After Build completion goto S3>sample-node-app-amit>devbuild/>first-cd-project**
- Click on Object actions>Make Public>make public>Exit
- Copy S3 URI

**Step 16. Goto AWS Console>Developers Tools>CodeDeploy>Applications>cd-app>cd-app-asg**
- Click on Edit
- Provide the following details:
  - Deployment group name - cd-app-asg-alb
  - Deployment type - In-place
  - Environment configuration - keep Selected Auto Scaling groups 
  - Deployment settings - CodeDeployDefault:AllAtOnce
  - Load Balancer - Enable Load Balancing with Application load Balancer
    - Choose target group
	
Click on Save changes

**Step 17. Click on Create Deployment**
- No change in Deployment group 
- Revision type - My application is stored in Amazon S3
- Revision location - Paste the S3 URI

Click on Create Deployment

**Step 18.Monitor Life Cycle Events**
```sh
BeforeBlockTraffic
BlockTraffic
AfterBlockTraffic
ApplicationStop
DownloadBundle
BeforeInstall
Install
AfterInstall
ApplicationStart
Validateservice
BeforeAllowTraffic
AllowTraffic
AfterAllowTraffic
```
**Step 18.Open DNS of load balancer in Browser**
- See that it is serving the traffic

# End of Lab


# CodeDeploy Blue Green Deployment - Lab

**Step 1.AWS Console>Developers>CodeDeploy>Applications>cd-app>cd-app-asg-alb>cd-app-asg-alb>edit**

Provide the following details:
  - Deployment type - Blue/Green
  - Environment configuration - Select Automatically copy Amazon EC2 Auto Scaling group
  - In Deployment settings
    - Terminate the original instances in the deployment group running
      - 0 Days 0 Hours 0 Minutes
  
Click on Save changes

**Step 2. Open Terminal in Visual Studio Code**
- Change color to Green in pages>index.ejs
- Run the following commands
```sh
$ git status
$ git add . 
$ git commit -m "changed index color to green"
$ git push
```
**Step 2.AWS Management Console>Services>Developers Tools>CodeBuild>Build Projects>first-cd-project>Start Build**

Provide the following details:
- Change Timeout in build configurationto 0 hours 5 minutes
- In Source Tab - source version shows "changed index color to green"

Click on Start Build

**Step 3.After Build completion goto S3>sample-node-app-amit>devbuild/>first-cd-project>versions**
- Check the latest version
- Click on Object actions>Make Public>make public>Exit
- Copy S3 URI

**Step 4.Goto AWS Console>Developers Tools>CodeDeploy>Applications>cd-app>cd-app-asg-alb**
- Click on Create deployment
- Copy Revision location
- Revision file type - .zip

Click on Create deployment

**Step 5.Ec2>Auto Scaling groups>Instance management>Instances**
- See the behaviour of instances

**Step 6.EC2>Load Balancer>target groups>tg-cd**
- See the behaviour of 2 newer instances and 2 older instances(will be deleted)

**Step 7.CodeDeploy>Applications>cd-app>cd-app-asg-alb**
- See status in Deployment lifecycle events
- 2 Showing original and 2 replacement

**Step 8.Click to see Events of 1 Replacement Instance**

```sh
BeforeBlockTraffic
BlockTraffic
AfterBlockTraffic
ApplicationStop
DownloadBundle
BeforeInstall
Install
AfterInstall
ApplicationStart
Validateservice
BeforeAllowTraffic
AllowTraffic
AfterAllowTraffic
```
**Step 9. Deployment complete and now 4 instances are available**
- 2 newer
- 2 older

**Step 10. Check the DNS of load balancer and refresh it 2 times**
- You can see blue and green color as 2 original and 2 replacement are running now

**Step 11.Now events section in Oiginal instance**

Following 3 hooks will be faced during deregistration
- BeforeBlockTraffic
- BlockTraffic
- AfterBlockTraffic

**Step 12. Check the DNS of load balancer and refresh it**
- Now only 2 instances are serving the traffic
- You can only see green as original blue are terminated

**Step 13. See the deployment status**
- Original instances termination is in process

**Step 14.AWS Console>Developers Tools>CodeDeploy>Applications>cd-app>cd-app-asg-alb** 
- Edit deployment group to see the auto scaling group
- See "asg-cd" is replaced by new Auto scaling group "CodeDeploy-xxx" automatically


**Step 15. Check the DNS of load balancer and refresh**
- Only Green color instances are running

**Step 16. See the deployment status**

Traffic shifting process
- Original - 0
- Replacement - 2

# End of lab


# Codedeploy Deployment Configuration - Lab

**Step 1.Aws Console>Ec2>Auto Scaling groups>asg-cd**
- Edit group size as following-
  - Desired - 4
  - Minimum - 4
  - Maximum - 4
 
**Step 2.Ec2>Auto Scaling groups>asg-cd>Instance management**
- See new instances are launching 

**Step 3.Ec2>Target groups>tg-cd>targets**
- See new instances are launching

**Step 4.Open Terminal in Visual Studio**
- Change color to Green in pages>index.ejs
- Run the following commands
```sh
$ git status
$ git add . 
$ git commit -m "another change in color of index"
$ git push
```

**Step 5. Developers Tools>CodeBuild>Build projects>first-cd-project**
Click on Retry Build

**Step 6.Goto S3>sample-node-app-amit>devbuild/>first-cd-project**
- Copy S3 URI

**Step 7.Keep monitoring Auto Scaling groups>Instances**
- Wait for 4 instances to be healthy

**Step 8.See Build is succeeded**

**Step 9.Goto S3>sample-node-app-amit>devbuild/>first-cd-project**

- Click on Object actions>Make Public>make public>Exit
- Copy S3 URI

**Step 10. Copy ALB DNS in the browser and Hit refresh 4 times**
- See that 4 instances are serving the traffic now

**Step 11.Goto AWS Console>Developers Tools>CodeDeploy>Applications>cd-app>cd-app-asg-alb**

Click on Create deployment
- My application is stored in S3
- Copy Revision location from S3
- Revision file type - .zip

In Deployment group overrides configuration
- Select CodeDeploy:HalfAtATime

Click on Create deployment

**Step 12.See in Deployment lifecycle events**
- 2 are pending
- 2 are in progress

**Step 13. Ec2>Target groups**
- 2 are healthy
- 2 are deregistering

**Step 14.Check the DNS of load balancer and refresh**
- It is serving only 2 instances

**Step 15.See in Deployment lifecycle events**
- 2 are succeeded
- 2 are in progress

**Step 16. Ec2>Target groups**
- All 4 are healthy

**Step 17.Check the DNS of load balancer and refresh 4 times**
 
# End of lab
























