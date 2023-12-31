# Deployment 5

<p align="center"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/Graphic%20Design-7.png"></p>

## Purpose

##### The purpose of this deployment was to continue evolving our appliction infrastructure and our deployment methods. In Deployment 4, we moved away from managed services & default infrastructure configurations. 

##### We set up our infrastructure on a private network and used tools like Nginx & CloudWatch. The private network gave us more control over traffic coming into our infrastructure. Nginx (used as a reverse proxy) & CloudWatch allowed us to control and monitor ingress traffic to our application. 

##### _These benifits came at a price of a lot of manual interaction with the AWS console to configure & deploy our infrastructure. More manual labor = more potential human error + more costs + more time to complete a task._

#### In Deployment 5, we use Terraform to solve those problems. Terraform is an Infrastructure as Code tool that allows us to configure and deploy our application infrastructure using configuration files. This makes our deployments...
* #### _Faster, because after an initial configuration, that infrastructure can be redeployed as fast as it takes to run a script._
* #### _More flexible, because our infrastructre is being built with editable files_
* #### _More scalable through the use of "main" configuration templates with editable variable files that store the details of our infrastructure like virtual machine sizes, ingress/egress traffic control policys, availabiliy zones, ect..._

## Issues

### _1. Terraform Build Fails_
<p align="left"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/dp5error.PNG" width="500"></p>

* #### _Problem: CIDR blocks were not referenced correctly in my Terraform variables file and it caused a few build failures_
* #### _Solution: CIDR blocked should be referenced as strings inside of an array_
* #### _Takeaway:Try to use updated methods from the tools documentation instead of online tutorials_

### _2. AWS Credentials Issue_
<p align="center"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/awscredentials.png"></p>

* ##### _Problem: No built in way to set AWS credentials when doing a automated installation. Tried doing a script to make credentials file, it worked but theres a simpler way_
* #### _Solution: Use an IAM Role that give our EC2 instance permissions to list instance details. Attach the role to our instance and then we wont need credentials_
* #### _Takeaway: Remember the power of IAM roles and policies and how they can solve permission related problems_

### _3. Dynamic IP Issue_
* #### _Problem: When our instance got disconnected, our hardcoded IP addresses would no longer work for our builds and SSH'ing into our servers_
* #### _Solution: Use AWS's `ec2 describe` method to pull the curreny IP address. This way we wont have to hard code it and we'll always know its the one attached to our current machine_
* #### _Takeaway: Try and use methods in our code that more flexible whey components of our systenm are dynamic, such as EC2 IP addresses that arent Elastic IPs_

## Steps
### _1. Plan Infrastructure Architecture and Deployment_
#### Where:
* ##### _AWS, Us-East-1 Region_

<p align="center"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/useast1.png"></p>

#### What:
* ##### _VPCs(1), Subnets(2), Availability Zones(2), EC2 Instances(2), Route Tables(1), Security Groups(1)_

<p align="center"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/Screenshot%202023-10-16%20at%2011.55.03%20AM.png" width="600"></p>

#### How: 
* ##### _ARCHITECTURE: We'll start the architcture by implementing a Virtual Private Cloud with an internet gateway to manage our network's ingress and egress traffic. Within this VPC, we'll put two subnets, each with their own Route Table. We'll deploy our Banking application and Jenkins server on separate 2 EC2 instances, each located in a different availability zones. Both instances will be protected by a Security Group that defines what kind of traffic is allowed._
<p align="center"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/DP5_Arch_Diagram.png"></p>

* ##### _DEPLOYMENT: We'll deploy our infrastructure by mapping our architectural requirements to our Terraform configuration files. Terraform uses a conecpt called "resource blocks" that basically standardizes a way to format and define our infrastructure requirements._
* <p align="center"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/ArchitectureDesign_to_TerraformMapping.drawio-3.png"></p>

### _2. Deploy Infrastructure Using Terraform_
#### To deploy our infrastructure using Terraform, we use 4 files.
* ##### _1. [`variables.tf`](https://github.com/djtoler/terraform_dp5/blob/main/variables.tf): This file will set up default variables for us to use throughout our Terraform config files_
<p align="center"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/Screenshot%202023-10-15%20at%201.04.23%20PM.png" width="500"></p>

* ##### _2. `x.tfvars`: This file will allow us to alter our variables from their default values when we choose to_
<p align="center"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/image6-3.png" width="500"></p>

* ##### _3. [`main.tf`](https://github.com/djtoler/terraform_dp5/blob/main/main.tf): This file is where we'll tell Terrafrom what cloud provider we want to use for our infrastructre deployment and which resources we want to deploy using that provider_
<p align="center"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/Screenshot%202023-10-15%20at%201.59.34%20PM.png" width="500"></p>

