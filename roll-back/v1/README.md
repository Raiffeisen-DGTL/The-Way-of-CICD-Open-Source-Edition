# Pipeline Modules for Application Rollback Stages in Case of Failures

## Description

This section contains a pipeline module for rollback stages of application versions.  
In this implementation, the rollback will occur to the previous version of the application.  
You can view all versions of releases deployed using `helm` with the command `helm history $RELEASE_NAME -n $NAMESPACE`

## Examples of Usage in .gitlab-ci.yml

```yaml
########################################
###     Include templated jobs       ###
###           from this repo         ###
########################################
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'master'
    file:
      - '/roll-back/v1/roll-back.yml'

########################################
###        Roll-back with helm       ###
###            via extends           ###
########################################
roll-back:
  stage: roll-back
  before_script: 
    - export KUBECONFIG="$K8S_CONFIG"
  extends:
    - .roll-back-helm
  variables:
    NAMESPACE: "$CI_PROJECT_NAME"
    RELEASE_NAME: "$CI_PROJECT_NAME"
  rules:
    - when: manual

########################################
###       Roll-back with helm        ###
###        using reference tag       ###
########################################
deploy-via-helm:
  stage: roll-back
  image: [.roll-back, image]
  variables:
    K8S_CONFIG: "YOUR_CONFIG" # add kubeconfig to your Settings -> CI/CD -> Variables
    RELEASE_NAME: "$CI_PROJECT_NAME"
    NAMESPACE: "$CI_PROJECT_NAME"
  script:
    - !reference [.roll-back, fetch_kubeconfig]
    - !reference [.roll-back, helm-roll-back]
```
