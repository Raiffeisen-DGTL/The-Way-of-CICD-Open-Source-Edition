# Pipeline constructor for defining registry

## Description

The section contains a description of the stage of determining the target repository for `docker` images and `helm` charts, for simplicity, let's call them `registry`.
ОThe definition of `registry` is based on comparing the current branch from where the pipeline was launched with the default branch of the project, as well as if you launched the pipeline using a tag. In this case, `registry` is determined from the `DOCKER_REGISTRY` variable, otherwise from `DOCKER_REGISTRY_SNAPSHOT`. The same applies to helm - `REGISTRY_HOME_REPO_LOCAL` for release charts, and `REGISTRY_HELM_REPO_LOCAL_SNAPSHOT` for test charts.

## 
## GitLab environment variables used in the pipeline
```yaml
DOCKER_REGISTRY: "Your release docker registry"
DOCKER_REGISTRY_SNAPSHOT: "Your temp registry for snapshots"
HELM_REPO_LOCAL: "Your release helm repository"
HELM_REPO_LOCAL_SNAPSHOT: "Your temp helm repository"
REGISTRY_GENERIC_REPO: "Your release generic repository"
REGISTRY_GENERIC_REPO_SNAPSHOT: "Your temp generic repository"
```

## Connecting in your project

```yaml
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'Major.minor.patch' # Please fill in with the desired tag
    file:
      - 'define-registry/define-registry.yml'

stages:
  - define_registry
```


## Examples of use in .gitlab-ci.yml

Examples are referred to the [build] constructor(../../build/v5/README.md)
```yaml
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'Major.minor.patch' # Please fill in with the desired tag
    file:
      - 'define-registry/define-registry.yml'

stages:
  - define_registry
  - build
  - package-helm-chart


define_docker_registry:
  extends: .define_docker_registry
  variables:
    DOCKER_REGISTRY: "YOUR_DOCKER_REGISTRY"
    DOCKER_REGISTRY_SNAPSHOT: "YOUR_DOCKER_SNAPSHOT_REGISTRY"

define_helm_repo:
  extends: .define_helm_repo
  variables:
    REGISTRY_HELM_REPO_LOCAL: "https://path.to.your.helm.repo/virtual-helm"
    REGISTRY_HELM_REPO_LOCAL_SNAPSHOT: "https://path.to.your.helm.snapshot.helm/virtual-snapshot-helm"

define-generic-registry:
  extends:
    - .define_generic_repo
  variables:
    REGISTRY_GENERIC_REPO: "https://path.to.your.generic"
    REGISTRY_GENERIC_REPO_SNAPSHOT: "https://path.to.your.snapshot.generic"

build-via-kaniko:
  stage: build
  needs:
   - job: define_docker_registry
  extends:
    - .build_kaniko
  variables:
    OCI_REGISTRY: "$REGISTRY"
    REGISTRY_USER: "YOUR_USER_LOGIN" # Hide this in Settings -> CI/CD -> Variables
    REGISTRY_PASSWORD: "YOUR_USER_PASSWORD" # Hide this in Settings -> CI/CD -> Variables
    IMAGE_NAME: "$CI_PROJECT_NAME"
    IMAGE_TAG: "YOUR TAG"
    DESTINATION: "$REGISTRY_HOST/$DOCKER_REGISTRY/${IMAGE_NAME}:$IMAGE_TAG"
    DOCKERFILE: "Dockerfile.debian"

package-helm-chart:
  stage: package-helm
  needs:
   - job: define_helm_repo
  extends:
    - .package_helm_chart
  variables:
    REGISTRY_HELM_REPO_VIRT: "https://path.to.your.helm.repo/virtual-helm" # It's an example, specify your repo :)
    REGISTRY_HELM_REPO_LOCAL: "$REGISTRY_HELM_REPO"
    PATH_TO_CHART: "."
```
