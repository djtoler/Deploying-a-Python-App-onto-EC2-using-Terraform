# Deployment 5

## Purpose

##### The purpose of this deployment was to continue evolving our appliction infrastructure and our deployment methods. In Deployment 4, we moved away from managed services & default infrastructure configurations. 

##### We set up our infrastructure on a private network and used tools like Nginx & CloudWatch. The private network gave us more control over traffic coming into our infrastructure. Nginx & CloudWatch allowed us to control and monitor ingress traffic to our application. 

##### _These benifits came at a price of a lot of manual interaction with the AWS console to configure & deploy our infrastructure. More manual labor = more potential human error + more costs + more time to complete a task._

#### In Deployment 5, we use Terraform to solve those problems with on a Banking Application. Terraform is an Infrastructure as Code tool that allows use to configure and deploy our application infrastructure using configuration files. This makes our deployments...
* #### _Faster, because after an initial configuration, that infrastructure can be redeployed as fast as it takes to run a script._
* #### _More flexible, because our infrastructre is being built with editable files_
* #### _More scalable through the use of "main" configuration templates with editable variable files that store the details of our infrastructure like virtual machine sizes, ingress/egress traffic control policys, availabiliy zones, ect..._

## Issues

### _1. Terraform Build Fails_
* #### _Problem:_
* #### _Solution:_
* #### _Takeaway:_

### _2. AWS Configuration Issue_
* #### _Problem:_
* #### _Solution:_
* #### _Takeaway:_

### _3. Dynamic IP Issue_
* #### _Problem:_
* #### _Solution:_
* #### _Takeaway:_

### _4. Automation Issue_
* #### _Problem:_
* #### _Solution:_
* #### _Takeaway:_

## Steps
### _1. Plan Infrastructure Design_
### _2. Deploy Infrastructure Using Terraform_
### _3. Configure Our Application & Jenkins Servers_'
### _4. Edit Setup & Jenkinsfiles_
### _5. Configure Multi-Branch Jenkins Pipeline_
### _6. Reconfigure Multi-Branch Jenkins Pipeline_
### _7. Edit Banking Application HTML_


## Questions
#### _How did you decide to run the Jenkinsfilev2?_
* ##### I ran it by switching to it in the Jenkins UI.

#### _Should you place both instances in the public subnet? Or should you place them in a private subnet? Explain why?_
* ##### Both of these instances should 100% be in private subnets. Access to the Jenkins server should be private because Jenkins may have code or other sensitive information stored. The banking application would definitly need to be in a private subnet because they have unlimited amounts of sensitive data








  - System Diagram
  - Optimization 
