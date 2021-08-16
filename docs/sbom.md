## **Objective**

The aim of this section is to generate a Software Bill of Materials (SBoM) for DVNA and generate a report.

About SBOM

A Software Bill of Materials (SBOM) is a list of all the open source and third-party components present in a codebase. It lists the licenses that govern those components, the versions of the components used in the codebase, and their patch status.


### **CycloneDX**

The CycloneDX module for Node.js creates a valid CycloneDX SBoM containing an aggregate of all project dependencies. It's a lightweight SBoM specification that is easily created, human and machine readable, and simple to parse.

Installation of cyclonedx using npm.

```bash
npm install -g @cyclonedx/bom
```

To generate a SBoM report, use the `-o` flag and specify the filename and its format. The report can be either XML  or JSON.

```bash
cyclonedx-bom -o bom.json
```