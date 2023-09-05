# Pipeline Modules for Deployment Stages

## Description

This section contains a pipeline module for deployment stages of applications.

## Usage Examples in .gitlab-ci.yml

### Building OCI Images

```yaml
########################################
###    Deploy via kubectl apply -f   ###
########################################
deploy-via-kubectl:
  image: [.deploy, image]
  stage: deploy
  variables:
    K8S_CONFIG: "YOUR_CONFIG" # add kubeconfig to your Settings -> CI/CD -> Variables
    YAML_TO_DEPLOY: >-
      ./manifests/*.yml
    DEPLOY_ARGS: >-
      --dry-run=client
  script:
    - !reference [.deploy, fetch_kubeconfig]
    - !reference [.deploy, kubectl]

########################################
###        Deploy with kubectl       ###
###            via extends           ###
########################################
deploy-with-kubectl:
  stage: deploy
  extends:
    - .deploy-with-kubectl
  variables:
    K8S_CONFIG: "YOUR_CONFIG" # add kubeconfig to your Settings -> CI/CD -> Variables
    YAML_TO_DEPLOY: "$CI_PROJECT_DIR/manifests/"
    DEPLOY_ARGS: >-
      --dry-run=client

########################################
###        Deploy with helm          ###
###            via extends           ###
########################################
deploy-with-helm:
  stage: deploy
  before_script:
    - export KUBECONFIG="$K8S_CONFIG" # add kubeconfig to your Settings -> CI/CD -> Variables
  extends:
    - .deploy-with-helm
  variables:
    NAMESPACE: "$CI_PROJECT_NAME"
    RELEASE_NAME: "$CI_PROJECT_NAME"
    CHART_TO_DEPLOY: "helm-chart-go-web-server"
    HELM_ARGS: >-
      --wait

########################################
###          Deploy with helm        ###
###        using reference tag       ###
########################################
deploy-via-helm:
  stage: deploy
  image: [.deploy, image]
  variables:
    K8S_CONFIG: "YOUR_CONFIG" # add kubeconfig to your Settings -> CI/CD -> Variables
    RELEASE_NAME: "$CI_PROJECT_NAME"
    CHART_TO_DEPLOY: "./charts/my-chart"
    NAMESPACE: "$CI_PROJECT_NAME"
    HELM_ARGS: >-
      --wait
      --atomic
  script:
    - !reference [.deploy, fetch_kubeconfig]
    - !reference [.deploy, helm]
```
