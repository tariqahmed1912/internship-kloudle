## **Objective**

The aim of this section is to perform code linting and generate a code quality report.

### **Code Linting**

Code linting is the automated checking of your source code for programmatic and stylistic errors. It's great for identifying violations of standard rules and is the most basic form of static analysis. Lint tools (aka linters) help accelerate developement process and reduce costs by finding errors earlier.

### **JSHint**

JSHint is a static code analysis tool that helps to detect errors and potential problems in your JavaScript code. It scans a program written in JavaScript and reports about commonly made mistakes and potential bugs. The potential problem could be a syntax error, a bug due to an implicit type conversion, a leaking variable, or something else entirely.

To get started with JSHint, I followed this [documentation](https://jshint.com/docs/). Since I'm performing a test on a  NodeJs application (DVNA), I installed JSHint using NPM.

```bash
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


### **ESLint**

ESLint is a tool for identifying and reporting on patterns found in ECMAScript/JavaScript code. Unlike JSHint, ESLint uses Espree for JavaScript parsing and an AST to evaluate patterns in code. It's completely pluggable, every single rule is a plugin and you can add more at runtime.

To get started with JSHint, I followed the [official documentation](https://eslint.org/docs/user-guide/getting-started). 

```bash
npm install -g eslint 
```
After installing eslint, eslint requires a `.eslintrc` configuration file which contains environment variables, rules and other extra config details. To create this file, navigate to your project folder and run `eslint --init`. You will be prompted a few questions to create the config file. The questions and my responses to them are given below.

```bash
✔ How would you like to use ESLint? · problems
✔ What type of modules does your project use? · commonjs
✔ Which framework does your project use? · none
✔ Does your project use TypeScript? · No
✔ Where does your code run? · browser
✔ What format do you want your config file to be in? · JSON
```

The contents of the new `.eslintrc.json` config file are:
```bash
{
    "env": {
        "browser": true,
        "commonjs": true,
        "es2021": true
    },
    "extends": "eslint:recommended",
    "parserOptions": {
        "ecmaVersion": 12
    },
    "rules": {
    }
}
```
**Note:** The `.eslintrc.*` config file is required everytime you want to run `eslint` code linting on a file or directory. While running the scans in a pipeline, its not possible to create the config file by running `eslint --init` as the command line waits for responses to file configuration questions. To work around this problem, copy the configuration details shown above in `<Jenkins-Home-Dir>/.eslintrc.json`. You can now specify `eslint` to use this config file during its execution.

To perform linting using eslint, run `eslint` command with the following flags;
`-c`, specify the config file to use;
`-f`, format of output report;
`--ext`, specify the extensions of files to be scannned;
`-o`, specify file to write report to

```bash
eslint -c ~/.eslintrc.json -f html --ext .js,.ejs -o ~/reports/eslint-report.html ~/app
```

### **Code Quality Pipeline**

I added the following stages in the Jenkins pipeline for performing code linting.

```bash
stage ('JSHint Analysis') {
  steps {
    sh 'jshint $(find ~/app -type f \( -name "*.js" -o -name "*.ejs" \) | grep -v node_modules) > ~/reports/jshint-report || true'
  }
}
stage ('ESLint Analysis') {
  steps {
    sh 'eslint -c ~/.eslintrc.json -f html --ext .js,.ejs -o ~/reports/eslint-report.html ~/app || true'
  }
}
```

