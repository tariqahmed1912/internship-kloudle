## **Objective**

The aim of this section is to explain the software composition of DVNA using SCA tools in a Jenkins pipeline.

About SCA

- Software composition analysis (SCA) identifies all the open source in a codebase and maps that inventory to a list of current known vulnerabilities.
- It helps identify vulnerabilities in open source code (dependencies) used in code.

Prerequisites

-   An application (DVNA) running on Production Server.

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

### **SCA Pipeline**

The Jenkinsfile for performing SCA of DVNA via Jenkins pipeline is given below:

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
