Steps:

1. Launch an EC2 instance.
2. Add security group (firewall rules) and SSH key pair. When adding key pair, the browser will automatically ask to download the private key file. Keep the file safe in directory of your choice. You will need it to establish an SSH connection with the instance.
3. After launch, wait till the instance is up and running.
4. SSH into the instance using the following command. 

        ssh -i /path/to/private-key instance-username@instance-IP-address

    To get the instance-username for SSH login based on your instance OS/distro, refer this [documentation](https://alestic.com/2014/01/ec2-ssh-username/). For an Ubuntu instance, the username is `ubuntu`.

### Install Jenkins, SAST and DAST Tools

Follow the same steps mentioned in the previous sections to setup Jenkins, SAST and DAST Tools.
        
1. Jenkins - Follow steps mentioned in section `Setup of Jenkins`

2. Docker - Followed this [documentation](https://geekylane.com/install-docker-on-aws-ec2-ubuntu-18-04-script-method/) as I was facing some issue with the executing the `curl` command mentioned in section `Setup of Production Server` -> `Install Docker`.

        curl -fsSL https://get.docker.com -o get-docker.sh

        sudo sh get-docker.sh

