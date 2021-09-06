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