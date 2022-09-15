## 1. Learning outcome
You set up environments and tools which support your chosen software development process. You provide governance for all stakeholders’ goals. You aim for as much automation as possible, to enable short release times and high software quality.

## 2. Pipeline 
To automate as much of the development process as possible and decrease turnaround time, I want to setup a pipeline using Github actions to do the following; Lint the code, run the unit tests, run a Sonarcloud analysis, create a new docker image (on main push).

### 2.1 Code Lint
This [workflow](https://github.com/github/super-linter) runs on every push and lints the code. It checks for programmatic and stylistic errors. By using Sonarlint in the IDE hopefully most errors are already taken care of before the push.

### 2.2 PHPUnit
[PHPUnit](https://phpunit.de/) is a programmer-oriented trsting framework for PHP. Laravel offer out of the box support for PHPunit testing. A test workflow can be used to prevent merging or depolying broken commits.

### 2.3 Sonarcloud
To check the quality of my code, I added a [Sonarcloud](https://sonarcloud.io/) workflow to my repositories. Sonarcloud checks the codebase not only for code smells and bugs, but also security hotspots and vulnerabilities. Using Sonarcloud can give a team a great insight into their code quality and can help prevent merging broken branches.


## 3. Sonarcloud

## 4. Branch Protection

## 5. Status badges

## 6. Postman

## 7. Reflection
