## **Objective**

The aim of this section is to perform code linting and generate a code quality report.

### **Code Linting**

Code linting is the automated checking of your source code for programmatic and stylistic errors. It's great for identifying violations of standard rules and is the most basic form of static analysis. Lint tools (aka linters) help accelerate developement process and reduce costs by finding errors earlier.

### **JSHint**

JSHint is a static code analysis tool that helps to detect errors and potential problems in your JavaScript code. It scans a program written in JavaScript and reports about commonly made mistakes and potential bugs. The potential problem could be a syntax error, a bug due to an implicit type conversion, a leaking variable, or something else entirely.

To get started with JSHint, I followed this [documentation](https://jshint.com/docs/). Since I'm performing a test on a  NodeJs application (DVNA), I installed JSHint using NPM.

``bash
npm install -g jshint
```

Run the `jshint` command with the application/project directory to scan all `.js` files. To generate a report, we can either just redirect the output stream to a file or make use of `reporters` to create XML or JSON reports. 

```bash
jshint ~/app > ~/report/jshint-report
```

The above command will scan all the files in the `/app` directory. To restrict the scan to only .js files, use a `find` command. We will further exclude all files in the `node_modules` directory.

```bash
jshint $(find ~/app -type f -name "*.js" -o -name "*.ejs" | grep -v node_modules) > ~/reports/jshint-report
```

I added the following stage in the Jenkins pipeline to perform code linting on DVNA

```bash
stage ('JSHint Analysis') {
  steps {
    sh 'jshint $(find ~/app -type f \( -name "*.js" -o -name "*.ejs" \) | grep -v node_modules) > ~/reports/jshint-report; echo $? > /dev/null'
  }
}
```

### **ESLint**


### **NOTE**

A lot of scans like NodeJsScan, AuditJs, JSHint, etc. return a non-zero exit code, even on successful completion. Jenkins considers non-zero status code as `FAILED` and stops the build. To overcome this, you can add either of the following at the end of the scan commands. Both of these will give a `0` status code.

```bash
<scan command> || true 
OR
<scan command>; echo $? > /dev/null
```


