### **Objective**

The aim of this section is to shift the entire setup from local machine to AWS Cloud and solve the 10th point of the [Problem Statement](problem_statements.md) under Task 1.

About AWS

- Amazon Web Services (AWS) is the world's most comprehensive and broadly adopted cloud platform, offering over 200 fully featured services from data centers globally.  
- Migrating your local/on-prem infrastructure to cloud can help reduce costs of operations, increase IT staff productivity, and reduce downtime.

### **Setup EC2 Instances**

Amazon Elastic Compute Cloud (Amazon EC2) is a web service that provides secure, resizable compute capacity in the cloud. It is a service that enables business subscribers to run application programs in the computing environment. It can serve as a practically unlimited set of VMs. 

An EC2 instance is a virtual server in Amazon's EC2 for running applications on the AWS infrastructure. Instances are created from Amazon Machine Images (AMI). AMIs are like templates that are configured with an OS, CPU power, memory, storage and other networking resources to suit user needs.

Steps to create an EC2 instance:

1. Create an AWS account on [https://aws.amazon.com](https://aws.amazon.com). 
2. After logging in to your account, select `All Services` and under the `Compute` section, click `EC2` in the `AWS Management Console` page. You will be redirected to `EC2 Management Console`
3. Select `Launch EC2 instance`.
4. Select AMI of your choice, add security group (firewall rules) and SSH key pair. When adding key pair, the browser will automatically ask to download the private key file. Keep the file safe in directory of your choice. You will need it to establish an SSH connection with the instance.
5. After launch, wait till the instance is up and running.
6. SSH into the instance using the following command. 

        ssh -i /path/to/private-key instance-username@instance-IP-address

    To get the instance-username for SSH login based on your instance OS/distro, refer this [documentation](https://alestic.com/2014/01/ec2-ssh-username/). For an Ubuntu instance, the username is `ubuntu`.
    
7. Create three instances; Jenkins instance (master), DAST instance (agent), Production instance. 


### **Jenkins Server**

Spin up an instance for Jenkins server. Automate the installation process of Jenkins, Docker and static analysis tools by running the following script in the Jenkins instance.  
**Note:** Initially, I tried running all the scans in Jenkins instance via pipeline. But the instance crashed/hung when running the OWASP ZAP scan. Since I'm using a Free Tier version, I can only start instances with 1GB memory, which isn't sufficient to run all these scans. To solve this issue, I'm using a Master-Agent architecture in which the DAST scan will be allocated to an Agent (separate instance).

```bash
#!/bin/bash
sudo apt update

# Install Java
sudo apt install -y default-jre default-jdk 

# Install Jenkins
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add - &&
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list' &&
sudo apt install -y jenkins &&
sudo systemctl start jenkins

# Install Docker
sudo curl -fsSL https://get.docker.com -o get-docker.sh &&
sudo sh get-docker.sh &&
sudo usermod -aG docker jenkins

# Install Python3 and Pip3
sudo apt install -y python3-pip

# Install NodeJs and NPM
sudo curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash - &&
sudo apt install -y nodejs 

# Install NodeJsScan (SAST)
pip3 install njsscan

# Install AuditJS (SAST)
sudo npm install -g auditjs

# Install OWASP Dependency-Check (SCA)
wget -P ~/ https://github.com/jeremylong/DependencyCheck/releases/download/v6.2.2/dependency-check-6.2.2-release.zip &&
wget -P ~/ https://github.com/jeremylong/DependencyCheck/releases/download/v6.2.2/dependency-check-6.2.2-release.zip.asc &&
unzip ~/dependency-check-6.2.2-release.zip

# Install CycloneDX (SBoM)
sudo npm install -g @cyclonedx/bom

# Install JSHint (Code Linting)
sudo npm install -g jshint

# Install ESLint (Code Linting)
# Note: To use eslint, manually create a .eslintrc.json file in the `jenkins` home directory.
# Copy the file content from documentation
sudo npm install -g eslint
```

### **Production Server**

Install docker on the production server the same way you installed it on the Jenkins server. 