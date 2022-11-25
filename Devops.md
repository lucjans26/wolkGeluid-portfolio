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

## 3. SonarCloud

## 4. Branch Protection

## 4. Status badges
To give the developers working on a project a quick insight into the workflow status of a project, [workflow status badges](https://docs.github.com/en/actions/monitoring-and-troubleshooting-workflows/adding-a-workflow-status-badge) can be added to the readme.md.

![image](https://user-images.githubusercontent.com/46562627/173389970-481b6e76-7721-4c95-9db4-08bd52f64e16.png)

## 5. Postman
During the development process, especially when building an API, a way to test manually can be really helpful using a client. However it saves a lot of time if every developer within a team has access to the same information to test with and that all endpoints are documented with example data and responses.
For this a team can be created in [Postman](https://www.postman.com/). Within this team everybody has access to previously made requests, saved requests and responses, and if documented well withon postman gives the team the ability to convert a collection straight into an API specification.

![image](https://user-images.githubusercontent.com/46562627/173391926-83919ecc-1b04-403e-a473-793c7ec077a3.png)
## 7. Reflection
