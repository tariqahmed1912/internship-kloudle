## Objective

In this section, we will be using various tools to perform SAST on DVNA in Production server.

About SAST

-   Static application security testing (SAST) is a testing methodology that analyzes source code to find security vulnerabilities.
-   SAST scans an application before the code is compiled. Its also known as white box testing.

Prerequisites

-   An application (DVNA) running on Production Server.

### NodeJsScan

#### Web-based 

Pull NodeJsScan docker image

```bash
sudo docker pull opensecurity/nodejsscan:latest
```
    
Run nodjsscan container

```bash
sudo docker run -it -p 9090:9090 opensecurity/nodejsscan:latest -d
```
    
You can access the website by typing `<ip-address>:9090` in the browsers URL. To perform SAST, upload the files (individual files or a zip file) and run the scan.

#### CLI-based

In the Production server, enter DVNA container in exec mode.

```bash
sudo docker exec -it -u 0 dvna-app /bin/bash
```

To install njsscan, we first need to install pip3 in 'dvna-app' container in production server.

```bash
apt update

apt install python3-pip
```

Install njsscan

```bash
pip3 install njsscan
```

Scan the /app directory (which holds the files for DVNA) and store the scan result in `/app/report/nodejsscan-report`

```bash
mkdir /app/report

njsscan --json -o /app/report/nodejsscan-report /app
```

### Auditjs

In the Production server, enter DVNA container in exec mode.

```bash
sudo docker exec -it -u 0 dvna-app /bin/bash
```

To install auditjs, we first need to install npm in 'dvna-app' container in production server.

```bash
apt update

apt install npm
```

Install auditjs using npx

```bash
npx auditjs@latest ossi
```

Scan the /app directory (which holds the files for DVNA) and store the scan result in `/app/report/nodejsscan-report`

```bash
mkdir /app/report

njsscan --json -o /app/report/nodejsscan-report /app
```



### Jenkins Pipeline

The static analysis is done by copying the DVNA code in Production server to Jenkins server, and then running multiple static analysis scans.

Install njsscan in the Jenkins server.

```bash
apt update && apt install python3-pip && pip3 install njsscan
```

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
        sh 'ssh -o StrictHostKeyChecking=no tariq@192.168.56.102 "docker cp dvna-app:/app/ ~/"'
        sh 'scp -rC tariq@192.168.56.102:~/app ~/ && mkdir -p ~/report'
      }
    }
    
    stage('NodeJsScan') {
      steps {
        sh 'njsscan --json -o ~/report/nodejsscan-report ~/app && (exit 0) || (exit 1)'
      }
    }
    
    stage ('Final') {
      steps {
        sh 'rm -rf ~/app && rm -rf ~/vars.env'
        sh 'echo "Scan successfully completed!"'
      }
    }
  }
}
```
