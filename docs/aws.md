Steps:

1. Launch an EC2 instance.
2. Add security group (firewall rules) and SSH key pair. When adding key pair, the browser will automatically ask to download the private key file. Keep the file safe in directory of your choice. You will need it to establish an SSH connection with the instance.
3. After launch, wait till the instance is up and running.
4. SSH into the instance using the following command. 

        ssh -i /path/to/private-key instance-username@instance-IP-address

    To get the instance-username for SSH login based on your instance OS/distro, refer this [documentation](https://alestic.com/2014/01/ec2-ssh-username/). For an Ubuntu instance, the username is `ubuntu`.

### Jenkins Server

Follow the same steps mentioned in the previous sections to setup Jenkins, SAST and DAST Tools.
        
1. Jenkins - Follow steps mentioned in section `Setup of Jenkins`

2. Docker - Followed this [documentation](https://geekylane.com/install-docker-on-aws-ec2-ubuntu-18-04-script-method/) as I was facing some issue with the executing the `curl` command mentioned in section `Setup of Production Server` -> `Install Docker`.

        curl -fsSL https://get.docker.com -o get-docker.sh

        sudo sh get-docker.sh

Automate the installation process by running the following script in the Jenkins instance.

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

### Production Server 

Install docker on the production server the same way you installed it on the Jenkins server. 