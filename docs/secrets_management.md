### **Objective**

The aim of this section is to set up a secrets management service on the cloud to segregate the app and the associated secrets required for it to function and solve the 11th point of the [Problem Statement](problem_statements.md) under Task 1.

---

### **Secrets Management**

Secrets management refers to the tools and methods for managing digital authentication credentials (secrets), including passwords, keys, APIs, and tokens for use in applications, services, privileged accounts. They also provide additional services such as rotation of secrets.

---

### **AWS Secrets Manager**

AWS Secrets Manager is a secrets management service that helps you protect access to your applications, services, and IT resources. It centralizes storage of secrets and enables you to rotate, manage, and retrieve database credentials, API keys, and other secrets throughout their lifecycle. An application running within an Amazon VPC can communicate with the Secrets Manager to retrieve reqired credentials. 

**Create Secrets**

1. Login to AWS Secrets Manager Console
2. Click on `Store new secret`
3. Select `Other Types of Secrets` and specify the secrets as key value pairs. Then, choose the default encryption key - `DefaultEncryptionKey` to encrypt the secrets, and click `Next`.
4. Give a name for the secret (Eg. `/dvna/username`). The hierarchy allows you to group your secrets and maintain them better. You can add description, tags, etc. here if you want. Click `Next` and finish from the `Review Page`.

**Retrieve Secrets**

I'll be using AWS CLI to retrieve db configurations. As a best practice, do not use the AWS account root user for any task where it's not required. I followed this [documentation](https://docs.aws.amazon.com/IAM/latest/UserGuide/getting-started_create-admin-group.html) to create a new IAM user for each person that requires administrator access.

Next, I followed the official [documentation](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html) to install the latest version of aws-cli on Jenkins Master instance. 

```bash
sudo apt update && sudo apt install unzip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

After complete installation of aws-cli, I followed their official [documentation](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html) to configure basic settings required by aws-cli to interact with AWS. When running a `aws configure`, you'll be prompted to enter:  

1. Access key ID  
2. Secret access key  
3. AWS Region  
4. Output format  

Retrieve the db configurations stored in `dvna/database/mysql` by running the following command.  
**Note:** I used the `sed` command to replace certain charaters in the `aws` command output to write to `vars.env` file in the required format.

```bash
aws secretsmanager get-secret-value --secret-id dvna/database/mysql --query SecretString --version-stage AWSCURRENT --output text | sed -e 's/:/=/g' -e 's/{//g' -e 's/}//g' -e 's/,/\n/g' -e 's/"//g' > vars.env
```


