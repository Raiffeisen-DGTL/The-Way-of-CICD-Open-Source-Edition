# Pipeline Modules for Code Checking

## Description

This section contains a pipeline module for code checking with `SonarQube`, `lint` configurations, manifests, and more.

For `SonarQube`, there is an option to check code in `Java`, `Kotlin`, and with build systems `Maven` and `Gradle`.
You can use the `.sonarqube-java`/`.sonarqube-kotlin` templates, which support both build types and select the appropriate one using the `BUILD_TYPE` variable with values `maven` or `gradle`. Alternatively, you can use separate templates for `Maven` builds: `.sonarqube-java-maven`/`.sonarqube-kotlin-maven`, and `Gradle` builds: `.sonarqube-java-gradle`/`.sonarqube-kotlin-gradle`.
The `SONAR_CUSTOM_OPTIONS` variable has been added to the job templates for scanning Java code, in which additional parameters for SonarQube can be passed, for example:
```yaml
SONAR_CUSTOM_OPTIONS="-Dsonar.exclusions=**/test/** -Dsonar.coverage.exclusions=**/test/**"
```

You can scan any other code using `.sonarqube-scan`, as well as lint help charts using `.help-line` and Dockerfile using `.docker-lint`

`.docker-lint` can be run without additional settings, since the main ones are set by default:
```yaml
variables:
  DOCKERFILE: $CI_PROJECT_DIR/Dockerfile # Path to your Dockerfile
  FORMAT: tty # The output format for the results [tty | json | checkstyle | codeclimate | gitlab_codeclimate | gnu | codacy | sonarqube | sarif]
  THRESHOLD: info # Exit with failure code only when rules with a severity equal to or above THRESHOLD are violated. Accepted values: [error | warning | info | style | ignore | none]
  PATH_IN_REPORT: $DOCKERFILE # The file path referenced in the generated report. This only applies for the 'checkstyle' format and is useful when running Hadolint with Docker to set the correct file path.
  HADOLINT_OPTIONS: "" # Your custom hadolint settings
                          
```

To analyze the project (calculate test coverage), sonarqube requires the test results that are supposed to be obtained in the previous step, in a separate job.

To use `SonarQube` scanning, don't forget to add the `SONAR_TOKEN` variable to your project. You can learn how to generate a token [here](https://docs.sonarqube.org/latest/user-guide/user-token/).

Also, in order to comply with the bank's IT standards, the `SONAR_PROJECT_KEY` variable must be present in the Gitlab CI/CD variables in the project repository. 
The value of the variable `SONAR_PROJECT_KEY` must match the template `[SonarQube Team Space].[Project Name]`, so its presence and compliance with the template is checked before running the check based on the variable `SONAR_PROJECT_NAME`, into which the value `[SonarQube Team Space]` must be inserted. 

