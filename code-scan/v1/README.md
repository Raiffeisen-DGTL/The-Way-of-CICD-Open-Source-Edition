# Pipeline Modules for Code Checking

## Description

This section contains a pipeline module for code checking with `SonarQube`, as well as various ways to perform linting on configurations, manifests, and more.

For `SonarQube`, there is an option to check code in `Java`, `Kotlin`, and with build systems `Maven` and `Gradle`. You can use the `.sonarqube-java`/`.sonarqube-kotlin` templates, which support both build types and select the appropriate one using the `BUILD_TYPE` variable with values `maven` or `gradle`. Alternatively, you can use separate templates for `Maven` builds: `.sonarqube-java-maven`/`.sonarqube-kotlin-maven`, and `Gradle` builds: `.sonarqube-java-gradle`/`.sonarqube-kotlin-gradle`.

To use `SonarQube` scanning, don't forget to add the `SONAR_TOKEN` variable to your project. You can learn how to generate a token [here](https://docs.sonarqube.org/latest/user-guide/user-token/).

Also, to comply with the bank's IT standards, you need to have the `SONAR_PROJECT_KEY` variable in the Gitlab CI/CD variables in your project's repository. The value of the `SONAR_PROJECT_KEY` variable should follow the pattern `[SonarQube Team Space].[Project Name]`. Therefore, its presence and conformity to the pattern are checked before running the code checking based on the `SONAR_PROJECT_NAME` variable, in which you should insert the value `[SonarQube Team Space]`.

## GitLab Environment Variables Used in Templates
```yaml
.sonarqube-*:
$CI_PROJECT_NAME
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
      - '/code-scan/v1/code-scan.yml'

########################################
###   Create job in your pipeline    ###
###     extending included job       ###
########################################
sonarqube:
  stage: code-scan
  extends:
    - .sonarqube-maven
  variables:
    SONAR_PROJECT_KEY: "[SonarQube Team Space].$CI_PROJECT_NAME"
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
    SONAR_HOST_URL: "https://path.to.your.sonarqube.com/"
    SONAR_PROJECT_KEY: "[SonarQube Team Space].$CI_PROJECT_NAME"
    SONAR_LANGUAGE_OPTIONS: "-Dsonar.language=java -Dsonar.java.coveragePlugin=jacoco -Dsonar.sources=src/main/java -Dsonar.tests=src/test/java"
    SONAR_BUILD_OPTIONS: "-Dsonar.coverage.jacoco.xmlReportPaths=target/site/jacoco/jacoco.xml -Dsonar.java.binaries=target/classes -Dsonar.java.libraries=target/dependency/*.jar"
    GIT_DEPTH: "0"
  script: !reference [.sonarqube-maven, script]
  rules:
    - if: '$SONAR_CHECK == "false"'
      when: never
    - when: always

```
