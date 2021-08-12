## **Objective**

The aim of this section is to perform static analysis on DVNA, using SAST and SCA tools, in a Jenkins pipeline.

About SAST

-   Static application security testing (SAST) is a testing methodology that analyzes source code to find security vulnerabilities.
-   SAST scans an application before the code is compiled. Its also known as white box testing.

About SCA

- Software composition analysis (SCA) identifies all the open source in a codebase and maps that inventory to a list of current known vulnerabilities.
- It helps identify vulnerabilities in open source code (dependencies) used in code.

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

Auditjs is a SAST tool which detects vulnerablilites in dependencies (node modules) used in your application. It interacts with Sonatype Nexus IQ Server to check for known and disclosed vulnerabailites.

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

**Note**: You can find the API-token in the `User settings` after logging into the OSS Index website.

```bash
auditjs ossi --username <USERNAME> --token <API-TOKEN> > ~/report/auditjs-report
```

### **OWASP Dependancy-Check**

OWASP Dependency-Check is a software composition analysis (SCA) tool that detects publicly disclosed vulnerabilities contained within a project’s dependencies.

To start working with Dependency Check, I followed the [official documentation](https://jeremylong.github.io/DependencyCheck/dependency-check-cli/index.html). First, download the Dependency-Check CLI tool and the associated GPG signature file.

```bash
wget -P ~/ https://github.com/jeremylong/DependencyCheck/releases/download/v6.2.2/dependency-check-6.2.2-release.zip

wget -P ~/ https://github.com/jeremylong/DependencyCheck/releases/download/v6.2.2/dependency-check-6.2.2-release.zip.asc
```

Next, extract the files from the dependency-check tool zip file.

```bash
unzip ~/dependency-check-6.2.2-release.zip
```

Perform the scan by specifying the path to the project, output report format and its location.

```bash
~/dependency-check/bin/dependency-check.sh --scan ~/app --out ~/report/dependency-check-report --format JSON --prettyPrint
```

This scan is inclusive of Retire.js scan, NPM Audit scan, and Auditjs scan, to name a few.

### **SAST Pipeline**

The static analysis is done by copying the DVNA code in Production server to Jenkins server, and then running multiple static analysis scans.

The Jenkinsfile for performing SAST and SCA of DVNA via Jenkins pipeline is given below:

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

    stage ('OWASP Dependency-Check') {
      steps {
        sh '~/dependency-check/bin/dependency-check.sh --scan ~/app --out ~/report/dependency-check-report --format JSON --prettyPrint || true'
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
