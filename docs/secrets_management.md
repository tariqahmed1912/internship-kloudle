### **Objective**

The aim of this section is to set up a secrets management service on the cloud to segregate the app and the associated secrets required for it to function and solve the 11th point of the [Problem Statement](problem_statements.md) under Task 1.

---

### **Secrets Management**

Secrets management refers to the tools and methods for managing digital authentication credentials (secrets), including passwords, keys, APIs, and tokens for use in applications, services, privileged accounts. They also provide additional services such as rotation of secrets.

---

### **AWS Secrets Manager**

AWS Secrets Manager is a secrets management service that helps you protect access to your applications, services, and IT resources. It centralizes storage of secrets and enables you to rotate, manage, and retrieve database credentials, API keys, and other secrets throughout their lifecycle. An application running within an Amazon VPC can communicate with the Secrets Manager to retrieve reqired credentials. 

#### **Create Secrets**

1. Login to AWS Secrets Manager Console
2. Click on `Store new secret`
3. Select `Other Types of Secrets` and specify the secrets as key value pairs. Instead of creating multiple secrets with each individual key-value pair, I added all the key value-pairs in one secret. 
4. Choose the default encryption key - `DefaultEncryptionKey` to encrypt the secrets, and click `Next`.
5. Give a name for the secret (I kept `/dvna/db/mysql`). The hierarchy allows you to group your secrets and maintain them better. You can add description, tags, etc. here if you want. Click `Next` and finish from the `Review Page`.

#### **Retrieve Secrets**

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

Retrieve the db configurations stored as `dvna/db/mysql` in Secrets Manager by running the following command.  
**Note:** I used the `sed` command to replace certain charaters in the `aws` command output to write to `vars.env` file in the required format.

```bash
aws secretsmanager get-secret-value --secret-id dvna/db/mysql --query SecretString --version-stage AWSCURRENT --output text | sed -e 's/:/=/g' -e 's/{//g' -e 's/}//g' -e 's/,/\n/g' -e 's/"//g' > vars.env
```

#### **AWS Secrets Manager Pipeline**

In the `Build` stage of Jenkins pipeline, remove the environment variables and the 1st shell command. Add another stage prior to the `Build` stage as given below.

```bash
stage ('Retrieve DB Configuration - AWS Secrets Manager') {
  steps {
    sh "aws secretsmanager get-secret-value --secret-id dvna/db/mysql --query SecretString --version-stage AWSCURRENT --output text | sed -e 's/:/=/g' -e 's/{//g' -e 's/}//g' -e 's/,/\n/g' -e 's/\"//g' > vars.env"
  }
}
```

---

### **HashiCorp Vault**

HashiCorp Vault is a secrets management tool specifically designed to control access to sensitive credentials in a low-trust environment. It can be used to store sensitive values and at the same time dynamically generate access for specific services/applications on lease. It is a more generic secrets manager as its not specific to a particular cloud vendor.

#### **Configuring Vault**

