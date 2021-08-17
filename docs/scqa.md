## **Objective**

The aim of this section is to analyse the quality of software code and generate a report.

### **SCQA**

A Software Bill of Materials (SBOM) is a list of all the open source and third-party components present in a codebase. It lists the licenses that govern those components, the versions of the components used in the codebase, and their patch status.


### **JSHint**

JSHint is a static code analysis tool that helps to detect errors and potential problems in your JavaScript code. It scans a program written in JavaScript and reports about commonly made mistakes and potential bugs. The potential problem could be a syntax error, a bug due to an implicit type conversion, a leaking variable, or something else entirely.

### **ESLint**

### **Generating SBoM for DVNA**

Installation of CycloneDX using NPM.

```bash
npm install -g @cyclonedx/bom
```

To generate a SBoM report, use the `-o` flag and specify the filename and its format. The report can be either XML  or JSON.  
```bash
cyclonedx-bom -o sbom.xml
```

I added a stage in the Jenkins pipeline for generating the SBoM.  
```bash
stage ('Generating Software Bill of Materials') {
    steps {
        sh 'cyclonedx-bom -o ~/reports/sbom.xml'
    }
}
```