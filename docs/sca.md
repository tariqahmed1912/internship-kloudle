## **Objective**
In this section, we will perform Software Composition Analysis (SCA) of DVNA.

About SCA

- Software composition analysis (SCA) identifies all the open source in a codebase and maps that inventory to a list of current known vulnerabilities.
- It helps identify vulnerabilities in open source code used.

### **OWASP Dependancy Check**

[OWASP Dependency-Check](https://owasp.org/www-project-dependency-check/) is an SCA tool that attempts to detect publicly disclosed vulnerabilities contained within a project’s dependencies. It does this by determining if there is a Common Platform Enumeration (CPE) identifier for a given dependency. If found, it will generate a report linking to the associated CVE entries.

