# Pipeline Modules for Release Stages

## Description

This section contains a pipeline module for creating a release in GitLab and a tag in Git.

## GitLab Environment Variables Used in the Pipeline
```yaml
$CI_SERVER_URL
$CI_JOB_TOKEN
$CI_PROJECT_ID
$CI_PROJECT_NAME
$CI_COMMIT_SHA
```

## Examples of Usage in .gitlab-ci.yml

```yaml
########################################
###      Include template job        ###
###     .release from this repo      ###
########################################
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'master'
    file:
      - '/release/v1/release.yml'

########################################
###   Create job in your pipeline    ###
###     extending included job       ###
########################################
create-release:
  stage: release
  extends:
    - .release
  variables:
    REGISTRY_HOST: #default points to hub.docker.com
    REGISTRY_PATH: "" #relative path on your registry
    VERSION: "1.0.0"
    ASSET_URL: "https://${REGISTRY_HOST}/${REGISTRY_PATH}/${CI_PROJECT_NAME}"
    DESCRIPTION: "Artifact can be reached using the following link ${ASSET_URL}"
  rules:
    - if: $CI_COMMIT_BRANCH == "$CI_DEFAULT_BRANCH"
      when: manual
    - when: never

########################################
###    Or construct your own job     ###
###       using reference tags       ###
########################################
create-release:
  stage: release
  variables:
    REGISTRY_HOST: ""
    REGISTRY_PATH: ""
    VERSION: "1.0.0"
    ASSET_URL: "https://${REGISTRY_HOST}/${REGISTRY_PATH}/${VERSION}"
    DESCRIPTION: "Artifact can be reached using the following link ${ASSET_URL}"
  image: !reference [.release, image]
  script: !reference [.release, script]
  rules:
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
      when: manual
    - when: never
```
