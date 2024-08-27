# Pipeline Modules for Deployment Stages

## Description

This section contains a pipeline module for deployment stages of applications.

## GitLab environment variables used in pipelinе
```yaml
$
$
```

## Usage Examples in .gitlab-ci.yml

```yaml
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'Major.minor.patch' # Please fill in with the desired tag
    file:
      - 'deploy/deploy.yml'


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

########################################
###          Deploy with helm        ###
###         using gitlab-agent       ###
########################################
deploy-via-helm-and-gitlab-agent:
  stage: deploy
  extends: .deploy-with-helm-by-gitlab-agent
  variables:
    RELEASE_NAME: "$CI_PROJECT_NAME"
    CHART_TO_DEPLOY: "./charts/my-chart"
    NAMESPACE: "$CI_PROJECT_NAME"
    GITLAB_AGENT_PROJECT: "YOUR_PROJECT_WITH_GITLAB_AGENT" # Example: group_name/gitlab-agent
    KUBERNETES_CLUSTER_NAME: "NAME_OF_YOUR_KUBERNETES_CLUSTER" # Infrastructure -> Kubernetes clusters
    HELM_ARGS: >-
      --wait
      --atomic

########################################
###          Deploy with werf        ###
###            local chart           ###
########################################
werf-deploy-local-chart:
  extends: .werf-deploy-local-chart
  variables:
    GITLAB_AGENT_PROJECT: "YOUR_PROJECT_WITH_GITLAB_AGENT"     # Example: group_name/gitlab-agent
    KUBERNETES_CLUSTER_NAME: "NAME_OF_YOUR_KUBERNETES_CLUSTER" # Operate -> Kubernetes clusters
    WERF_NAMESPACE: "$CI_PROJECT_NAME"
    CHART_TO_DEPLOY: "helm"
    WERF_PLAN: "true"                                          # dry-run without deploy
    WERF_ARGS: >-
      --set param1.param2=my_new_value

########################################
###          Deploy with werf        ###
###           remote chart           ###
########################################
werf-deploy-remote-chart:
  extends: .werf-deploy-remote-chart
  variables:
    GITLAB_AGENT_PROJECT: "YOUR_PROJECT_WITH_GITLAB_AGENT"     # Example: group_name/gitlab-agent
    KUBERNETES_CLUSTER_NAME: "NAME_OF_YOUR_KUBERNETES_CLUSTER" # Operate -> Kubernetes clusters
    WERF_NAMESPACE: "$CI_PROJECT_NAME"
    REMOTE_CHART_NAME: "REMOTE_CHART_NAME"
    REMOTE_CHART_VERSION: "REMOTE_CHART_VERSION"
    REMOTE_REPO_ALIAS: "REMOTE_REPO_ALIAS"
    REMOTE_REPO_URL: "REMOTE_REPO_URL"
    WERF_PLAN: "true"                                          # dry-run without deploy
    REGISTRY_USER: "REGISTRY_USER"
    REGISTRY_PASSWORD: "REGISTRY_PASSWORD"
    WERF_ARGS: >-
      --values my_values.yaml

# To parameterize a dependent chart, you can use a file with parameters (values.yaml) in the format:
#  remote_chart_name:
#    param1: value1
#    param2: value2
#
# To create such a nesting, you can use the command:
# yq -n ".\"$REMOTE_CHART_NAME\" = load(\"tmp_values.yaml\")" > values.yaml

```
