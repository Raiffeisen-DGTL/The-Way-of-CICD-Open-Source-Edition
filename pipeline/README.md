# Pipeline Modules for Executing Various Tasks: Notifications, Time Tracking in Pipeline, Working with Certificates, Vault and etc.

## Description

This section contains various auxiliary templates for the pipeline, including the following functionality:

1. Time tracking in the pipeline using `.time_tracker`.

2. Obtaining certificates inside the pipeline for Unix servers and more using `.get_certificate`. This step uses [Vaultbot](https://gitlab.com/msvechla/vaultbot) for certificate retrieval.

3. Getting secrets from Vault `.get_any_vault_secrets`. This step allows you to get the specified list of secrets from the Vault with any keys from any secret engines and connect them as environment variables.
   - **The format for specifying the list of secrets** is given [in the description of the necessary variables](#necessary-variables-for-work-get_any_vault_secrets).
   - **The names of the environment variables** will be of the form `PATH_TO_SECRET` + `_` + `FIELD_NAME`. For a secret with the path `kv/secret1` and the field `field1` inside this secret, the resulting variable will have the name `KV_SECRET1_FIELD1`.
   
4. Adding annotations to the dashboard in the grafana `.grafana_annotations`. Allows you to add an annotation to record an event. For example, after the deployment is completed or to display the start and stop of a process (CRQ work, start and end of a load test, full regression, etc.).

Description of the parameters:

```yaml
REGISTRY_USER  # The username for deleting versions. Must have RW rights
REGISTRY_PASSWORD  # User password
REPOSITORY_NAME  # The repository where the deletion will be performed
REPOSITORY_TYPE  # Repository type
ARTIFACTS_PATH - # the full path to the artifact or the path to the folder with artifacts whose versions need to be deleted (it is advisable to use in conjunction with the ARTIFACTS_TO_KEEP_REGEX variable). You can set the folder hierarchy in this variable. For example, when specifying `ARTIFACTS_PATH`: REGISTRY, a search will be performed for all artifacts that lie inside the following path: REGISTRY/*.
ARTIFACTS_TO_KEEP  # Versions that need to be save
ARTIFACTS_TO_KEEP_REGEX  # the regular expression corresponding to the saved versions (will delete everything that does not match the regular expression )
ARTIFACTS_TO_KEEP_COUNT  # the number of artifact versions that need to be left in addition to those listed in ARTIFACTS_TO_KEEP
ARTIFACTS_TO_KEEP_COUNT_NOT_REGEX  # (optional) number of images that do NOT correspond to regex that need to be saved (N of the most recent images are saved (by artifact update date))  
```

## Usage Examples `.time_tracker` in .gitlab-ci.yml

```yaml
########################################
###      Include template job        ###
###     .release from this repo      ###
########################################
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'Major.minor.patch' # Please fill in with the desired tag
    file:
      - 'pipeline/pipeline.yml'

########################################
###      Construct your own job      ###
###       using reference tags       ###
########################################
sonarqube:
  stage: code-scan
  before_script:
    - !reference [.time_tracker, before_script]  # enabling time logging
  ...
```

## Usage Examples `.get_certificate` in .gitlab-ci.yml

```yaml
########################################
###       Obtaining Certificates      ###
###       Inside a GitLab Job        ###
########################################
get_certificate:
  stage: get-cert
  extends:
    - .get_certificate
  variables:
    VAULT_ADDRESS: "https://path.to.your.pki:8200"
    PKI_ROLE_ID: "$PKI_TEST_ROLE_ID"
    PKI_SECRET_ID: "$PKI_TEST_SECRET_ID"
    PKI_COMMON_NAME: "test.example.com"
    PKI_ALT_NAMES: "test2.example.com,test3.example.com"
    PKI_TTL: "8760h"
```

## Necessary variables for operation `.get_any_vault_secrets`

```yaml
get_any_vault_secrets:
VAULT_ADDR # Vault project area address
VAULT_SECRETS # A list of secrets to be obtained. It is set in the following format:
              # VAULT_SECRETS: |
              #   path: test/secret1
              #   fields:
              #     - username
              #     - password
              #     - url
              #   path: kv/secret2
              #   fields:
              #     - login
              #     - host

```

## Usage Examples `.get_any_vault_secrets` in .gitlab-ci.yml

```yaml
#############################################
###    An example of getting different    ###
### secrets from different secret engines ###
#############################################
# An example of using a template to get secrets from the Vault.
# In this example, we get secrets from two different secret engines,
# And connect them as environment variables.
get_my_vault_secrets:
  extends: .get_any_vault_secrets
  stage: vault
  variables:
    VAULT_SECRETS: |
      path: kv/secret123
      fields:
        - username
        - url
        - password
      path: anotherkv/secret456
      fields:
        - surname        
# As a result, we will get the following variables:
# - KV_SECRET123_USERNAME
# - KV_SECRET123_URL
# - KV_SECRET123_PASSWORD
# - ANOTHERKV_SECRET456_SURNAME

############################################
###  Example using $CI_ENVIRONMENT_NAME  ###
############################################
# This example can be used if you want to get the same secrets for different environments,
# And at the same time in your Vault they are decomposed into secret engines with names equal to the names of the environments.
.get_my_vault_secrets_by_environment:
  extends: .get_any_vault_secrets
  stage: vault
  variables:
    VAULT_SECRETS: |
      path: $CI_ENVIRONMENT_NAME/secret1
      fields:
        - user
        - password
        - otherfield

get_my_secrets_test:
  extends: .get_my_vault_secrets_by_environment
  environment:
    name: "test"

get_my_secrets_preview:
  extends: .get_my_vault_secrets_by_environment
  environment:
    name: "preview"

# If it is required that the login and password from the vault are available in other pipeline steps, 
# then you need to explicitly specify the use of the same cache using the construction:
#       cache:
#         - key: $CI_PIPELINE_ID
#           paths:
#            - vault.env
# And execute in the script:
#         source vault.env
get_my_vault_secrets:
  extends: .get_any_vault_secrets
  stage: vault
  variables:
    VAULT_SECRETS: |
      path: engine1/secret1
      fields:
        - field1

use_vault_again:
  image: some_image
  stage: another_stage
  script:
    - source vault.env
    - echo $ENGINE1_SECRET1_FIELD1
  cache:
    - key: $CI_PIPELINE_ID
      paths:
        - vault.env
```

## Necessary variables for operation `.grafana_annotations`.

```yaml
GRAFANA_USER # the name of the slot to connect to the grafana
GRAFANA_PASSWORD # the password from to connect to the graphana
GRAFANA_TEXT # annotation text
GRAFANA_ORG ID # org id that the user has access to
DASHBOARD_LIST # list of boards where the annotation should be displayed
GRAFANA_TAGLIST # a list of tags that will be indicated in the annotation
GRAFANA_URL # url of the grafana for the corresponding contour
```

## Usage Examples `.grafana_annotations` in .gitlab-ci.yml

```yaml
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'Major.minor.patch' # Please fill in with the desired tag
    file:
      - 'pipeline/pipeline.yml'

# ad via extend
annotation:
  stage: annotation
  extends:
    - .grafana_annotations
  variables:
    GRAFANA_TEXT: "Deploy ${CI_PROJECT_NAME}"
    GRAFANA_ORGID: 331  # enter your orgid from grafana
    DASHBOARD_LIST: >
      5681
      5680
    GRAFANA_TAGLIST: >
      deploy
      preview
    GRAFANA_URL: https://path.to.grafana.ru

# announcement via references
annotation:
  stage: annotation
  image: alpine:3.18.3
  variables:
    GRAFANA_ORGID: 331
    DASHBOARD_LIST: 5681 5680  # here and below are various examples of passing the list.
    GRAFANA_TAGLIST: >
      fulltest
      sometag
    GRAFANA_URL: https://path.to.grafana.ru
  before_script:
    - export GRAFANA_TEXT="Start job ${CI_PROJECT_NAME} on ${CI_JOB_ID}"
    - !reference [.grafana_annotations, script]
  after_script:
    - export GRAFANA_TEXT="Stop job ${CI_PROJECT_NAME} on ${CI_JOB_ID}"
    - !reference [.grafana_annotations, script]
  script:
    - echo "Long time job"
    - sleep 90
    - echo "done"
```
