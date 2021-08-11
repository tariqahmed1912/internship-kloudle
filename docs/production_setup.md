## **Objective**

The aim of this section is to set up a production server for deploying an application (DVNA).

About DVNA

-   Damn Vulnerable NodeJS Application is a simple NodeJS application to demonstrate **[OWASP Top 10 Vulnerabilities](https://www.owasp.org/index.php/Top_10-2017_Top_10)** and guide on fixing and avoiding these vulnerabilities.

Prerequisites

-   VM running Ubuntu 18.04 LTS.
-   Docker installed

### **Step 1 - Install docker**

Update the apt package index and install packages to allow apt to use a repository over HTTPS

```bash
sudo apt-get update

sudo apt-get install apt-transport-https ca-certificates curl gnupg lsb-release
```
    
Add Docker’s official GPG key

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings docker-archive-keyring.gpg
```
    
Use the following command to set up the stable repository.

```bash
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
    
Update the apt package index, and install the latest version of Docker Engine and containerd.

```bash
sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io

sudo docker run hello-world # Test if docker installation is successful 
```
    

### **Step 2 - Setup Production Server**

DVNA interacts with a MySQL database. Store the db configuration in a file named 'vars.env'.

```bash
MYSQL_USER=dvna
MYSQL_DATABASE=dvna
MYSQL_PASSWORD=passw0rd
MYSQL_RANDOM_ROOT_PASSWORD=yes
MYSQL_HOST=mysql-db
MYSQL_PORT=3306
```
	
Start MySQL container (using environment variables in vars.env file). Run an infinite command in detached mode (using `-d`), so the command never ends and the container never stops. I used `tail -f /dev/null` because it is quite light weight and /dev/null is present in most linux images.
   
```bash
sudo docker run -d --name dvna-mysql --env-file vars.env mysql:5.7 tail -f /dev/null
```
    
Start/run the DVNA application.

```bash
sudo docker run --name dvna-app --env-file vars.env --link dvna-mysql:mysql-db -p 9090:9090 -d appsecco/dvna
```

To see the if the containers are running, run a `docker ps`. We will see two containers running; dvna-app and dvna-mysql. You can stop the running containers by using `docker stop <container-name-or-id>`. 
	
<b>**Note:**</b> 

1.  You can start the containers again using `docker start <container-name-or-id>`. When starting, however, you will have to start `dvna-mysql` container first because the `dvna-app` is dependant on it.

2. The `docker` commands can only be run as sudo user. To enable executing `docker` commands without sudo, type the following in the terminal.

    ```bash
    sudo chmod 666 /var/run/docker.sock
    ```