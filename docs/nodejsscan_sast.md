## Objective

In this section, we will perform SAST using NodeJsScan on DVNA in Production server.

About SAST

-   Static application security testing (SAST) is a testing methodology that analyzes source code to find security vulnerabilities.
-   SAST scans an application before the code is compiled. Its also known as white box testing.

Prerequisites

-   An application (DVNA) running on Production Server.

### Web-based

Pull NodeJsScan docker image

```bash
sudo docker pull opensecurity/nodejsscan:latest
```
    
Run nodjsscan container

```bash
sudo docker run -it -p 9090:9090 opensecurity/nodejsscan:latest -d
```
    
You can access the website by typing `<ip-address>:9090` in the browsers URL. To perform SAST test, upload the files (individual files or a zip file) and run the scan.

### CLI-based

In the Production server, enter DVNA container in exec mode.

```bash
sudo docker exec -it -u 0 dvna-app /bin/bash
```

To install nodejsscan, we first need to install pip3 in 'dvna-app' container in production server.

```bash
apt update

apt install python3-pip
```

Install nodejsscan

```bash
pip3 install nodejsscan
```

Scan the /app directory (which holds the files for DVNA) and store the scan result in `/app/report/nodejsscan-report`

```bash
nodejsscan -d /app -o /app/report/nodejsscan-report
```


### Jenkins Pipeline

The static analysis is done by copying the DVNA code in Production server to Jenkins server, and then running a NodeJsScan.

First, install docker on the Jenkins server. Follow the steps given in the `Install Docker` section of the `Setup of Production Server` page. The `docker` commands can only be run as sudo user. To enable executing `docker` commands without sudo, type the following in the terminal.

```bash
sudo chmod 666 /var/run/docker.sock
```

The Jenkinsfile for performing SAST of DVNA via Jenkins pipeline is given below:

```bash
pipeline {
  agent any
  stages {
    stage ('Initialization') {
      steps {
        sh 'echo "Starting the build..."'
      }
    }
    
    stage('MySQL db') {
      steps {
        sh 'scp tariq@192.168.56.102:~/vars.env ~/'
      }
    }
    
    stage('Run Application Container') {
      steps {
        sh 'docker run -d --name dvna-mysql-test --env-file ~/vars.env mysql:5.7 tail -f /dev/null'
      }
    }
    
    
    stage('NodeJsScan') {
      steps {
        sh 'ssh -o StrictHostKeyChecking=no tariq@192.168.56.102 "docker exec -u 0 dvna-app-test nodejsscan -d /app -o /app/report/nodejsscan-report-test"'
      }
    }
    
    stage ('Final') {
      steps {
        sh 'echo "Scan successfully completed!"'
      }
    }
  }
}
```
