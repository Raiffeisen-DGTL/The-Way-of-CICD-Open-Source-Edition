# Pipeline constructor for the build and publishing stages
- [Pipeline constructor for the build and publishing stages](#pipeline-constructor-for-the-build-and-publishing-stages)
  - [Description](#description)
  - [Instructions for different programming languages](#instructions-for-different-programming-languages)
  - [External Variables Used in Templates](#external-variables-used-in-templates)
  - [Variables with Default Values That Can Be Overridden](#variables-with-default-values-that-can-be-overridden)
  - [GitLab environment variables used in templates](#gitlab-environment-variables-used-in-templates)
  - [Kaniko usage examples in .gitlab-ci.yml](#kaniko-usage-examples-in-gitlab-ciyml)

## Description

This section contains a pipeline template for building applications into OCI images and publishing various packages. To publish an image and access the registry during the Docker image building process, you can use the `REGISTRY_USER` and `REGISTRY_PASSWORD` variables, which are recommended to be configured in the GitLab CI/CD project or group-level variables. (NEVER EXPOSE YOUR CREDENTIALS INTO GIT) Additionally, the variables `IMAGE_NAME` and `IMAGE_TAG` can be received as artifacts from the [versioning step](../../version/v1/README.md).

## Instructions for different programming languages

- [Golang](golang.md)
- [NodeJS](nodejs.md)
- [Java](java.md)
- [Python](python.md)

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

.package_helm_chart:
REGISTRY_HELM_REPO_VIRT  # Specify path to your virtual repository with helm charts, it's used for download your charts
REGISTRY_HELM_REPO_LOCAL # Specify path to your local repository with helm charts, it's used for upload your charts
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
```

## GitLab environment variables used in templates
```yaml
.build_kaniko:
$CI_PROJECT_DIR
$CI_PROJECT_NAME

.build_docker:

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
    ref: 'Major.minor.patch' # Please fill in with the desired tag
    file:
      - 'build/build-package.yml'

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
    REGISTRY_USER: "YOUR_USER_LOGIN" # Hide this in Settings -> CI/CD -> Variables
    REGISTRY_PASSWORD: "YOUR_USER_PASSWORD" # Hide this in Settings -> CI/CD -> Variables
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
    REGISTRY_USER: "YOUR_USER_LOGIN" # Hide this in Settings -> CI/CD -> Variables
    REGISTRY_PASSWORD: "YOUR_USER_PASSWORD" # Hide this in Settings -> CI/CD -> Variables
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
    REGISTRY_HELM_REPO_VIRT: "https://path.to.your.helm.repo.com" # It's an example, specify your repo :)
    REGISTRY_HELM_REPO_LOCAL: "https://path.to.your.helm.repo.com"        # It's an example, specify your repo :)
    PATH_TO_CHART: "." # Specify path to your chart, "." find the chart in the current directory
```
