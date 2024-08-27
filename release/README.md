# Pipeline Modules for Release Stages

## Description

This section contains a pipeline module for creating a release in GitLab and a tag in Git.

## Changes in version v2

- Now the `release` task has a new version of the image `v0.16.0`.
- Now the `release` task is executed automatically when pushing to the `$CI_DEFAULT_BRANCH` branch.
- - In the `release` task, a check for the presence of an existing version in the repository has been added to avoid a crash when restarting `$CI_DEFAULT_BRANCH`, if there is no tag in the repository, the tag will be created.

## GitLab environment variables used in the pipeline

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
      - 'release/release.yml'

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
    - when: never
```
