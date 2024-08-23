- [Variables with default values that can be overridden](#variables-with-default-values-that-can-be-overridden)
- [GitLab environment variables used in templates](#gitlab-environment-variables-used-in-templates)
- [Usage example](#usage-example)

## Variables with default values that can be overridden
```yaml
.go-build:
GOOS: linux
GOARCH: amd64
CGO_ENABLED: 0
PACKAGE_NAME: $CI_PROJECT_NAME
GO_ARGS: ""
LD_FLAGS: ""
```

## GitLab environment variables used in templates
```yaml
.go-cache:
${CI_PROJECT_NAME}

.go-build:
${CI_PROJECT_NAME}
```

## Usage example
```yaml
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'Major.minor.patch' # Please fill in with the desired tag
    file:
      - 'build/build-package.yml'
      - 'version/version.yml'

stages:
  - pre-build
  - build

define-cache:
  stage: pre-build
  extends: .go-cache


build-app:
  extends: .go-build
  variables:
    PACKAGE_NAME: myapp ## Default is $CI_PROJECT_NAME
    LD_FLAGS: "-X main.version=$CI_COMMIT_TAG -X main.commit=$CI_COMMIT_SHA"
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
      when: never
    - when: always

```