It is possible to set up [integration with Hitler](https://docs.sonarsource.com/sonarqube/9.9/devops-platform-integration/gitlab-integration /) to automatically receive comments with the results of scanning `SonarQube` in Merge Requests. To do this, you need to define the variable `ENABLE_GITLAB_INTEGRATION: "true"`, as well as `IS_MONREPO: "true"` if the project is a monorepository. The token type must be User Token (must start with squ_).

To use a scan other than `java`, create a file `sonar-project.properties` in the root of the project.

Example:
```yaml
sonar.projectKey=${env.SONAR_PROJECT_KEY}.${env.CI_PROJECT_NAME}
sonar.projectName=${env.SONAR_PROJECT_KEY}.${env.CI_PROJECT_NAME}
sonar.python.coverage.reportPaths=reports/coverage.xml
sonar.python.xunit.reportPath=reports/junit.xml
sonar.sources=src
sonar.tests=tests
sonar.inclusions=src/**
sonar.exclusions=.venv/**,alembic/**,db/**,docs/**,helm-chart/**,logo/**
sonar.host.url=${env.SONAR_HOST_URL}
sonar.login=${env.SONAR_TOKEN}
sonar.gitlab.commit_sha=${env.CI_COMMIT_SHA}
sonar.gitlab.project_id=${env.CI_PROJECT_ID}
sonar.qualitygate.wait=true

```

### Checking the code for credits

To check the code for credits that have leaked there, there is a job .gitleaks. Jobs are launched only on branches because only the re-creation of the repo will save you from credits in the wizard. If job has found credits and these are real credits, then you should:
* either delete the branch, clean up the history locally and re-upload the branch
*  either leave it as it is (and hope that the security guards will not find it), or make an example of squash commits.
If the credits are unrealistic or outdated and they should remain in the repo, then use exceptions via the file
[.gitleaksignore](https://github.com/gitleaks/gitleaks?tab=readme-ov-file#gitleaksignore).

However, unreachable commits can still be found by direct links. So the option of deleting a branch from the repository just doesn't work like that.

In order for such comets to disappear, you need * no earlier than half an hour after deleting the branch* to go to the repo settings, Repository, Repository cleanup. There's just an empty file to slip in. You will receive an email about the completion of the cleaning. After that, all deleted comments will become 404.

## External Variables Used in Templates
```yaml
.sonarqube-java:
BUILD_TYPE
SONAR_TOKEN
SONAR_PROJECT_KEY
SONAR_PROJECT_NAME
SONAR_CUSTOM_OPTIONS
ENABLE_GITLAB_INTEGRATION # optional, default "false"
IS_MONOREPO # optional, default "false"

.sonarqube-kotlin:
BUILD_TYPE
SONAR_TOKEN
SONAR_PROJECT_KEY
SONAR_PROJECT_NAME
SONAR_CUSTOM_OPTIONS
ENABLE_GITLAB_INTEGRATION # optional, default "false"
IS_MONOREPO # optional, default "false"

.sonarqube-java-maven:
SONAR_TOKEN
SONAR_PROJECT_KEY
SONAR_PROJECT_NAME
SONAR_CUSTOM_OPTIONS
ENABLE_GITLAB_INTEGRATION # optional, default "false"
IS_MONOREPO # optional, default "false"

.sonarqube-kotlin-maven:
SONAR_TOKEN
SONAR_PROJECT_KEY
SONAR_PROJECT_NAME
SONAR_CUSTOM_OPTIONS
ENABLE_GITLAB_INTEGRATION # optional, default "false"
IS_MONOREPO # optional, default "false"

.sonarqube-java-gradle:
SONAR_TOKEN
SONAR_PROJECT_KEY
SONAR_PROJECT_NAME
SONAR_CUSTOM_OPTIONS
ENABLE_GITLAB_INTEGRATION # optional, default "false"
IS_MONOREPO # optional, default "false"

.sonarqube-kotlin-gradle:
SONAR_TOKEN
SONAR_PROJECT_KEY
SONAR_PROJECT_NAME
SONAR_CUSTOM_OPTIONS
ENABLE_GITLAB_INTEGRATION # optional, default "false"
IS_MONOREPO # optional, default "false"

```

## Variables with Default Values That Can Be Overridden
```yaml
.sonarqube-java:
SONAR_HOST_URL: "https://path.to.your.sonarqube.com/"
SONAR_JAVA_OPTIONS: "-Dsonar.language=java -Dsonar.java.coveragePlugin=jacoco -Dsonar.sources=src/main/java -Dsonar.tests=src/test/java"
ENABLE_GITLAB_INTEGRATION: "false"
IS_MONOREPO: "false"
GIT_DEPTH: "0"

.sonarqube-java-maven:
SONAR_HOST_URL: "https://path.to.your.sonarqube.com/"
SONAR_JAVA_OPTIONS: "-Dsonar.language=java -Dsonar.java.coveragePlugin=jacoco -Dsonar.sources=src/main/java -Dsonar.tests=src/test/java"
SONAR_MAVEN_OPTIONS: "-Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml -Dsonar.java.binaries=target/classes -Dsonar.javalibraries=target/dependency/*.jar"
ENABLE_GITLAB_INTEGRATION: "false"
IS_MONOREPO: "false"
GIT_DEPTH: "0"

.sonarqube-java-gradle:
SONAR_HOST_URL: "https://path.to.your.sonarqube.com/"
SONAR_JAVA_OPTIONS: "-Dsonar.language=java -Dsonar.java.coveragePlugin=jacoco -Dsonar.sources=src/main/java -Dsonar.tests=src/test/java"
SONAR_GRADLE_OPTIONS: "-Dsonar.coverage.jacoco.xmlReportPaths=build/reports/jacoco/test/jacocoTestReport.xml -Dsonar.java.binaries=build/classes"
ENABLE_GITLAB_INTEGRATION: "false"
IS_MONOREPO: "false"
GIT_DEPTH: "0"

.sonarqube-kotlin-gradle:
SONAR_HOST_URL: "https://path.to.your.sonarqube.com/"
SONAR_JAVA_OPTIONS: "-Dsonar.language=kotlin\n
  -Dsonar.java.coveragePlugin=jacoco\n
  -Dsonar.sources=src/main/kotlin\n
  -Dsonar.tests=src/test/kotlin"
SONAR_GRADLE_OPTIONS: "-Dsonar.coverage.jacoco.xmlReportPaths=build/reports/jacoco/test/jacocoTestReport.xml\n
  -Dsonar.java.binaries=build/classes/java/main,build/classes/kotlin/main"
ENABLE_GITLAB_INTEGRATION: "false"
IS_MONOREPO: "false"
GIT_DEPTH: "0"

.docker-lint:
DOCKERFILE: $CI_PROJECT_DIR/Dockerfile
FORMAT: tty
THRESHOLD: info
PATH_IN_REPORT: $DOCKERFILE
HADOLINT_OPTIONS: ""

.gitleaks:
GITLEAKS_LOG_OPTS: '--'

```

## GitLab Environment Variables Used in Templates
```yaml
.sonarqube-*:
$CI_PROJECT_ID
$CI_MERGE_REQUEST_ID
$CI_MERGE_REQUEST_IID
$CI_MERGE_REQUEST_TARGET_BRANCH_NAME
$CI_MERGE_REQUEST_SOURCE_BRANCH_NAME

.validate-by-yamllint:
$CONFIGS_TO_VALIDATE
$YAMLLINT_ARGS

.validate-by-schema:
$CONFIGS_TO_VALIDATE
$VALIDATION_SCHEMA_FILE

.docker-lint:
$CI_PROJECT_DIR

.gitleaks:
$CI_PROJECT_DIR
```

## Usage Examples in .gitlab-ci.yml

```yaml
########################################
###      Include template job        ###
###     .release from this repo      ###
########################################
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'Major.minor.patch' # Please fill in with the desired tag
    file:
      - 'code-scan/code-scan.yml'

########################################
###   Create job in your pipeline    ###
###     extending included job       ###
########################################

# Tests will be required to analyze the sonarqube project. The run-test step schematically shows this, but is not a full-fledged copy step.
run-tests:
  stage: test
  script:
    - ./gradlew $WRAPPER_PARAMS $GRADLE_PARAMS build
  artifacts:
    paths:
      - build/classes
      - build/reports
      - target/site/jacoco/jacoco.xml

sonarqube:
  stage: code-scan
  extends:
    - .sonarqube-java
  variables:
    SONAR_PROJECT_NAME: "[SonarQube Team Space]"
    SONAR_PROJECT_KEY: "[SonarQube Team Space].[Project Name]" # should be set in Gitlab CI/CD variables in order to follow IT Standards
    SONAR_CUSTOM_OPTIONS: "-Dsonar.exclusions=**/test/** -Dsonar.coverage.exclusions=**/test/**"
  rules:
    - if: '$SONAR_CHECK == "false"'
      when: never
    - when: always

# Checking branches (not the default branch) at every commit
gitleaks:
  stage: code-scan
  extends: .gitleaks
#  allow_failure: true # If you don't want the job drop the pipeline
# Verification is only in MR pipeline.
# Can be used together with checking branches (if you don't drop the pipeline there, for example).
# Checking only in MR is not recommended because the security scanner can find credits before you do MR.
gitleaks-mr:
  stage: code-scan
  extends: .gitleaks
  variables:
    GITLEAKS_LOG_OPTS: '--all ${CI_MERGE_REQUEST_DIFF_BASE_SHA}..${CI_COMMIT_SHA}'
  rules:
  - if: '$CI_MERGE_REQUEST_IID'

########################################
###    Or construct your own job     ###
###       using reference tags       ###
########################################
sonarqube:
  stage: code-scan
  variables:
    SONAR_PROJECT_NAME: "[SonarQube Team Space]"
    SONAR_HOST_URL: "https://path.to.your.sonarqube.com/"
    SONAR_PROJECT_KEY: "[SonarQube Team Space].SONAR_PROJECT_NAME" # should be set in Gitlab CI/CD variables in order to follow IT Standards
    SONAR_JAVA_OPTIONS: "-Dsonar.language=java -Dsonar.java.coveragePlugin=jacoco -Dsonar.sources=src/main/java -Dsonar.tests=src/test/java"
    ENABLE_GITLAB_INTEGRATION: "true"
    GIT_DEPTH: "0"
  script: !reference [.sonarqube-java, script]
  image: !reference [.sonarqube-java, image]
  rules:
    - if: '$SONAR_CHECK == "false"'
      when: never
    - when: always

lint-helm-chart:
  stage: lint
  extends:
    - .helm-lint
  variables:
    PATH_TO_CHART: "./chart"
  allow_failure: true

lint-dockerfile:
  stage: lint
  extends:
    - .docker-lint
  variables:
    FORMAT: sonarqube
    THRESHOLD: none
    HADOLINT_OPTIONS: "--require-label=maintainer:text --verbose"

code-scan:
  stage: code-scan
  extends: .sonarqube-scan
  variables:
    SONAR_PROJECT_KEY: "" # should be set in Gitlab CI/CD variables
    SONAR_TOKEN: ""       # hide the value in Gitlab CI/CD variables


### For debug scan
code-scan:
  stage: code-scan
  extends: .sonarqube-scan
  script: sonar-scanner -X
  variables:
    SONAR_PROJECT_KEY: "" # should be set in Gitlab CI/CD variables
    SONAR_TOKEN: ""       # hide the value in Gitlab CI/CD variables
```


```yaml
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'Major.minor.patch' # Please fill in with the desired tag
    file:
      - 'code-scan/code-scan.yml'

validate-yaml:
  extends:
    - .validate-by-yamllint
  variables:
    CONFIGS_TO_VALIDATE: "configs/*.yaml"

validate-configs:
  extends:
    - .validate-by-schema
  variables:
    VALIDATION_SCHEMA_FILE: "configs/schema.yaml"
    CONFIGS_TO_VALIDATE: "configs/*.yaml"
```