* ##### _4. [`tf-deploy.sh`](https://github.com/djtoler/terraform_dp5/blob/main/user_data_jenkins_python.sh): This file will allow us to run our terraform deployment from a shell script. Doing it from this script will give us additional automation capabilities in case we wanted to do other things after our infastructure finishes deploying & it saves us time because we dont have to run our Terrafrom deployment commands 1 by 1 in the command line._
<p align="center"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/Screenshot%202023-10-15%20at%202.15.57%20PM.png" width="500"></p>

#### _This is how our `tf_deploy.sh` works to deploy our infrastructure_
<p align="center"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/dp5_tf_auto.png"></p>

### _3. Configure Our Application & Jenkins Servers_
> #### _In our [`main.tf`](https://github.com/djtoler/terraform_dp5/blob/main/main.tf) file, we also included a [_user data script_](https://github.com/djtoler/terraform_dp5/blob/main/user_data_jenkins_python.sh) that will launch our 1st EC2 instance with Jenkins, Python & the AWS CLI already installed._
> #### _We do this for our 2nd EC2 instance also but with a [different user data script](https://github.com/djtoler/terraform_dp5/blob/main/user_data_python.sh) that just installs Python._

#### Both of these user data scripts use the `curl` bash command to download premade scripts for automating our installation tasks. We store them in this repo called [automated installation scripts](https://github.com/djtoler/automated_installation_scripts/tree/main).
> * ##### [Jenkins automated install script](https://github.com/djtoler/automated_installation_scripts/blob/main/auto-jenkins.sh)
> * ##### [Python automated install script](https://github.com/djtoler/automated_installation_scripts/blob/main/auto-python.sh)
> * ##### [AWS CLI automated install script](https://github.com/djtoler/automated_installation_scripts/blob/main/auto-aws_cli.sh)

> #### _Next, we finish setting up our Jenkins server by SSH'ing into it & switching to the Jenkins user. We download [this script](https://github.com/djtoler/terraform_dp5/blob/main/auto-instance_setup.sh), make it executable, and run it. This script will generate a key that we can use to SSH into our Banking application server from our Jenkins user on our Jenkins server._

### _4. Edit Setup & Jenkinsfiles_
* #### From our local machnine, we edit our `setup.sh` & `setup2.sh` files to setup up our banking application from GitHub repo where our source code is located.
```
# From the root directory of `ubuntu` user on our Banking application server
cd ~

# Store original files as backups
mv setup.sh setup_backup.sh
mv setup2.sh setup2_backup.sh

# Change the name of the repo in both files
sed -i 's|git clone https://github.com/kura-labs-org/c4_deployment-5.git|git clone https://github.com/djtoler/Deployment5_v1.git|' setup.sh
sed -i 's|git clone https://github.com/kura-labs-org/c4_deployment-5.git|git clone https://github.com/djtoler/Deployment5_v1.git|' setup2.sh

# Change the name of the folder that our script will 'cd' into
sed -i 's|cd c4_deployment-5|cd Deployment5_v1|' setup.sh
sed -i 's|cd c4_deployment-5|cd Deployment5_v1|' setup2.sh

```
* #### Next, we edit our `Jenkinsfilev1` & `Jenkinsfilev2` to download those 2 setup files we just edited. We'll also use the key we generated to SSH into our application server from our Jenkins server, as the Jenkins user, to download and run files for deploying our Banking application.

