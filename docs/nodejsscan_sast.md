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
	
EXTRAAAA

### Jenkins Pipeline

SSH into server
Copy code to server
docker build
docker start
run nodejsscan
stop and remove docker container

Install Mkdocs

Auditjs SAST tool
