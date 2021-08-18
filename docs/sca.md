## **Objective**

The aim of this section is to explain the software composition of DVNA using SCA tools in a Jenkins pipeline.


Prerequisites

-   An application (DVNA) running on Production Server.

### **SCA**

Software composition analysis (SCA) identifies all the open source in a codebase and maps that inventory to a list of current known vulnerabilities. It helps identify vulnerabilities in open source code (dependencies) used in code.


### **OWASP Dependancy-Check**

OWASP Dependency-Check is a software composition analysis (SCA) tool that detects publicly disclosed vulnerabilities contained within a project’s dependencies. It does this by determining if there is a Common Platform Enumeration (CPE) identifier for a given dependency. If found, it will generate a report linking to the associated CVE entries.

### **Performing SCA on DVNA**

To start working with Dependency Check, I followed the [official documentation](https://jeremylong.github.io/DependencyCheck/dependency-check-cli/index.html). First, download the Dependency-Check CLI tool and the associated GPG signature file.

```bash
wget -P ~/ https://github.com/jeremylong/DependencyCheck/releases/download/v6.2.2/dependency-check-6.2.2-release.zip

wget -P ~/ https://github.com/jeremylong/DependencyCheck/releases/download/v6.2.2/dependency-check-6.2.2-release.zip.asc
```

Next, extract the files from the dependency-check tool zip file.

```bash
unzip ~/dependency-check-6.2.2-release.zip
```

Perform the scan by specifying the path to the project, output report format and its location.

```bash
~/dependency-check/bin/dependency-check.sh --scan ~/app --out ~/report/dependency-check-report --format JSON --prettyPrint
```

**Note**: This scan is inclusive of Retire.js scan, NPM Audit scan, and Auditjs scan, to name a few.

### **SCA Pipeline**

I added the following stage in the Jenkins pipeline to perform SCA of DVNA.

```bash
stage ('OWASP Dependency-Check') {
    steps {
    sh '~/dependency-check/bin/dependency-check.sh --scan ~/app --out ~/report/dependency-check-report --format JSON --prettyPrint || true'
    }
}
```
