## **Objective**

In this section, we will be using various tools to perform SAST on DVNA in Production server.

About SAST

-   Static application security testing (SAST) is a testing methodology that analyzes source code to find security vulnerabilities.
-   SAST scans an application before the code is compiled. Its also known as white box testing.

Prerequisites

-   An application (DVNA) running on Production Server.

### **NodeJsScan**

NodeJsScan is a static code scanner which is used to find security flaws in Node.js applications.

#### Web-based 

Pull NodeJsScan docker image

```bash
sudo docker pull opensecurity/nodejsscan:latest
```
    
Run nodjsscan container

```bash
sudo docker run -it -p 9090:9090 opensecurity/nodejsscan:latest -d
```
    
You can access the website by typing `<ip-address>:9090` in the browsers URL. To perform SAST, upload the files (as a zip file) or individual file and run the scan.

#### CLI-based

In the Production server, enter DVNA container in exec mode.

```bash
sudo docker exec -it -u 0 dvna-app /bin/bash
```

To install njsscan, we first need to install pip3 in 'dvna-app' container in production server.

```bash
apt update && apt install python3-pip
```

Install njsscan

```bash
pip3 install njsscan
```

Scan the ~/app directory (which holds the files for DVNA) and store the scan result in `~/report/nodejsscan-report`

```bash
mkdir ~/report

njsscan --json -o /app/report/nodejsscan-report ~/app
```

### **Auditjs**

In the Production server, enter DVNA container in exec mode.

```bash
sudo docker exec -it -u 0 dvna-app /bin/bash
```

To install auditjs, we first need to install npm and nodejs in 'dvna-app' container in production server. After running the following commands, the package versions are: `npm v6.14.14` and `nodejs v14.17.4`

```bash
sudo apt update
sudo curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt install -y nodejs npm 
sudo npm install latest-version
```

Install auditjs using npm

```bash
sudo npm install -g auditjs
```

Scan the ~/app directory (which holds the files for DVNA) and store the scan result in `~/report/auditjs-report`. 

```bash
cd ~/app
auditjs ossi > ~/report/auditjs-report
```

If your NodeJs project is very large, you might face rate-limit issues. To solve this issue, create a free account
at OSS Index and run the scan with your accounts 'Username' and 'API-token'. 

**Note**: You can find the API-token in the `User settings` after logging into the OSSI Index website.

```bash
auditjs ossi --username <USERNAME> --token <API-TOKEN> > ~/report/auditjs-report
```

### **SAST Pipeline**

The static analysis is done by copying the DVNA code in Production server to Jenkins server, and then running multiple static analysis scans.

The Jenkinsfile for performing SAST of DVNA via Jenkins pipeline is given below:

```bash
pipeline {
  agent any
  stages {
    stage ('Initialization') {
      steps {
        sh 'echo "Starting the build!"'
      }
    }
    
    stage('Copy Application Code') {
      steps {
        sh 'ssh -o StrictHostKeyChecking=no tariq@192.168.56.102 "docker start dvna-mysql && docker start dvna-app; docker cp dvna-app:/app/ ~/; docker stop dvna-app && docker stop dvna-mysql;"'
        sh 'scp -rC tariq@192.168.56.102:~/app ~/ && mkdir -p ~/report'
      }
    }
    
    stage('NodeJsScan') {
      steps {
        sh 'njsscan --json -o ~/report/nodejsscan-report ~/app || true'
      }
    }
    
    stage('Auditjs') {
      steps {
        sh 'cd ~/app; auditjs ossi > ~/report/auditjs-report || true'
      }
    }

    stage ('Final') {
      steps {
        sh 'rm -rf ~/app'
        sh 'echo "Scan successfully completed!"'
      }
    }
  }
}

```