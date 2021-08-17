## **Objective**

The aim of this section is to perform code linting and generate a code quality report.

### **Code Linting**

Code linting is the automated checking of your source code for programmatic and stylistic errors. It's great for identifying violations of standard rules and is the most basic form of static analysis. Lint tools (aka linters) help accelerate developement process and reduce costs by finding errors earlier.

### **JSHint**

JSHint is a static code analysis tool that helps to detect errors and potential problems in your JavaScript code. It scans a program written in JavaScript and reports about commonly made mistakes and potential bugs. The potential problem could be a syntax error, a bug due to an implicit type conversion, a leaking variable, or something else entirely.

To get started with working with JSHint, I followed this [documentation](https://jshint.com/docs/). To test NodeJs application (DVNA), install JSHint using NPM.

``bash
npm install -g jshint
```

Run the `jshint` command with the application/project directory to scan all `.js` files. To generate a report, we can either just redirect the output stream to a file or make use of `reporters` to create XML or JSON reports. 

```bash
jshint ~/app > ~/report/jshint-report
```


### **ESLint**



