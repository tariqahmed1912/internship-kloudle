The following is the report/documentation for the problem statements stated in the next section. The contents of the report are:

1. [Introduction](index.md)
2. [Table of Content](contents.md)
3. [Setup of VMs](vm_setup.md)
4. [Setup of Jenkins](jenkins_setup.md)
5. [Setup of Production Server](production_setup.md)
6. [SSH Connection between VMs](ssh_connection.md)
7. [Static Analysis](static_analysis.md)

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