```
nano Jenkinsfilev1

# Remove the 'steps' block from the 'Deploy' stage in Jenkinsfilev1. Add these lines of code to reconfigure the pipeline to download our Banking application setup files

            steps {
                sh '''#!/bin/bash
                APPLICATION_IP=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=ec2_2" --query "Reservations[*].Instances[*].PublicIpAddress" --output text)
                ssh -i ~/.ssh/testkey8 -o StrictHostKeyChecking=no ubuntu@$APPLICATION_IP "
                    curl -O https://raw.githubusercontent.com/djtoler/Deployment5_v1/main/setup.sh
                    chmod +x setup.sh
                    ./setup.sh
                "
                '''
            }


nano Jenkinsfilev2

# Remove the 'steps' block from the 'Clean' stage in Jenkinsfilev2. Add these lines of code to reconfigure the pipeline to download and run our pkill.sh file. Gunicorn is what were using to run our Banking application. Running this file kills the current 'gunicorn' process that will be running on our Banking application server after the first build. It clears the way for our second build and redeployment.


        stage ('Clean') {
            steps {
                sh '''#!/bin/bash
                APPLICATION_IP=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=ec2_2" --query "Reservations[*].Instances[*].PublicIpAddress" --output text)
                ssh -i ~/.ssh/testkey8 -o StrictHostKeyChecking=no ubuntu@$APPLICATION_IP "
                    cd Deployment5_v1
                    curl -O https://raw.githubusercontent.com/djtoler/Deployment5_v1/main/pkill.sh
                    chmod 777 pkill.sh
                    ./pkill.sh
                "
                '''
            }

# Remove the 'steps' block from the 'Deploy' stage in Jenkinsfilev2. Add these lines of code to reconfigure the pipeline to download our Banking application setup files for the redeployment

        stage ('Deploy') {
            steps {
                sh '''#!/bin/bash
                APPLICATION_IP=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=ec2_2" --query "Reservations[*].Instances[*].PublicIpAddress" --output text)
                ssh -i ~/.ssh/testkey8 -o StrictHostKeyChecking=no ubuntu@$APPLICATION_IP "
                    curl -O https://raw.githubusercontent.com/djtoler/Deployment5_v1/main/setup2.sh
                    chmod 777 setup2.sh
                    ./setup2.sh
                "
                '''
            }
```

### _5. Install CloudWatch for CPU Monitoring_
<p align="center"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/Screenshot-from-2022-01-03-14-01-22.png"></p>

#### We can download this script that will make a CloudWatch config file and install a CloudWatch agent.This will help us monitor how our CPU is performing and alarm us when it reaches a certain threshold. We can then take whatever actions necessary to make sure our continues to run according to our needs.
#### [CloudWatch Automated Installation Script](https://github.com/djtoler/automated_installation_scripts/blob/main/auto-cloud%20watch_config.sh)

#### This script will take 4 arguments...
* ##### How many seconds per logging interval = 60 seconds
* ##### Which system user to use = ubunu user
* ##### What directory to write the logs to = /var/log/syslog
* ##### What group the logs belong to syslog
  
#### We can use it like this...

```
curl -O https://raw.githubusercontent.com/djtoler/automated_installation_scripts/main/auto-cloud%20watch_config.sh
chmod +x auto-cloudwatch_config.sh
./auto-cloudwatch_config.sh 60 ubuntu /var/log/syslog syslog
```

### _6. Configure Multi-Branch Jenkins Pipeline_
#### Run this command to get our current IP address.
```
JENKINS_IP=$(aws ec2 describe-instances --filters "Name=tag:Name,Values=ec2_1" --query "Reservations[*].Instances[*].PublicIpAddress" --output text)

# Login to your Jenkins UI by visiting: <jenkins-server-IP>:8080. In our case...

JENKINS_IP:8080

```
#### Follow the steps outlined in this [configure a Jenkins Multibranch pipeline file](https://github.com/djtoler/automated_installation_scripts/blob/main/manual_jenkinsMultiBranch.txt). Make sure to change Jenkinsfile to Jenkinsfilev1. Also, for the second branch, change Jenkinsfile to Jenkinsfilev2. Run a build from the first branch

<p align="center"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/jenkinsv1success.PNG"></p>

#### Visit <application_ip>:5000. We'll see our app there...

<p align="center"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/bankingapp.PNG"></p>

### _6. Edit Banking Application HTML_
* #### SSH into the application server
* #### `cd` into the application directory. 
* #### Run `git switch -c dev` to make a new branch and switch to it
* #### `cd` into the templates directory 
* #### Run `nano home.html`. 
* #### Make an edit to the HTML file.
* #### Run `git add .`, `git commit -m'changed html'` & `git push -u origin dev`
* #### Switch back to main branch to merge our new changes `git checkout main`
* #### Merge our new changes `git merge dev`
* #### Push our new changes to our main branch `git push origin main`

### A new Jenkins build will be triggered
<p align="center"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/jenkinsv2success.PNG"></p>

### We can also see our HTML changes
<p align="center"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/htmlchange.PNG"></p>

## Questions
#### _How did you decide to run the Jenkinsfilev2?_
* ##### I ran it by switching to it in the Jenkins UI.

#### _Should you place both instances in the public subnet? Or should you place them in a private subnet? Explain why?_
* ##### Both of these instances should 100% be in private subnets. Access to the Jenkins server should be private because Jenkins may have code or other sensitive information stored. The banking application would definitly need to be in a private subnet because they have unlimited amounts of sensitive data

# System Diagram
<p align="center"><img src="https://github.com/djtoler/Deployment5_v1/blob/main/dp5Diagram2.png"></p>


