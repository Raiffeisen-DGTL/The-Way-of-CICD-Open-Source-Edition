- [Introduction](#introduction)
- [External Variables Used in Templates](#external-variables-used-in-templates)
- [Variables with Default Values That Can Be Overridden](#variables-with-default-values-that-can-be-overridden)
- [GitLab environment variables used in templates](#gitlab-environment-variables-used-in-templates)
- [NPM and Kaniko Use case in .gitlab-ci.yml](#npm-and-kaniko-use-case-in-gitlab-ciyml)

## Introduction

When building using `npm` in the `.build_npm` template, the `.npmrc` file is generated with authentication data for accessing the `Registry`, then the necessary modules are downloaded using the `npm install` command, and then the assembly takes place using `npm run`.

Authentication data is created based on the variables `REGISTRY_USER` and `REGISTRY_PASSWORD`, which it is advisable to set in the variables `Gitlab CI/CD` at the repository level of the project or group.

It also supports working with scope, for this you can specify the necessary scope in the `NAME_SCOPE` variable in the format `scope name:repository url`, for example:
```yaml
variables:
  NPM_SCOPE: "@inv:path.to.your.registry/api/npm/investment-release-npm/ @fcc:path.to.your.registry/api/npm/fcc-d-npm/"
```

After that, additional lines will be added to the `.npmrc` file, while the variables `REGISTRY_USER` and `REGISTRY_PASSWORD` will also be used to generate authentication data for all scope:
```yaml
@inv:registry=https://path.to.your.registry/api/npm/investment-release-npm/
//path.to.your.registry/api/npm/investment-release-npm/:_auth=[AUTH]
@fcc:registry=https://path.to.your.registry/api/npm/fcc-d-npm/
//path.to.your.registry/api/npm/fcc-d-npm/:_auth=[AUTH]
```

When executing the `npm install` command, you can specify additional packages to install using the `RPM_PACKAGES` variable, as well as additional parameters to run using the `NPM_OPTIONS` variable:
```bash
npm install $NPM_PACKAGES $NPM_OPTIONS
```

When running `npm run`, you can specify the command using the `NUM_COMMAND` variable, which by default has the value `build`:
```bash
npm run $NPM_COMMAND
```

Also, to speed up the build, the cache for the folder `node_modules/` is connected in the job template `.build_npm` using the key `CI_PROJECT_NAME`.

## External Variables Used in Templates

```yaml
.build_npm:
NPM_REPO
NPM_SCOPE
NPM_PACKAGES
NPM_OPTIONS
NPM_COMMAND
REGISTRY_USER     # Hide this in Settings -> CI/CD -> Variables
REGISTRY_PASSWORD # Hide this in Settings -> CI/CD -> Variables
```
## Variables with Default Values That Can Be Overridden

```yaml
.build_npm:
NPM_REPO: "path.to.your.registry/api/npm/npm/"
NPM_OPTIONS: "--save --save-exact --timing --loglevel verbose"
NPM_COMMAND: "build"
```

## GitLab environment variables used in templates
```yaml
.build_npm:
$CI_PROJECT_NAME      # Used as a key for cache
```

## NPM and Kaniko Use case in .gitlab-ci.yml

```yaml
########################################
###      Include template jobs       ###
###  .build_npm, .build_kaniko       ###
###       and .version_git           ###
########################################
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'Major.minor.patch' # Please fill in with the desired tag
    file:
      - 'build/build-package.yml'
      - 'version/version.yml'

########################################
###   Create jobs in your pipeline   ###
###     extending included jobs      ###
########################################
stages:
  - package
  - version
  - build
  - deploy

npm_build:
  stage: package
  extends:
    - .build_npm
  variables:
    NPM_SCOPE: "@inv:path.to.your.npm.repo/api/npm/investment-release-npm/ @fcc:path.to.your.npm.repo/api/npm/fcc-d-npm/"
    NPM_PACKAGES: "webpack"
  artifacts:
    paths:
      - dist/

version:
  stage: version
  extends:
    - .version_git

build:
  stage: build
  extends:
    - .build_kaniko
  variables:
    REGISTRY_PATH: #relative path on your registry
    IMAGE_NAME: "$CI_PROJECT_NAME"
    IMAGE_TAG: "$VERSION"
```


To build an OCI image, you will need to have a Dockerfile in the project repository, an example of a Dockerfile:

```Dockerfile
FROM docker.io/library/nginx:1.23.3
USER root
COPY dist/ /usr/share/nginx/html
RUN chown nginx -R /usr/share/nginx/html/
USER nginx
```
