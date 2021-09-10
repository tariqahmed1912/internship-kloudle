### **Objective**

The aim of this section is to set up a production server for deploying an application (DVNA) and solve the 3rd point of the [Problem Statement](problem_statements.md) under Task 1.

About DVNA

-   Damn Vulnerable NodeJS Application is a simple NodeJS application to demonstrate **[OWASP Top 10 Vulnerabilities](https://www.owasp.org/index.php/Top_10-2017_Top_10)** and guide on fixing and avoiding these vulnerabilities.

Prerequisites

-   VM running Ubuntu 18.04 LTS.
-   Docker installed

---

### **Step 1 - Install docker**

For docker installation, I followed this [documentation](https://geekylane.com/install-docker-on-aws-ec2-ubuntu-18-04-script-method/) as its very simple and easy to execute.

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

Alternatively, you can follow this [official documentation](https://docs.docker.com/engine/install/ubuntu/) for docker installation. First, Update the apt package index and install packages to allow apt to use a repository over HTTPS.

```bash
sudo apt-get update

sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
```
    
Then add Docker’s official GPG key.

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings docker-archive-keyring.gpg
```
    
Use the following command to set up the stable repository.

```bash
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Update the apt package index, and install the latest version of Docker Engine and containerd (a daemon process that manages and runs containers).

```bash
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io

sudo docker run hello-world # Test if docker installation is successful 
```

---

### **Step 2 - Setup Production Server**

I followed the [official documentation](https://github.com/appsecco/dvna) available on GitHub to setup DVNA. DVNA interacts with a MySQL database. Store the db configuration in a file named `vars.env`.

```bash
MYSQL_USER=dvna
MYSQL_DATABASE=dvna
MYSQL_PASSWORD=passw0rd
MYSQL_RANDOM_ROOT_PASSWORD=yes
MYSQL_HOST=mysql-db
MYSQL_PORT=3306
```

Start MySQL container (using environment variables in `vars.env` file). Run an infinite command in detached mode (using `-d`), so the command never ends and the container never stops. I used `tail -f /dev/null` because it is quite light weight and /dev/null is present in most linux images.
   
```bash
sudo docker run -d --name dvna-mysql --env-file vars.env mysql:5.7 tail -f /dev/null
```
    
Start/run the DVNA application.

```bash
sudo docker run --name dvna-app --env-file vars.env --link dvna-mysql:mysql-db -p 9090:9090 -d appsecco/dvna
```

To test if the containers are running, run a `docker ps`. You should see two containers running; dvna-app and dvna-mysql. You can stop running containers by using `docker stop <container-name-or-id>`. 

For deploying DVNA using Jenkins pipeline, the application will first be run in a docker container in the Jenkins VM. Static and dynamic analysis will be done and only then, will the application be deployed in the production server.

---

<b>**Note:**</b> 

1.  You can start the containers again using `docker start <container-name-or-id>`. When starting, however, you will have to start `dvna-mysql` container first because `dvna-app` is dependant on it.

2. The `docker` commands can only be run as sudo user. There are 2 ways to enable executing `docker` commands without sudo. Although, you must know, its highly discouraged to do so. 

    (i) Add 'user' to group 'docker' by typing the following command. The changes might not take effect without rebooting your VM. 
    
        sudo usermod -aG docker $USER
        sudo reboot

    (ii) This method should only be used if no other method seems to work, since it grants every user permission to execute and run docker containers.

        sudo chmod 666 /var/run/docker.sock

3. Follow these steps to completely uninstall/remove nodejs and npm from your VM.

    Removing Nodejs and Npm using `apt` package manager.

        sudo apt-get remove nodejs npm node
        sudo apt-get purge nodejs


    Now manually remove .node and .npm folders from your VM.

        sudo rm -rf /usr/local/bin/npm 
        sudo rm -rf /usr/local/share/man/man1/node* 
        sudo rm -rf /usr/local/lib/dtrace/node.d 
        sudo rm -rf ~/.npm 
        sudo rm -rf ~/.node-gyp 
        sudo rm -rf /opt/local/bin/node 
        sudo rm -rf opt/local/include/node 
        sudo rm -rf /opt/local/lib/node_modules  
        sudo rm -rf /usr/local/lib/node*
        sudo rm -rf /usr/local/include/node*
        sudo rm -rf /usr/local/bin/node*

    Go to home directory and remove any node or node_modules directory.

    You can verify your uninstallation by running these commands; they should not return any output.

        which node
        which nodejs
        which npm