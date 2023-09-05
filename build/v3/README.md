# Pipeline Modules for build and upload stages

## Description

This section contains a pipeline template for building applications into OCI images and publishing various packages. To publish an image and access the registry during the Docker image building process, you can use the `REGISTRY_USER` and `REGISTRY_PASSWORD` variables, which are recommended to be configured in the GitLab CI/CD project or group-level variables. (NEVER EXPOSE YOUR CREDENTIALS INTO GIT) Additionally, the variables `IMAGE_NAME` and `IMAGE_TAG` can be received as artifacts from the [versioning step](../../version/v3/README.md).

## External Variables Used in Templates
```yaml
.build_kaniko:
REGISTRY_PATH
IMAGE_NAME
IMAGE_TAG
DESTINATION          # The image will be pushed to specified path
DOCKER_BUILD_ARGS    # optional, for example "--build-arg AAA=\"$BBB\" --build-arg XXX=\"$YYY\""
REGISTRY_USER     # Hide this in Settings -> CI/CD -> Variables
REGISTRY_PASSWORD # Hide this in Settings -> CI/CD -> Variables

.build_docker:
REGISTRY_PATH
IMAGE_NAME
IMAGE_TAG
DESTINATION          # The image will be pushed to specified path
DOCKER_BUILD_ARGS    # optional, for example "--build-arg AAA=\"$BBB\" --build-arg XXX=\"$YYY\""
REGISTRY_USER     # Hide this in Settings -> CI/CD -> Variables
REGISTRY_PASSWORD # Hide this in Settings -> CI/CD -> Variables

.build_and_test_gradle:
JVM_IMAGE           # Specify your image with gradle, default value is gradle:jdk11-alpine
GRADLE
GRADLE_PARAMS
GRADLE_USER_HOME
JUNIT_PATH

.build_image_jib:
JVM_IMAGE           # Specify your image with gradle, default value is gradle:jdk11-alpine
GRADLE
GRADLE_PARAMS
GRADLE_USER_HOME

.package_helm_chart:
HELM_REPO  # Specify path to your repository with helm charts
```

## Variables with Default Values That Can Be Overridden
```yaml
.build_kaniko:
REGISTRY_HOST: #default points to hub.docker.com
DOCKERFILE: "Dockerfile"

.build_docker:
DOCKER_HOST: tcp://dind:2375
REGISTRY_HOST: #default points to hub.docker.com
DOCKERFILE: "Dockerfile"

.build_and_test_gradle:
JVM_IMAGE: "gradle:jdk11-alpine"
DOCKER_HOST: tcp://dind:2375
DOCKER_TLS_CERTDIR: ""
DOCKER_DRIVER: overlay2
TESTCONTAINERS_RYUK_DISABLED: "true"
TESTCONTAINERS_CHECKS_DISABLE: "true"
PIPELINE_CACHE_KEY: $CI_COMMIT_REF_NAME-$CI_PIPELINE_ID

.build_image_jib:
JVM_IMAGE: "gradle:jdk11-alpine"
DOCKER_HOST: tcp://localhost:2375
PIPELINE_CACHE_KEY: $CI_COMMIT_REF_NAME-$CI_PIPELINE_ID
```

## GitLab Environment Variables Used in Templates
```yaml
.build_kaniko:
$CI_PROJECT_DIR
$CI_PROJECT_NAME

.build_docker:

.build_and_test_gradle:

.build_image_jib:
$CI_COMMIT_REF_NAME
$CI_PIPELINE_ID

.package_helm_chart:
$CI_PROJECT_NAME
```

## Kaniko usage examples in .gitlab-ci.yml

```yaml
########################################
###      Include template job        ###
###  .build_kaniko from this repo    ###
########################################
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'master'
    file:
      - '/build/v3/build-package.yml'

########################################
###   Create job in your pipeline    ###
###     extending included job       ###
########################################
build-via-kaniko:
  stage: build
  extends:
    - .build_kaniko
  variables:
    REGISTRY_PATH: "" #relative path on your registry
    IMAGE_NAME: "$CI_PROJECT_NAME"
    IMAGE_TAG: "YOUR TAG"
    DESTINATION: "$REGISTRY_HOST/$REGISTRY_PATH/${IMAGE_NAME}:$IMAGE_TAG"
    DOCKERFILE: "Dockerfile.debian"

########################################
###    Or construct your own job     ###
###       using reference tags       ###
########################################
build-via-kaniko:
  stage: build
  variables:
    REGISTRY_HOST: #default points to hub.docker.com
    DOCKERFILE: "Dockerfile"
    REGISTRY_PATH: "" #relative path on your registry
    IMAGE_NAME: "$CI_PROJECT_NAME"
    IMAGE_TAG: "YOUR TAG"
    DESTINATION: "$REGISTRY_HOST/$REGISTRY_PATH/${IMAGE_NAME}:$IMAGE_TAG"
  image:
    name: !reference [.build_kaniko, image, name]
    entrypoint: !reference [.build_kaniko, image, entrypoint]
  script:
    - !reference [.build_kaniko, script]

########################################
###   Create job in your pipeline    ###
###     extending included job!      ###
###     Let's package helm chart     ###
########################################
package-helm-chart:
  stage: build
  extends:
    - .package_helm_chart
  variables:
    HELM_REPO: "https://path.to.your.helm.repo.com"
    PATH_TO_CHART: "." # Specify path to your chart, "." find the chart in the current directory
```

## Gradle usage examples in .gitlab-ci.yml

More info about `Jib`: https://github.com/GoogleContainerTools/jib/tree/master

```yaml
########################################
###      Include template job        ###
###   .build_and_test_gradle         ###
########################################

include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'master'
    file:
      - '/build/v3/build-package.yml'

########################################
###   Create job in your pipeline    ###
###     extending included job       ###
########################################
build-via-gradle:
  stage: build
  extends:
    - .build_and_test_gradle
  variables:
    JVM_IMAGE: "gradle:jdk11-alpine"
    SERVICE_NAME: "$CI_PROJECT_NAME"
    GRADLE: "./gradlew --stacktrace"
    GRADLE_OPTS: "-Dorg.gradle.daemon=false"
    GRADLE_PARAMS: "-Pregistry_user=${cicd_user} -Pregistry_password=${cicd_password}"
    GRADLE_USER_HOME: ".gradle/"
    JAVA_HOME: "/usr/lib/jvm/java-11-openjdk"
    JUNIT_PATH: "./build/test-results/**/*.xml"

build-image:
  stage: publish-image
  extends:
    - .build_image_jib
  variables:
    JVM_IMAGE: "gradle:jdk11-alpine"
    SERVICE_NAME: "$CI_PROJECT_NAME"
    GRADLE: "./gradlew --stacktrace"
    GRADLE_OPTS: "-Dorg.gradle.daemon=false"
    GRADLE_PARAMS: "-Pregistry_user=${cicd_user} -Pregistry_password=${cicd_password}"
    GRADLE_USER_HOME: ".gradle/"
    JAVA_HOME: "/usr/lib/jvm/java-11-openjdk"
```
