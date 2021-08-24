## **Objective**

The aim of this section is to generate a Software Bill of Materials (SBoM) for DVNA and generate a report and solve the 7th point of the [Problem Statement](problem_statements.md) under Task 1.

### **SBoM**

A Software Bill of Materials (SBOM) is a list of all the open source and third-party components present in a codebase. It lists the licenses that govern those components, the versions of the components used in the codebase, and their patch status.


### **CycloneDX**

The CycloneDX module for Node.js creates a valid CycloneDX SBoM containing an aggregate of all project dependencies. It's a lightweight SBoM specification that is easily created, human and machine readable, and simple to parse. It also comes in a variety of implementations to serve projects using different stacks such as Python, Maven, .NET, etc. For my use case, I stuck with the NPM package as DVNA only utilizes Nodejs.

### **Generating SBoM for DVNA**

Following the [official documentation](https://github.com/CycloneDX/cyclonedx-node-module), I installed CycloneDX using NPM.

```bash
npm install -g @cyclonedx/bom
```

To generate a SBoM report, use the `-o` flag and specify the filename and its format. The report can be either XML  or JSON.  
```bash
cyclonedx-bom -o sbom.xml
```

### **SBoM Pipeline**

Add the following stage in the Jenkins pipeline for generating the SBoM.
  
```bash
stage ('Generating Software Bill of Materials') {
    steps {
        sh 'cd ~/app && cyclonedx-bom -o ~/reports/sbom.xml'
    }
}
```