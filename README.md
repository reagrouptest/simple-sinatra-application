# sinatra-app
This project is used to host sinatra app in AWS. 
# Purpose
The aim of this repository is to house sinatra application to server on port 80. It is a internet facing application which has been designed for high availability using AWS resources.



# Assignment specific links
* [Question](https://github.com/rea-cruitment/simple-sinatra-app.git)


 
# Instruction to run the project.

## Prerequisite.
* User should have AWS account.
* User with root level privileges.

## Assumptions
* This architecture should be used only for development purpose as it serves HTTP request.
  Recommendation is to use HTTPS for security purpose.
* The application is designed in single region - Australia.
* The request is served on HTTP port using load balancer.
* Root user is used only for test purpose. It is advised not to use root user.
* AWS infrastructure has public subnet. You should not deploy your application in public subnet.
* Due to limited time, the project has not been designed in best way. Documentation can also be  
  made much better. Feel free to reach on rsrahulsingh666@gmail.com for queries.



# Design decision.

The application uses custom VPC instead of using default VPC. This helps to control your network security in more efficient way. 


# Instruction to create project.

Please download the GIT repository(as mentioned in email) and follow below instruction.
1. Create network in AWS.
2. Create application infrastructure in AWS.
3. Deploy sinatra application.

## Create AWS network:
1. Login to AWS console.
 [aws login](https://aws.amazon.com/console/)
2. Look for AWS service- cloud formation. Go to the service and select 'create stack'. To create stack you can upload the file to S3 or use your local machine to provide the file. In this case you can use your local machine to 'upload a template file' called vpc.yml which was downloaded from git repository. This will build your network in AWS.
Process flow-
AWS service-->cloudformation-->create stack-->upload a template file-->choose file-->stack name
3. Wait for the cloud formation to create the stack of your network. The status should change to 'Create_complete'. Once the status changes to completed. Your network is ready to build your first ec2 instance.


## Create application infrastructure in AWS.
1. Go to the service ec2 and create your key pairs with the name 'sinatra-key'. Once you have created the key-pair it will be downloaded to your local which you can use to login to the application server.
Process flow-
AWS service-->ec2-->key pairs-->create key pair.
2. Create KMS key to encrypt the ec2 volume by following below process.
Process flow-
AWS service-->customer managed keys-->create alias(sinatrakmskey)-->key administrators(cloudformationrole)-->key usage permission(cloudformationrole)-->finish
3. Create application stack including all resources using ec2.yml file.
Process-
AWS service-->cloudformation-->create stack-->upload a template file(ec2.yml)-->choose file-->stack name-->mark tick to acknowledge IAM creation




# Application deployment and automation.

## Why ansible?
1. Low overhead- due to agentless model, Ansible reduces the overheads on the network by preventing the nodes from polling the controlling machine.
2. Secure and consistent- Ansible only uses SSH and Python on the managed nodes. This ensures safety and security. Also, Ansible ensures consistent environments.
3. Reliable- an Ansible playbook can be idempotent when written carefully. This prevents unexpected side-effects on the managed systems.
4. Good performance- Ansible delivers flawless performance. Though it is very easy to set up yet it is a powerful tool for deploying software applications using SSH.

## Application deployment.

###Assumptions.
 1. You have git bash installed in your local machine.




Steps:

1. You will need to login to the public linux box and pull the GIT repo(as mentioned in the email).
   To login- Open your git bash and login to the public server using the public ip.(You can get the public IP from AWS stack)
   ```
   ssh ec2-user@ip -i sinatra_key.pem 
   ```
   where ip=your public linux server ip and sinatra_key is the key created through aws console.

 2. Append your sinatra_key.pem to authorized_keys of ec2-user. This is not recommended in production servers.
    ```
     cd  /home/ec2-user/.ssh
     vi authorized_keys
    ```

   
2. Install ansible and git as below
  ```
  sudo yum -y install python-pip
  sudo pip install ansible
  sudo yum -y install git

  ```

3. Clone the git repo for application deployment through ansible.

  ```
   sudo mkdir /git
   cd /git
   sudo chmod 777 /git
   sudo git clone https://github.com/reagrouptest/simple-sinatra-application.git

   ```


4. Go to the playbook path and run the ansible playbook as below.
 ```
 cd /git/simple-sinatra-application/deploy
 ansible-playbook app_deploy.yml -i hosts

 ```
  
You can follow above method to deploy your application. Though the preferred way of doing is through CI-CD tools like Jenkins etc.




# Network design


This document contains the information about network and it's various component. The network is designed for test purposes only and should not be used for production.

Assumptions:


* The network is designed in single region- AWS Sydney and uses most resources from AWS Sydney DC.
* Application is standalone with no databases.



# VPC


VPC is most important component of cloud based infrastructure and it is important to get it right from the first. VPC design has far-reaching implications for scaling, fault-tolerance and security. It is responsible for infrastructure flexibility and will save your important time to free up address space. It is important to lay out VPC in correct way then the careless way. VPC for this project is set up in AWS Sydney region.

## Subnet


 Subnet layout is the key to a well-functioning VPC. Subnets determine routing, Availability Zone distribution and Network Access Control Lists. VPC is not a data center, data center network or switches. A VPC is a software defined network optimized for moving massive amount of packets into, out of and across AWS region.

 Below are few reason of creating different subnets:
* You need different hosts to route in different ways- Internal only, public-facing.
* You need to distribute workloads across different multiple AZ to achieve fault-tolerance.
* Security requirements for NACL and route table to serve specific ports and subnets.

In this project 2 subnets in each AZ are created to server traffic.


## Routing


It is important to apply the principle of proper and secure routing when you are exposing your servers to internet. Route tables helps to route traffic from Internet to the VPC.
The project uses 2 route tables - Public and Private. Public route table is associated with Public subnet and private route table is associated with Private subnet. 

## Fault-Tolerance


AWS provides geographic distribution out of the box in the form of Availability Zones (AZs). Every region has at least two.

Subnets cannot span multiple AZs. So to achieve fault tolerance, you need to divide your address space among the AZs evenly and create subnets in each. The more AZs, the better. if you have three AZs available, split your address space into different parts. 
The reason you need to divide your address space evenly is so the layout of each AZ is the same as others. This will help resources like autoscaling groups to be evenly distributed. If you create disjointed address blocks, it will require lot of maintenance.

In this project we have divided our subnet into similar block . The division is based on block not on number of IPs.


## Security


The first layer of defense in a VPC is the routing layer.

Above the routing layer are two levels of complementary controls: Security Groups and NACLs. Security Groups are dynamic, stateful and capable of spanning the entire VPC. NACLs are stateless (meaning you need to define inbound and outbound ports), static and subnet-specific.

Generally, you only need both if you want to distribute change control authority over multiple groups of admins. For instance, you might want your sys admin team to control the security groups and your networking team to control the NACL’s. That way, no one party can single-handedly defeat your network restrictions.

In practice, NACLs should be used sparingly and, once created, left alone. Given that they’re subnet-specific and punched down by IP addresses, the complexity of trying to manage traffic at this layer increases geometrically with each additional rule.

Security Groups are where the majority of work gets done. Unless you have a specific use-case like the ones described earlier, you’ll be better served by keeping your security as simple and straightforward as possible. That’s what Security Groups do best.

The VPC for this project is  subdivided into different blocks as below -

```
10.0.0.0/16:
AZ A
    10.0.0.0/19 — Private
            10.0.32.0/20 — Public
AZ B
    10.0.64.0/19 — Private
            10.0.96.0/20 — Public
                    
AZ C
   10.0.128.0/19 — Private
            10.0.160.0/20 — Public

     
```



# References

* [AWS whitepapers.](https://aws.amazon.com/whitepapers/)
* [AWS best practices.](https://aws.amazon.com/whitepapers/architecting-for-the-aws-cloud-best-practices/)
* AWS and git blogs.