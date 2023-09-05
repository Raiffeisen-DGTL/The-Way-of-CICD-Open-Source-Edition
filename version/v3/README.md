# Pipeline Modules for Artifact Versioning

## Description

This section contains a pipeline module for the stages of versioning various artifacts. Currently, the only implemented option is semantic versioning based on git history using the gitversion utility. You can read more about it here: [GitVersion Documentation](https://t.ly/R4pr). To explicitly specify the use of versioning with gitversion, it is recommended to place a GitVersion.yml file in the root of your repository, but this is not mandatory. In the absence of this file, it will be automatically generated. By default, MAJOR_VERSION is considered to be 0, and to change it, it is recommended to create a majorVersion file in the repository's root and specify the desired version number inside.

## GitLab Environment Variables Used in the Pipeline
```yaml
$CI_COMMIT_BRANCH
$CI_DEFAULT_BRANCH
$CI_COMMIT_REF_SLUG
$CI_COMMIT_SHORT_SHA
```

## Usage Examples in .gitlab-ci.yml
```yaml
# Include .verson template job from this repo
########################################
###     Include .verson template     ###
###       job from this repo         ###
########################################
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'master'
    file:
      - '/version/v3/version.yml'

########################################
###   Create job in your pipeline    ###
###     extending included job       ###
########################################
version:
  stage: version
  extends:
    - .version_git

########################################
###    Or construct your own job     ###
###       using reference tags       ###
########################################
version:
  stage: version
  image: !reference [.version_git, image]
  variables:
    GIT_STRATEGY: clone
    GIT_DEPTH: 0
  before_script:
    - !reference [.version_git, before_script]
  script:
    - !reference [.version_git, script]
  artifacts:
    reports:
      dotenv:
        - ./version.env
```
