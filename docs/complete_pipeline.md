## **Objective**

The aim of this section is to show the complete CI/CD pipeline structure.


### **Complete Pipeline**

```bash
pipeline {
  agent any
  stages {
    stage ('Initialization') {
      steps {
        sh 'echo "Starting the build!"'
      }
    }
    
    stage('Start DVNA and Copy DVNA Code') {
      steps {
        sh 'ssh -o StrictHostKeyChecking=no tariq@192.168.56.102 "docker start dvna-mysql && docker start dvna-app; docker cp dvna-app:/app/ ~/;"'
        sh 'scp -rC tariq@192.168.56.102:~/app ~/ && mkdir ~/reports && chmod 777 ~/reports'
      }
    }
    
    stage('NodeJsScan Analysis') {
      steps {
        sh 'njsscan --json -o ~/reports/nodejsscan-report ~/app || true'
      }
    }
    
    stage('Auditjs Analysis') {
      steps {
        sh 'cd ~/app; auditjs ossi > ~/reports/auditjs-report || true'
      }
    }

    stage ('OWASP Dependency-Check Analysis') {
      steps {
        sh '~/dependency-check/bin/dependency-check.sh --scan ~/app --out ~/reports/dependency-check-report --format JSON --prettyPrint || true'
      }
    }
    
    stage('OWASP ZAP Analysis') {
      steps {
        sh 'docker run --rm -i -u zap --name owasp-zap -v ~/reports/:/zap/wrk/ owasp/zap2docker-stable zap-baseline.py -t http://192.168.56.102:9090 -r zap-report.html -l PASS || true'
      }
    }

    stage ('Generating Software Bill of Materials') {
      steps {
        sh 'cyclonedx-bom -o ~/reports/sbom.xml'
      }
    }
    
    stage ('Stop DVNA') {
      steps {
        sh 'ssh -o StrictHostKeyChecking=no tariq@192.168.56.102 "docker stop dvna-app && docker stop dvna-mysql;"'
        sh 'rm -rf ~/app'
        sh 'echo "Scan successfully completed!"'
      }
    }
    
  }
}
```