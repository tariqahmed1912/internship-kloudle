## **Objective**

The aim of this section is to understand the Jenkins pipeline to deploy DVNA and solve the points 2-8 in [Problem Statement](problem_statements.md) under Task 1.


### **Jenkins Pipeline**

Jenkins is a continuous integration server which has the ability to support the automation of software development processes. It can be used to create several automation jobs and run them as a pipeline. Jenkins pipelines are made up of multiple stages (jobs/events) that allow you to build, test and deploy applications. Every stage has some sort of dependency on at least one or more stages in a pipeline. In DevSecOps, security is integrated into the pipeline by performing static and dynamic analysis using various SAST and DAST tools.

To start creating a new pipeline in Jenkins, login to the Jenkins web interface by typing `jenkins_server_ip:8080` in the url. Follow the steps below:

1. On the left hand side of the `Dashboard` page, click `New Item` to create a new project or pipeline. Provide a name for your pipeline (say `DVNA_pipeline`), select `Pipeline` as the project type and click `OK`.

2. You will then be redirected to the `Configure` page for your newly created project. 

    - Under the `General` section-  
    Check the `Discard Old Builds` option if you don't want to keep the console output, archived artifacts, and any other metadata related to older builds. They take up unnecessary disk space on the server. You can then specify the max number of days to keep builds and how many builds at max should be kept.

    - Under the `Pipeline` section-  
    Select `Pipeline script from SCM` and the SCM as `Git`. You will then have to specify your GitHub URL where your file containing details about the pipeline (aka Jenkinsfile) resides. There is an option to specify the Git branch and Jenkinsfile location in the repository.  
    **Note:** GitHub has deprecated the use of username and passwords for cloning Git repositories. You must instead use access tokens, which you can create from your GitHub account settings. The URL format should be like this-

            https://<access_token>@github.com/<github_username>/<repo>

    - Click `Save` to save and apply the configurations.

3. To initiate build, Go to `Dashboard` -> `DVNA_Pipeline` -> `Build Now`

### **Jenkinsfile**

A Jenkinsfile is a text file that contains the definition of a Jenkins Pipeline.

```bash
pipeline {
  agent any
  
  stages {
    stage ('Initialization') {
      steps {
        sh 'echo "Starting the build!"'
      }
    }
    
    stage ('Build') {
      environment {
        MYSQL_USER="dvna"
        MYSQL_DATABASE="dvna"
        MYSQL_PASSWORD=<PASSWORD>
        MYSQL_RANDOM_ROOT_PASSWORD=<ROOT_PASSWORD>
        MYSQL_HOST="mysql-db"
        MYSQL_PORT=3306
      }
      steps {
        sh 'echo "MYSQL_USER=$MYSQL_USER\nMYSQL_DATABASE=$MYSQL_DATABASE\nMYSQL_PASSWORD=$MYSQL_PASSWORD\nMYSQL_RANDOM_ROOT_PASSWORD=$MYSQL_RANDOM_ROOT_PASSWORD\nMYSQL_HOST=$MYSQL_HOST\nMYSQL_PORT=$MYSQL_PORT" > ~/vars.env'
        sh 'docker run --rm -d --name dvna-mysql --env-file ~/vars.env mysql:5.7 tail -f /dev/null'
        sh 'docker run --rm -d --name dvna-app --env-file ~/vars.env --link dvna-mysql:mysql-db -p 9090:9090 appsecco/dvna'
        sh 'docker cp dvna-app:/app/ ~/ && mkdir ~/reports && chmod 777 ~/reports'        
      }
    } 
       

    stage('SAST and DAST Scans') {
      ........
    }
    

    stage ('Remove DVNA from Jenkins') {
      steps {
        sh 'rm -rf ~/app'
        sh 'docker stop dvna-app && docker stop dvna-mysql'
        sh 'docker rmi appsecco/dvna && docker rmi mysql:5.7'
      }
    }
    
    stage ('Deploy DVNA to Production') {
      steps {
        sh 'ssh -o StrictHostKeyChecking=no tariq@192.168.56.102 "docker stop dvna-app && docker stop dvna-mysql && docker rm dvna-app && docker rm dvna-mysql && docker rmi appsecco/dvna || true"'
        sh 'scp ~/vars.env tariq@192.168.56.102:~/'
        sh 'ssh -o StrictHostKeyChecking=no tariq@192.168.56.102 "docker run -d --name dvna-mysql --env-file ~/vars.env mysql:5.7 tail -f /dev/null"'
        sh 'ssh -o StrictHostKeyChecking=no tariq@192.168.56.102 "docker run -d --name dvna-app --env-file ~/vars.env --link dvna-mysql:mysql-db -p 9090:9090 appsecco/dvna"'
      }
    }

  }
}
```

Components of the Jenkinsfile-  

- `pipeline` - Constitutes the entire definition of the pipeline.  
- `agent` - Used to choose the way the Jenkins instance(s) are used to run the pipeline. The `any` keyword defines that Jenkins should allocate any available agent (an instance of Jenkins/a slave/the master instance) to execute the pipeline.  
- `stages` - Consists of all the stages/jobs to be performed during the execution of the pipeline.  
- `stage` - Specify the task to be performed.  
- `steps` - Defines actions to be performed within a particular stage.  
- `sh` - Used to execute shell commands.

### **CI/CD Pipeline for Deploying DVNA**

The pipeline is divided into various stages based on the operations being performed. They are as follows:

**Initialization**

This is the first stage in the pipeline and is used just to indicate the start of the build.

**Build**

In this stage, `vars.env` file is created as it contains the environment variables for `dvna-mysql` db container. Two containers, `dvna-app` and `dvna-mysql`, are then run and the application code is copied into the hosts `jenkins` home directory (in my case, `/var/lib/jenkins`).

**SAST and DAST Scans**

All the stages following the `Build` stage, excluding the last two stages, are for performing SAST and DAST on the application. The scans are performed on the application running on the Jenkins VM and their output reports are stored in a folder named `reports` in `jenkins` home directory.  

**Note:** Most of the scans such as like NodeJsScan, AuditJs, JSHint, etc. return a non-zero exit code, even on successful completion. Jenkins considers non-zero status code as `FAILED` and stops the build. To overcome this, you can add either of the following at the end of the scan commands to give a `0` status code.  
```bash
<scan command> || true 
OR
<scan command>; echo $? > /dev/null
```

**Remove DVNA from Jenkins**

After the scans are complete, the containers running in Jenkins VM are stopped and removed. Since we're working with a containerized application (DVNA), we need to perform tests on the latest available image on DockerHub. Hence, we remove the existing local  `appsecco/dvna` docker image to avoid running a container with older release of the application image. On the other hand, you don't need to remove the `mysql:5.7` image, since we require v5.7 and not the latest version.

**Deploy DVNA to Production**

Finally, operations are performed on the Production VM over SSH (configured in the section titled `SSH Connection Between VMs`). `vars.env` file is copied into Production server, and the two containers; `dvna-app` and `dvna-mysql`, are run. The application is now successfully deployed!