First, Install Vault. I followed the official [documentation](https://learn.hashicorp.com/tutorials/vault/getting-started-install?in=vault/getting-started) as its very concise and easy to follow.

Next, I followed the `Deploy Vault` section in the documentation.  
Create the Vault configuration in the file `config.hcl`.

```bash
disable_cache = true
disable_mlock = true

storage "raft" {
  path    = "./vault/data"
  node_id = "node1"
}

listener "tcp" {
  address     = "127.0.0.1:8200"
  tls_disable = "true"
}

api_addr = "http://127.0.0.1:8200"
cluster_addr = "http://127.0.0.1:8201"
ui = true
```

Set the `VAULT_ADDR` to enable the Vault client to talk to the Vault server. Vault CLI determines which Vault servers to send requests using the `VAULT_ADDR` environment variable. Create a directory for raft storage to use.

```bash
export VAULT_ADDR='http://127.0.0.1:8200'
mkdir -p ./vault/data
```

Start the Vault server and set the -config flag to point to the configuration file created prior.

```bash
vault server -config=config.hcl
```

To initialize Vault, use `vault operator init`. It outputs two incredibly important pieces of information: the unseal keys and the initial root token. Store these keys somewhere as you'll be needing them later on.  

```bash
$ vault operator init

Unseal Key 1: xrrxTkC/v89nFmDLaXDYvkqoZKor/uqPWotlbStiHXTU
Unseal Key 2: YWZG/4xnL77YOxqxMqV/uvhsmSL/mMeJ8mOxxMjETa8C
Unseal Key 3: aLst0tPFejnuGzpK8O3ru5oonWf0P+Y/WHOe91/2mTrR
Unseal Key 4: as8e7wV7Rli96onEi1XnIFWtj4NBG1OOXLl4hx8re6R3
Unseal Key 5: i8F61xHsazlp+MGpZxJTqDx5n+L7ZwixankeIBWLSd7e

Initial Root Token: s.pXT2YENqxxfJcffeks2bj6rt
```

Every initialized Vault server starts in the sealed state. From the configuration, Vault can access the physical storage, but it can't read any of it because it doesn't know how to decrypt it. The process of teaching Vault how to decrypt the data is known as unsealing the Vault.  To unseal the Vault, you must have the threshold number of unseal keys. In the output above, notice that the "key threshold" is 3. This means that to unseal the Vault, you need 3 of the 5 keys that were generated. Run `vault operator unseal` command thrice, passing one of the 5 previously generated unseal keys each time.

```bash
vault operator unseal <UNSEAL_KEY_1>
vault operator unseal <UNSEAL_KEY_2>
vault operator unseal <UNSEAL_KEY_3>
```

Finally, authenticate with the initial root token (included in the output with the unseal keys).  
**Note:** As a root user, you can reseal the Vault with `vault operator seal`. 

```bash
vault login <ROOT_TOKEN>
```

Enables Key/Value version2 secrets engine (kv-v2) at the path `dvna/` to store all MySQL db configs there. 

```bash
vault secrets enable -path=dvna kv
```

Store MySQL DB config (secrets) in `dvna/mysql`.  
**Note:** We're writing all secrets in a single `vault kv put` command because if we write a key-value pair one by one, it doesnt append it to the secrets list. It instead overrides it.

```bash
vault kv put dvna/mysql MYSQL_USER="dvna" MYSQL_DATABASE="dvna" MYSQL_PASSWORD="passw0rd" MYSQL_RANDOM_ROOT_PASSWORD="yes" MYSQL_HOST="mysql-db" MYSQL_PORT=3306
```

Retrieve the db configurations stored as `dvna/mysql` in Vault by running the following command.  
**Note:** I used the `sed` command to replace certain charaters in the `vault kv put` command output to write to `vars.env` file in the required format.

```bash
vault kv get -format=json -field=data dvna/mysql | sed -e 's/:/=/g' -e 's/{//g' -e 's/}//g' -e 's/,//g' -e 's/\"//g' -e 's/[[:blank:]]\+//g' > vars.env
```

#### **Vault Pipeline**

In the `Build` stage of Jenkins pipeline, remove the environment variables and the 1st shell command. Add another stage prior to the `Build` stage as given below.

```bash
stage ('Retrieve DB Configuration - Vault') {
    steps {
        sh 'ssh -o StrictHostKeyChecking=no jenkins@18.222.213.198 "export VAULT_ADDR=http://127.0.0.1:8200"'
        sh 'ssh -o StrictHostKeyChecking=no jenkins@18.222.213.198 "vault operator unseal <UNSEAL-KEY-1> && vault operator unseal <UNSEAL-KEY-2> && vault operator unseal <UNSEAL-KEY-3>"'
        sh "ssh -o StrictHostKeyChecking=no jenkins@18.222.213.198 'vault kv get -format=json -field=data dvna/mysql | sed -e \'s/:/=/g\' -e \'s/{//g\' -e \'s/}//g\' -e \'s/,//g\' -e \'s/\"//g\' -e \'s/[[:blank:]]\+//g\' > vars.env'"
        sh 'ssh -o StrictHostKeyChecking=no jenkins@18.222.213.198 "vault operator seal"'
        sh  'scp jenkins@18.222.213.198:~/vars.env ~/'
    }
}
```
