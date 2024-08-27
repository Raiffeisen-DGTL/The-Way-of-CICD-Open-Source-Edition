- [Variables with default values that can be overridden](#variables-with-default-values-that-can-be-overridden)
- [GitLab environment variables used in templates](#gitlab-environment-variables-used-in-templates)
- [External variables used in templates](#external-variables-used-in-templates)
- [Usage example](#usage-example)

## Variables with default values that can be overridden
```yaml
PYTHON_IMAGE
```

## GitLab environment variables used in templates
```yaml
.python-cache:
$CI_PROJECT_NAME
```

## External variables used in templates
```yaml
.python-cache:
REGISTRY_USER     # Hide this in Settings -> CI/CD -> Variables
REGISTRY_PASSWORD # Hide this in Settings -> CI/CD -> Variables
LOCAL_POETRY_ARGS # Pay special attention to this variable, if you want to add more than one command, put a separator between them ;
```

## Usage example
```yaml
include:
  - project: 'devops_community/the-way-of-cicd'
    ref: 'Major.minor.patch' # Please fill in with the desired tag
    file:
      - 'build/build-package.yml'

stages:
  - pre-build

define-cache:
  stage: pre-build
  extends: .python-cache
```