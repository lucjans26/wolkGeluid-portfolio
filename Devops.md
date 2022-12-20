## 1. Learning outcome
You set up environments and tools which support your chosen software development process. You provide governance for all stakeholders’ goals. You aim for as much automation as possible, to enable short release times and high software quality.

## 2. Pipeline 
To automate as much of the development process as possible and decrease turnaround time, I want to setup a pipeline using Github actions to do the following; Lint the code, run the unit tests, run a Sonarcloud analysis, create a new docker image (on main push).

### 2.1 Code Lint
This [workflow](https://github.com/github/super-linter) runs on every push and lints the code. It checks for programmatic and stylistic errors. By using Sonarlint in the IDE hopefully most errors are already taken care of before the push.

```yaml
name: Lint Code Base

on: push

jobs:
  lint:
    name: Lint Code Base
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2

      - name: Run linter
        uses: github/super-linter@v4.7.3
        env:
          VALIDATE_ALL_CODEBASE: false
          DEFAULT_BRANCH: main
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
``` 

### 2.2 PHPUnit
[PHPUnit](https://phpunit.de/) is a programmer-oriented trsting framework for PHP. Laravel offer out of the box support for PHPunit testing. A test workflow can be used to prevent merging or depolying broken commits.

```yaml
on: push
name: Test
jobs:
  phpunit:
    runs-on: ubuntu-latest
    container:
      image: kirschbaumdevelopment/laravel-test-runner:8.1

    services:
      mysql:
        image: mysql:latest
        env:
          MYSQL_ROOT_PASSWORD: password
          MYSQL_DATABASE: test
        ports:
          - 33306:3306
        options: --health-cmd="mysqladmin ping" --health-interval=10s --health-timeout=5s --health-retries=3

    steps:
      - uses: actions/checkout@v1
        with:
          fetch-depth: 1

      - name: Install composer dependencies
        run: composer install --no-scripts

      - name: Prepare Laravel Application
        run: cp .env.ci .env && php artisan key:generate

      - name: Run Testsuite
        run: vendor/bin/phpunit tests/
```

### 2.3 Sonarcloud
To check the quality of my code, I added a [Sonarcloud](https://sonarcloud.io/) workflow to my repositories. Sonarcloud checks the codebase not only for code smells and bugs, but also security hotspots and vulnerabilities. Using Sonarcloud can give a team a great insight into their code quality and can help prevent merging broken branches.

### 2.4 Docker
This final workflow automatically builds a new docker image based on the dockerfile created within the project. I created 2 different workflows with different purposes. The **nightly** build is built on every push to the "Development" branch. This build is not directly ready for release but may contain release candidates. The **latest** build is the build that is ready to be released. This build is pushed on every push to the "main" branch.

#### lucjans26/wolkgeluid-monolith:nightly
```yaml
on:
  push:
    branches:
      - Development

jobs:
  docker:
   name: Docker Nightly Image
   runs-on: ubuntu-latest
   steps:
     -
       name: Checkout
       uses: actions/checkout@v2
     -
       name: Set up Docker Buildx
       uses: docker/setup-buildx-action@v1
     -
       name: Login to DockerHub
       uses: docker/login-action@v1
       with:
         username: ${{ secrets.DOCKERHUB_USERNAME }}
         password: ${{ secrets.DOCKERHUB_PASSWORD }}
     -
        name: Build and push
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: lucjans26/wolkgeluid-monolith:nightly
```

#### lucjans26/wolkgeluid-monolith:latest
```yaml
on:
  push:
    branches:
      - main

jobs:
  docker:
   name: Docker Release Image
   runs-on: ubuntu-latest
   steps:
     -
       name: Checkout
       uses: actions/checkout@v2
     -
       name: Set up Docker Buildx
       uses: docker/setup-buildx-action@v1
     -
       name: Login to DockerHub
       uses: docker/login-action@v1
       with:
         username: ${{ secrets.DOCKERHUB_USERNAME }}
         password: ${{ secrets.DOCKERHUB_PASSWORD }}
     -
        name: Build and push
        uses: docker/build-push-action@v2
        with:
          context: .
          push: true
          tags: lucjans26/wolkgeluid-monolith:latest
```
## 3. Testing
Testing is an important part of the software engineering process because it helps ensure that a software application or system meets its requirements and functions as intended. In the context of continuous integration/continuous delivery (CI/CD), testing plays a crucial role because it helps to verify that code changes do not introduce new defects or break existing functionality. By performing testing at various stages of the CI/CD pipeline, developers can catch and fix issues early on, which helps to reduce the risk of errors and improve the overall quality of the software. Additionally, testing can help to ensure that the software is ready for deployment by verifying that it meets the necessary performance, security, and other requirements.

![image](https://user-images.githubusercontent.com/46562627/208777972-ed4e35c6-78f4-4c44-bb6f-84a229108f83.png)




### 3.1 Unit Testing
Unit testing is a testing technique that involves testing individual units or components (like for example methods, other business logic, or front-end units) of a software application in isolation from the rest of the application. It is important because it helps to ensure that individual units or components of the application are working as intended and are free of defects, which can help to improve the overall reliability and quality of the software.
Unit testing is usually the most earliest level of testing. It is quick and inexpensive to do.

I implemented unit testing by writing laravel unit tests which are run using [PhpUnit](https://phpunit.de/), a PHP testing framework. The unit tests are written using the AAA pattern to keep the tests uniform and make use of database transactions so that the production database can be used for the most consistent testing. After the tests end, the transactions are rolled back. It is also possible to use an in-memory database using Laravel. 

![image](https://user-images.githubusercontent.com/46562627/208780079-9155ea07-b33b-41b3-912e-7cf8d47f4245.png)

Later these unit tests can be run in combination with [Xdebug](https://xdebug.org/), a PHP extension which is mostly usefull for code coverage analysis to show which parts of your code base are executed when running unit tests with PHPUnit. However this requires a lot of setting up which I haven't been able to get working as of now.

### 3.2 Integration Testing
Integration testing is a software testing technique that involves testing how different units or components of a software application work together. It is important because it helps to ensure that the integration between different units or components is working as intended, and that the application as a whole is functioning correctly. This is especially important because issues with integration can be difficult to detect and fix if they are not caught early on in the development process.
It is more complete than unit testing, as it tests the integration between different parts of the software. However, it can be more time-consuming and resource-intensive to perform than unit testing, as it may require setting up and configuring multiple units or components.

### 3.3 End-to-end testing
End-to-end (E2E) testing (or acceptance testing) is a software testing technique that involves testing a complete workflow of a software application, from start to finish, to ensure that it is functioning correctly. It is important because it helps to ensure that the software application is working correctly from the user's perspective, and that all the various components and systems are working together seamlessly. By performing end-to-end testing, a team can catch and fix issues that may not have been detected by unit or integration testing, and can ensure that the software is completely ready for deployment. This is especially important for software applications thathave a high level of user interaction, as defects or issues with these types of applications can have serious consequences.
This is the most complex and most expensive form of testing, and should really be saved the the most critical components.

E2E testing was done using [Cypress](https://www.cypress.io/), a fast, easy and reliable javascript automated testing framework. Cypress makes it really easy to install and set up.

![image](https://user-images.githubusercontent.com/46562627/208780776-be167fa2-25ad-4a8c-8848-52026b25289c.png)

Using the experimental [Cypress Studio](https://docs.cypress.io/guides/references/cypress-studio) you can really easily walk through your application while Cypress watches over your shoulder. It registers your every move and will be able to test them again in the future. If the flow fails of requests aren't succesfully completed (which in turn could also break the flow), the tests fail.

![image](https://user-images.githubusercontent.com/46562627/208781603-c420a3d7-45c5-4372-8367-48bdb568feec.png)

## 4. SonarCloud

## 5. Branch Protection
To prevent merging broken branches, I added branch protection rules. In this example the protection from the main branch requires any merge to pass the testing workflow and the Sonarcloud quality control gate. This means that all tests must pass and the coverage must be at least 80% (or other as defined in sonarcloud).

![image](https://user-images.githubusercontent.com/46562627/208779023-3aa92942-bc45-4d03-87a8-372a0c6038ca.png)

## 6. Status badges
To give the developers working on a project a quick insight into the workflow status of a project, [workflow status badges](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/adding-a-workflow-status-badge) can be added to the readme.md.

![image](https://user-images.githubusercontent.com/46562627/173389970-481b6e76-7721-4c95-9db4-08bd52f64e16.png)

## 7. Postman
During the development process, especially when building an API, a way to test manually can be really helpful using a client. However it saves a lot of time if every developer within a team has access to the same information to test with and that all endpoints are documented with example data and responses.
For this a team can be created in [Postman](https://www.postman.com/). Within this team everybody has access to previously made requests, saved requests and responses, and if documented well withon postman gives the team the ability to convert a collection straight into an API specification.

![image](https://user-images.githubusercontent.com/46562627/173391926-83919ecc-1b04-403e-a473-793c7ec077a3.png)

## 8. Reflection
I believe I have made significant efforts to set up environments and tools that support the software development process, and to automate as much of the development process as possible in order to increase the development lifecycle and improve software quality.

I have implemented a pipeline using GitHub Actions that includes several different steps, such as linting the code, running unit tests, performing Sonarcloud analysis, and creating new Docker images. These steps are designed to catch and address issues early on in the development process, which can help to ensure that the software is of high quality and free of defects.

In addition, I have implemented different workflows for different branches in order to build Docker images that are either ready for release or contain release candidates. This can help to ensure that the software is properly tested and ready for deployment.

Overall, I have put a lot of thought and effort into setting up a robust and effective software development process, and into automating as muchas possible in order to improve efficiency and quality. This is a valuable and important goal, and the work that I have done will help a lot in my upcoming internship and future as a software engineer.
