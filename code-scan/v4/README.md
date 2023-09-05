# Pipeline Modules for Code Checking

## Description

This section contains a pipeline module for code checking with `SonarQube`, as well as various ways to perform linting on configurations, manifests, and more.

For `SonarQube`, there is an option to check code in `Java`, `Kotlin`, and with build systems `Maven` and `Gradle`. You can use the `.sonarqube-java`/`.sonarqube-kotlin` templates, which support both build types and select the appropriate one using the `BUILD_TYPE` variable with values `maven` or `gradle`. Alternatively, you can use separate templates for `Maven` builds: `.sonarqube-java-maven`/`.sonarqube-kotlin-maven`, and `Gradle` builds: `.sonarqube-java-gradle`/`.sonarqube-kotlin-gradle`.

To use `SonarQube` scanning, don't forget to add the `SONAR_TOKEN` variable to your project. You can learn how to generate a token [here](https://docs.sonarqube.org/latest/user-guide/user-token/).

Also, to comply with the bank's IT standards, you need to have the `SONAR_PROJECT_KEY` variable in the Gitlab CI/CD variables in your project's repository. The value of the `SONAR_PROJECT_KEY` variable should follow the pattern `[SonarQube Team Space].[Project Name]`. Therefore, its presence and conformity to the pattern are checked before running the code checking based on the `SONAR_PROJECT_NAME` variable, in which you should insert the value `[SonarQube Team Space]`.

## External Variables Used in Templates
```yaml
.sonarqube-java:
BUILD_TYPE
SONAR_TOKEN
SONAR_PROJECT_KEY
SONAR_PROJECT_NAME

.sonarqube-kotlin:
BUILD_TYPE
SONAR_TOKEN
SONAR_PROJECT_KEY
SONAR_PROJECT_NAME

.sonarqube-java-maven:
SONAR_TOKEN
SONAR_PROJECT_KEY
SONAR_PROJECT_NAME

.sonarqube-kotlin-maven:
SONAR_TOKEN
SONAR_PROJECT_KEY
SONAR_PROJECT_NAME

.sonarqube-java-gradle:
SONAR_TOKEN
SONAR_PROJECT_KEY
SONAR_PROJECT_NAME

.sonarqube-kotlin-gradle:
SONAR_TOKEN
SONAR_PROJECT_KEY
SONAR_PROJECT_NAME

```

## Variables with Default Values That Can Be Overridden

```yaml
.sonarqube-java:
SONAR_HOST_URL: "https://path.to.your.sonarqube.com/"
SONAR_JAVA_OPTIONS: "-Dsonar.language=java -Dsonar.java.coveragePlugin=jacoco -Dsonar.sources=src/main/java -Dsonar.tests=src/test/java"
GIT_DEPTH: "0"

.sonarqube-java-maven:
SONAR_HOST_URL: "https://path.to.your.sonarqube.com/"
SONAR_JAVA_OPTIONS: "-Dsonar.language=java -Dsonar.java.coveragePlugin=jacoco -Dsonar.sources=src/main/java -Dsonar.tests=src/test/java"
SONAR_MAVEN_OPTIONS: "-Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml -Dsonar.java.binaries=target/classes -Dsonar.javalibraries=target/dependency/*.jar"
GIT_DEPTH: "0"

.sonarqube-java-gradle:
SONAR_HOST_URL: "https://path.to.your.sonarqube.com/"
SONAR_JAVA_OPTIONS: "-Dsonar.language=java -Dsonar.java.coveragePlugin=jacoco -Dsonar.sources=src/main/java -Dsonar.tests=src/test/java"
SONAR_GRADLE_OPTIONS: "-Dsonar.coverage.jacoco.xmlReportPaths=build/reports/jacoco/test/jacocoTestReport.xml -Dsonar.java.binaries=build/classes"
GIT_DEPTH: "0"

.sonarqube-kotlin-gradle:
SONAR_HOST_URL: "https://path.to.your.sonarqube.com/"
SONAR_JAVA_OPTIONS: "-Dsonar.language=kotlin\n
  -Dsonar.java.coveragePlugin=jacoco\n
  -Dsonar.sources=src/main/kotlin\n
  -Dsonar.tests=src/test/kotlin"
SONAR_GRADLE_OPTIONS: "-Dsonar.coverage.jacoco.xmlReportPaths=build/reports/jacoco/test/jacocoTestReport.xml\n
  -Dsonar.java.binaries=build/classes/java/main,build/classes/kotlin/main"
GIT_DEPTH: "0"

```

## GitLab Environment Variables Used in Templates
```yaml
.sonarqube-*:
$CI_MERGE_REQUEST_ID
$CI_MERGE_REQUEST_IID
$CI_MERGE_REQUEST_TARGET_BRANCH_NAME
$CI_MERGE_REQUEST_SOURCE_BRANCH_NAME

```

# Usage Examples in .gitlab-ci.yml

```yaml
########################################
###      Include template job        ###
###     .release from this repo      ###
########################################
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'master'
    file:
      - '/code-scan/v4/code-scan.yml'

########################################
###   Create job in your pipeline    ###
###     extending included job       ###
########################################
sonarqube:
  stage: code-scan
  extends:
    - .sonarqube-java
  variables:
    SONAR_PROJECT_NAME: "[SonarQube Team Space]"
    SONAR_PROJECT_KEY: "[SonarQube Team Space].[Project Name]" # should be set in Gitlab CI/CD variables in order to follow IT Standards
  rules:
    - if: '$SONAR_CHECK == "false"'
      when: never
    - when: always

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
