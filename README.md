# Gitlab CI course notes 💻📕🚀

## [GitLab CI: Pipelines, CI/CD and DevOps for Beginners](https://www.udemy.com/course/gitlab-ci-pipelines-ci-cd-and-devops-for-beginners/):🐌
- 👍Pro: examples of code that are very easy to understand or reuse. PDF file will all lecture notes👌 
- 👎Cons: Assignments are useful, but boring. Examples (java app, web-page) are not so much relevant for MLOps. 
- ⌚Duration: 5.5 hours
 
  
## CI/CD Pipeline steps 🐛🍎🚀🔨
- Code 👨‍💻
- CI Pipeline 📦:
  - build
  - code quality
  - tests
  - package
- CD Pipeline 🚀:
  - review/test
  - staging
  - production  

## What is CI / CD? 👨‍💻📦🚀
### CI as Continuous Integration: code is integrated with other developers all the time 📦
The goal: to build a package that can be deployed 
- the build step is still working?
- unit tests still work?
- whatt about coding guidelines? (can be mostly automated)

### CD as Continuous Delivery: always done after CI 🚀
The goal: to take the package created by the CI pipeline and to test it further 
- can teh package be installed? (by actually installing the package on a system similar to production)
- running additional tests to check if the package integrates with other systems
- after a manual check/decision, the package can be installed on a production system as well

### CD as Continuous Deployment: a step further, automatically installs every package to production
- the package must first go through all previous stages successfully
- no manual intervention is required

## Gitlab CI basics 🔨📦

### Jobs and pipelines 👷🏭
Pipeline configured and described in `.gitlab-ci.yml` file. Gitlab pipeline consists of jobs. GitLab will execute pipelines with GitLab Runner (runs similarly to a terminal). By default, Gitlab does not save anything. To run a job Gitlab:
- delegate job to a Gitlab Runner
- download and start Docker image
- clone the repo
- install any required dependencies
- run the actual step
- save the result (if needed) 

### Validate `.gitlab-ci.yml` with CI Lint 📄 
Not sure that your `.gitlab-ci.yml` will work? Then test the validity of your GitLab CI/CD configuration before committing the changes with CI Lint tool: navigate to CI/CD > Pipelines or CI/CD > Jobs in your project and click CI lint. This tool checks for syntax and logical errors by default, and can simulate pipeline creation to try to find more complicated issues as well.

### Caches 💾
Pipeline execution is too slow? Use caches! The best practice is caching the external project dependencies that are not stored in Git and that need to be downloaded. The cache can be used locally (on a job level) or globally

Sometimes caches misbehave: to delete the cache go from your project > Pipelines > Clear Runner Caches

🍎Idea: (scheduled) pipeline of only one step that updates cache 

### Variables 💾
[Predefined variables](https://docs.gitlab.com/ee/ci/variables/predefined_variables.html) and environment variables.

Different environments and destroying environments.

### Example

```
image: node:10

stages:
  - build
  - test
  - deploy
  - deployment tests

cache:
  key: ${CI_COMMIT_REF_SLUG}
  paths:
    - node_modules/

build website:
  stage: build
  script:
    - echo $CI_COMMIT_SHORT_SHA
    - npm install
    - npm install -g gatsby-cli
    - gatsby build
    - sed -i "s/%%VERSION%%/$CI_COMMIT_SHORT_SHA/" ./public/index.html
  artifacts:
    paths:
      - ./public

test artifact:
  image: alpine
  stage: test
  script:
    - grep -q "Gatsby" ./public/index.html

test website:
  stage: test
  script:
    - npm install
    - npm install -g gatsby-cli
    - gatsby serve &
    - sleep 3
    - curl "http://localhost:9000" | tac | tac | grep -q "Gatsby"

deploy to surge: 
  stage: deploy
  script:
    - npm install --global surge
    - surge --project ./public --domain instazone.surge.sh

test deployment:
  image: alpine
  stage: deployment tests
  script:
    - apk add --no-cache curl
    - curl -s "https://instazone.surge.sh" | grep -q "Hi people"
    - curl -s "https://instazone.surge.sh" | grep -q "$CI_COMMIT_SHORT_SHA"
```
