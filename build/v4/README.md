# Pipeline Modules for build and upload stages

## Description

This section contains a pipeline template for building applications into OCI images and publishing various packages. To publish an image and access the registry during the Docker image building process, you can use the `REGISTRY_USER` and `REGISTRY_PASSWORD` variables, which are recommended to be configured in the GitLab CI/CD project or group-level variables. (NEVER EXPOSE YOUR CREDENTIALS INTO GIT) Additionally, the variables `IMAGE_NAME` and `IMAGE_TAG` can be received as artifacts from the [versioning step](../../version/v3/README.md).

## External Variables Used in Templates
```yaml
.build_kaniko:
REGISTRY_HOST #default points to hub.docker.com
REGISTRY_PATH #relative path on your registry
IMAGE_NAME
IMAGE_TAG
DESTINATION          # The image will be pushed to specified path
DOCKER_BUILD_ARGS    # optional, for example "--build-arg AAA=\"$BBB\" --build-arg XXX=\"$YYY\""
REGISTRY_USER     # Hide this in Settings -> CI/CD -> Variables
REGISTRY_PASSWORD # Hide this in Settings -> CI/CD -> Variables

.build_docker:
REGISTRY_HOST #default points to hub.docker.com
REGISTRY_PATH #relative path on your registry
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
TESTCONTAINERS_HUB_IMAGE_NAME_PREFIX # you need to specify it if you use testcontainers

.build_image_jib:
JVM_IMAGE           # Specify your image with gradle, default value is gradle:jdk11-alpine
GRADLE
GRADLE_PARAMS
REGISTRY_PATH       # used in jib.gradle to build image path
IMAGE_NAME          # used in jib.gradle to build image path
IMAGE_TAG           # used in jib.gradle to get image tag

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
PIPELINE_CACHE_KEY: $CI_COMMIT_REF_NAME-$CI_PIPELINE_ID
REGISTRY_HOST: #default points to hub.docker.com
DOCKER_HOST: tcp://dind:2375
DOCKER_TLS_CERTDIR: ""
DOCKER_DRIVER: overlay2
TESTCONTAINERS_RYUK_DISABLED: "true"
TESTCONTAINERS_CHECKS_DISABLE: "true"
GRADLE_OPTS: "-Dorg.gradle.daemon=false"
GRADLE_USER_HOME: ".gradle/"
JUNIT_PATH: "./build/test-results/**/*.xml"

.build_image_jib:
JVM_IMAGE: "gradle:jdk11-alpine"
REGISTRY_HOST: #default points to hub.docker.com # used in jib.gradle to build image path
PIPELINE_CACHE_KEY: $CI_COMMIT_REF_NAME-$CI_PIPELINE_ID
GRADLE_OPTS: "-Dorg.gradle.daemon=false"
GRADLE_USER_HOME: ".gradle/"
```

## GitLab Environment Variables Used in Templates
```yaml
.build_kaniko:
$CI_PROJECT_DIR
$CI_PROJECT_NAME

.build_docker:

.build_and_test_gradle:
$CI_COMMIT_REF_NAME
$CI_PIPELINE_ID

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
      - '/build/v4/build-package.yml'

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

You can add a jib.gradle file to your project directory with the following configuration for building an image:
```
ext.imagePath = {
    def registry_host = System.getenv('REGISTRY_HOST')
    def registry_path = System.getenv('REGISTRY_PATH')
    def image_name = System.getenv('IMAGE_NAME')
    return [registry_host,registry_path,image_name].join('/')
}

ext.imageTag = {
    return System.getenv('IMAGE_TAG')
}

jib {
    from {
        image = 'eclipse-temurin:17.0.6_10-jre-alpine'
/* uncomment if you need source repo auth
        auth {
            username "$registryUser"
            password "$registryPassword"
        }
    }
    to {
        image = "${imagePath()}:${imageTag()}"
        auth {
            username "$registryUser"
            password "$registryPassword"
*/        }
    }

    container {
        user = 'www-data'
        jvmFlags = ['-Xms512m','-Xmx1024m']
        ports = ['8080']
        environment = [TZ: "Europe/Moscow",LANG: "en_US.UTF-8"]
        creationTime = "USE_CURRENT_TIMESTAMP"
    }
    new File(projectDir, "deploy.env").text = "image_path=${imagePath()}\nimage_tag=${imageTag()}\n"
}
```

To add the Jib plugin to your build.gradle file and apply the settings from the jib.gradle file, follow these steps:
```
...

plugins {
    ...
    id 'com.google.cloud.tools.jib' version '3.3.1'
}

...

apply from: 'jib.gradle'

```

More info about `Jib`: https://github.com/GoogleContainerTools/jib/tree/master

After setting up the Jib plugin in your build.gradle file and adding the jib.gradle file, you can add jobs to build and publish images using Jib in your .gitlab-ci.yml file. Here's an example of how to do it:

```yaml
########################################
###      Include template job        ###
###   .build_and_test_gradle         ###
########################################

include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'master'
    file:
      - '/build/v4/build-package.yml'

########################################
###   Create job in your pipeline    ###
###     extending included job       ###
########################################
build-via-gradle:
  stage: build
  extends:
    - .build_and_test_gradle
  variables:
    JVM_IMAGE: "eclipse-temurin:17.0.6_10-jdk-alpine"
    GRADLE: "./gradlew --stacktrace"
    GRADLE_PARAMS: "-Pregistry_user=$REGISTRY_USER -Pregistry_password=$REGISTRY_PASSWORD"
    TESTCONTAINERS_HUB_IMAGE_NAME_PREFIX: "" #define if you use test containers
  before_script:
    - chmod +x gradlew

build-image:
  stage: publish-image
  extends:
    - .build_image_jib
  variables:
    REGISTRY_HOST: #default points to hub.docker.com
    IMAGE_NAME: "$CI_PROJECT_NAME"
    IMAGE_TAG: "${VERSION}"
    GRADLE: "./gradlew --stacktrace"
    GRADLE_PARAMS: "-Pregistry_user=$REGISTRY_USER -Pregistry_password=$REGISTRY_PASSWORD"
  before_script:
    - chmod +x gradlew
```
