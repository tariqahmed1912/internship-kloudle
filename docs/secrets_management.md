### **Objective**

The aim of this section is to set up a secrets management service on the cloud to segregate the app and the associated secrets required for it to function and solve the 11th point of the [Problem Statement](problem_statements.md) under Task 1.

---

### **Secrets Management**

Secrets management refers to the tools and methods for managing digital authentication credentials (secrets), including passwords, keys, APIs, and tokens for use in applications, services, privileged accounts. They also provide additional services such as rotation of secrets.

---

### **AWS Secrets Manager**

AWS Secrets Manager centralizes storage of secrets so applications dont need to take on the added responsibility of storing secrets securely. It helps implement best practices such as periodic rotation of secrets/keys.