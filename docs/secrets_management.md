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
3. Select `Other Types of Secrets` and specify the secrets as key value pairs. Instead of creating multiple secrets with each individual key-value pair, I added all the key value-pairs in one secret. 
4. Choose the default encryption key - `DefaultEncryptionKey` to encrypt the secrets, and click `Next`.
5. Give a name for the secret (I kept `/dvna/databse/mysql`). The hierarchy allows you to group your secrets and maintain them better. You can add description, tags, etc. here if you want. Click `Next` and finish from the `Review Page`.

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

Retrieve the db configurations stored as `dvna/database/mysql` in Secrets Manager by running the following command.  
**Note:** I used the `sed` command to replace certain charaters in the `aws` command output to write to `vars.env` file in the required format.

```bash
aws secretsmanager get-secret-value --secret-id dvna/database/mysql --query SecretString --version-stage AWSCURRENT --output text | sed -e 's/:/=/g' -e 's/{//g' -e 's/}//g' -e 's/,/\n/g' -e 's/"//g' > vars.env
```

**Pipeline**

In the `Build` stage of Jenkins pipeline, remove the environment variables and the 1st shell command. Add another stage prior to the `Build` stage as given below.

```bash
stage ('Retrieve DB Configuration') {
  steps {
    sh "aws secretsmanager get-secret-value --secret-id dvna/database/mysql --query SecretString --version-stage AWSCURRENT --output text | sed -e 's/:/=/g' -e 's/{//g' -e 's/}//g' -e 's/,/\n/g' -e 's/\"//g' > vars.env"
  }
}
```

### **HashiCorp Vault**

HashiCorp Vault is a secrets management tool specifically designed to control access to sensitive credentials in a low-trust environment. It can be used to store sensitive values and at the same time dynamically generate access for specific services/applications on lease. It is a more generic secrets manager as its not specific to a particular cloud vendor.

**Configuring Vault**

First, Install Vault. I followed the official [documentation](https://learn.hashicorp.com/tutorials/vault/getting-started-install?in=vault/getting-started) as its very easy to follow.

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

Set the vault address and create a directory for raft storage to use.

```bash
export VAULT_ADDR='http://127.0.0.1:8200'
mkdir -p ./vault/data
```

Start the vault server and set the -config flag to point to the configuration file created prior.

```bash
vault server -config=config.hcl
```

To initialize Vault use `vault operator init`. It outputs two incredibly important pieces of information: the unseal keys and the initial root token. This is the only time ever that all of this data is known by Vault, and also the only time that the unseal keys should ever be so close together. 

```bash
vault operator init
```

Every initialized Vault server starts in the sealed state. From the configuration, Vault can access the physical storage, but it can't read any of it because it doesn't know how to decrypt it. The process of teaching Vault how to decrypt the data is known as unsealing the Vault.  To unseal the Vault, you must have the threshold number of unseal keys. In the output above, notice that the "key threshold" is 3. This means that to unseal the Vault, you need 3 of the 5 keys that were generated. Run the following command thrice and passing one of the 5 previously generated unseal keys each time.

```bash
vault operator unseal 
```

Finally, authenticate as the initial root token (it was included in the output with the unseal keys).  
**Note:** As a root user, you can reseal the Vault with `vault operator seal`. 

```bash
vault login <ROOT_TOKEN>
```

Enable the KV secrets engine.

```bash
vault secrets enable -path=secret/ kv
```

Create a secret and retrieve it.

```bash
vault kv put secret/hello foo=world
vault kv get -format=json secret/hello
```