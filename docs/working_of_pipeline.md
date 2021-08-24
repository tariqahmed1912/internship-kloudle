## **Objective**

The aim of this section is to understand the Jenkins pipeline to deploy DVNA and solve the points 1-8 in [Problem Statement](problem_statements.md) under Task 1.


### **Jenkins Pipeline**

Jenkins is a continuous integration server which has the ability to support the automation of software development processes. It can be used to create several automation jobs and run them as a pipeline. A Jenkins Pipeline is a series of jobs/events to perform build, testing and deployment of software. Every job has some sort of dependency on at least one or more jobs or events in a pipeline. In DevSecOps, security is integrated into the pipeline by performing static and dynamic analysis using various SAST and DAST tools.

### **Jenkinsfile**

A Jenkinsfile is a text file that contains the definition of a Jenkins Pipeline and is checked into source control. 

```bash
pipeline {
  agent any
  
  stages {
    stage ('Initialization') {
      steps {
        sh 'echo "Starting the build!"'
      }
    }

   stage ('Final') {
     steps {
       sh 'echo "Final stage!"'
     }
   }
```

The above sample consists of two `stages`; Initialization and Final. In each `stage`, shell commands can be run under `steps` using `sh`.

### **Pipeline for Deploying DVNA**

The pipeline is divided into various stages based on the operations being performed. They are as follows:

**Initialization**

This is the first stage in the pipeline and is used just to indicate the start of the build.

**Build**

In this stage, `vars.env` is created as it contains the environment variables required to link the `dvna-app` container with the `dvna-mysql` container. These containers are then run and the application code is copied into the hosts `jenkins` home directory (in my case, `/var/lib/jenkins`).

**SAST and DAST**

All the stages following the `Build` stage, excluding the last two stages, are for performing SAST and DAST on the application. The scan output reports are stored in a folder named `reports` in `jenkins` home directory.  
**NOTE:** A lot of scans like NodeJsScan, AuditJs, JSHint, etc. return a non-zero exit code, even on successful completion. Jenkins considers non-zero status code as `FAILED` and stops the build. To overcome this, you can add either of the following at the end of the scan commands to give a `0` status code.  
```bash
<scan command> || true 
OR
<scan command>; echo $? > /dev/null
```

**Take DVNA Offline**

After the scans are complete, the running containers are stopped. Since we're working with a containerized application (DVNA), we need to perform tests on the latest available image on DockerHub. Hence, we remove the existing local  `appsecco/dvna` docker image to avoid running a container with older release of the application image. On the other hand, you don't need to remove the `mysql:5.7` image, since we require v5.7 and not the latest version.

**Deploy DVNA to Production**

Finally, operations are performed on the Production VM over SSH (configured in the section titled `SSH Connection Between VMs`).  `vars.env` file is copied into Production server, and the two containers; `dvna-app` and `dvna-mysql`, are run.