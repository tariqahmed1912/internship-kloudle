## **Objective**

The aim of this section is to perform static analysis on DVNA using SAST tools in a Jenkins pipeline and solve the 4th point of the [Problem Statement](problem_statements.md) under Task 1.

About SAST

-   Static application security testing (SAST) is a testing methodology that analyzes source code to find security vulnerabilities.
-   SAST scans an application before the code is compiled. Since the test is performed on the source code, ie. the internal structures or workings of an application, and not its functionality, its also called white box testing.

Prerequisites

-   An application (DVNA) running on Production Server.

### **NodeJsScan**

NodeJsScan is a static code scanner which is used to find security flaws in Node.js applications.

In the Jenkins server, enter DVNA container in exec mode.

```bash
sudo docker exec -it -u 0 dvna-app /bin/bash
```

Following the [official documentation](https://github.com/ajinabraham/nodejsscan) available on GitHub, I installed njsscan using pip3. To use pip3, we first need to install it in the `dvna-app` container in production server.

```bash
apt update && apt install python3-pip
```

Install njsscan using pip3.

```bash
pip3 install njsscan
```

Scan the `~/app` directory (which holds the files for DVNA) and store the scan result in `~/reports/nodejsscan-report`

```bash
mkdir ~/reports

njsscan --json -o /app/reports/nodejsscan-report ~/app
```

**NodeJsScan Pipeline**

```bash
stage('NodeJsScan') {
  steps {
    sh 'njsscan --json -o ~/reports/nodejsscan-report ~/app || true'
  }
}
``` 

### **Auditjs**

Auditjs is a SAST tool which detects vulnerablilites in dependencies (node modules) used in your application. It interacts with Sonatype Nexus IQ Server to check for known and disclosed vulnerabailites.

In the Jenkins server, enter DVNA container in exec mode.

```bash
sudo docker exec -it -u 0 dvna-app /bin/bash
```

Following the [official documentation](https://github.com/ajinabraham/nodejsscan) available on GitHub, I installed auditjs using NPM (Node Packet Manager). To install `npm` and `nodejs` in `dvna-app` container in production server, I referred this [documentation](https://www.tecmint.com/install-nodejs-npm-in-centos-ubuntu/) as it's clear and easy to follow. After running the following commands, the package versions are: `npm v6.14.14` and `nodejs v14.17.4`.  
**Note:** Installing nodejs from deb.nodesource.com comes with a prepackaged NPM. So you don't need to install `npm` using the `apt` package manager. Additionally, to update npm to latest version, use `sudo npm install latest-version` after running the commands below. 

```bash
sudo apt update
sudo curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt install -y nodejs
```

Install auditjs using npm.

```bash
sudo npm install -g auditjs
```

Scan the `~/app` directory (which holds the files for DVNA) and store the scan result in `~/reports/auditjs-report`. 

```bash
cd ~/app
auditjs ossi > ~/reports/auditjs-report
```

If your NodeJs project is very large, you might face rate-limit issues. To solve this issue, create a free account
at OSS Index and run the scan with your accounts `username` and `API-token`. 

**Note**: You can find the `API-token` in the `User Settings` after logging into the OSS Index website.

```bash
auditjs ossi --username <USERNAME> --token <API-TOKEN> > ~/reports/auditjs-report
```

**AuditJs Pipeline**

```bash
stage('Auditjs') {
  steps {
    sh 'cd ~/app; auditjs ossi > ~/reports/auditjs-report || true'
  }
}
```

### **SAST Pipeline**

The static analysis is done on the application source code in `dvna-app` container running on Jenkins server.

Add the following stages to the Jenkinsfile for performing SAST of DVNA.

```bash
stage('NodeJsScan') {
  steps {
    sh 'njsscan --json -o ~/reports/nodejsscan-report ~/app || true'
  }
}

stage('Auditjs') {
  steps {
    sh 'cd ~/app; auditjs ossi > ~/reports/auditjs-report || true'
  }
}
```