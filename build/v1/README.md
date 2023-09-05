# Pipeline Modules for build and upload stages

## Description

This section contains a pipeline template for building applications into OCI images and publishing various packages. 

## GitLab Environment Variables Used in Templates
```yaml
$CI_PROJECT_DIR
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
      - '/build/v1/build-package.yml'

########################################
###   Create job in your pipeline    ###
###     extending included job       ###
########################################
build-via-kaniko:
  stage: build
  extends:
    - .build_kaniko
  variables:
    REGISTRY_HOST: #default points to hub.docker.com
    REGISTRY_PATH: "" #relative path on your registry
    DOCKERFILE: "Dockerfile" # Path from root of $CI_PROJECT_DIR, for example 'docker/Dockerfile.maven'
    IMAGE_NAME: "$CI_PROJECT_NAME"
    IMAGE_TAG: "YOUR TAG"
    KANIKO_BUILD_ARGS: >-
      --build-arg REGISTRY_USER=${REGISTRY_USER}
      --build-arg REGISTRY_PASSWORD=${REGISTRY_PASSWORD}

########################################
###    Or construct your own job     ###
###       using reference tags       ###
########################################
build-via-kaniko:
  stage: build
  variables:
    REGISTRY_HOST: #default points to hub.docker.com
    REGISTRY_PATH: "" #relative path on your registry
    DOCKERFILE: "Dockerfile" # Path from root of $CI_PROJECT_DIR, for example 'docker/Dockerfile.maven'
    IMAGE_NAME: "$CI_PROJECT_NAME"
    IMAGE_TAG: "YOUR TAG"
    KANIKO_BUILD_ARGS: >-
      --build-arg REGISTRY_USER=${REGISTRY_USER}
      --build-arg REGISTRY_PASSWORD=${REGISTRY_PASSWORD}
  image:
    name: !reference [.build_kaniko, image, name]
    entrypoint: !reference [.build_kaniko, image, entrypoint]
  before_script:
    - !reference [.build_kaniko, before_script]
  script:
    - !reference [.build_kaniko, script]
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
      - '/build/v1/build-package.yml'

########################################
###   Create job in your pipeline    ###
###     extending included job       ###
########################################
build-via-gradle:
  stage: build
  extends:
    - .build_and_test_gradle
  variables:
    SERVICE_NAME: "$CI_PROJECT_NAME"
    GRADLE_PARAMS: "-Pregistry_user=${cicd_user} -Pregistry_password=${cicd_password}"
    GRADLE_OPTS: "-Dorg.gradle.daemon=false"
    GRADLE: "./gradlew --stacktrace"
    JAVA_HOME: "/usr/lib/jvm/java-11-openjdk"
    JUNIT_PATH: "./build/test-results/**/*.xml"

build-image:
  stage: publish-image
  extends:
    - .build_image_jib
  variables:
    SERVICE_NAME: "$CI_PROJECT_NAME"
    GRADLE_PARAMS: "-Pregistry_user=${cicd_user} -Pregistry_password=${cicd_password}"
    GRADLE_OPTS: "-Dorg.gradle.daemon=false"
    GRADLE: "./gradlew --stacktrace"
    JAVA_HOME: "/usr/lib/jvm/java-11-openjdk"

```
