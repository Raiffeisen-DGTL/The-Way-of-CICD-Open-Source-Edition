# Pipeline Modules for Executing Various Tasks: Notifications, Time Tracking in Pipeline, Working with Certificates, etc.

## Description

This section contains various auxiliary templates for the pipeline, including the following functionality:

- Time tracking in the pipeline using `.time_tracker`.

- Obtaining certificates inside the pipeline for Unix servers and more using `.get_certificate`. This step uses [Vaultbot](https://gitlab.com/msvechla/vaultbot) for certificate retrieval.

- Retrieving secrets from Vault using `.get_vault_creds`. This step allows you to obtain two entries using two keys from Vault (login and password) and set them as environment variables (to do this, you MUST execute `source vault.env` - see examples).

## Required Variables for `.get_vault_creds`
```yaml
.get_vault_creds:
VAULT_ADDRESS # Vault project area address
VAULT_PATH_TO_SECRET # Path to the secret from the root storage (e.g., kv/secret or stage/service/secret - any level of nesting)
LOGIN_FIELD # Login field. Defaults to 'login'. It's better to specify it explicitly to avoid unexpected behavior in GitLab when merging variables.
PASSWORD_FIELD # Password field. Defaults to 'password'. Similarly to the login field, it's better to specify it explicitly.
```

# Usage Examples in .gitlab-ci.yml

```yaml
########################################
###      Include template job        ###
###     .release from this repo      ###
########################################
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'master'
    file:
      - '/pipeline/v2/pipeline.yml'

########################################
###   Create job in your pipeline    ###
###     extending included job       ###
########################################
sonarqube:
  stage: code-scan
  extends:
    - .time_tracker
    - .sonarqube-maven

########################################
###    Or construct your own job     ###
###       using reference tags       ###
########################################
sonarqube:
  stage: code-scan
  before_script:
    - !reference [.time_tracker, before_script]   # add time-tracking
  ...

########################################
###       Obtaining Certificates     ###
###       Inside a GitLab Job        ###
########################################
.get_certificate:
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
    PKI_ROLE_NAME: "$PKI_ROLE_NAME"

######################################
###       Obtaining Secrets        ###
###       Inside a GitLab Job      ###
######################################
use_vault:
  stage: vault
  variables:
    VAULT_ADDRESS: "https://your.vault.path.com"
    VAULT_PATH_TO_SECRET: "kv"
    LOGIN_FIELD: "login"
    PASSWORD_FIELD: "password"
  extends:
    - .get_vault_creds
  script:
    - source vault.env
    - echo $LOGIN

# If you need the login and password from Vault to be available in other stages of the pipeline, you need to explicitly specify the use of the same cache:

use_vault:
  stage: vault
  variables:
    VAULT_ADDRESS: "https://your.vault.path.com"
    VAULT_PATH_TO_SECRET: "kv"
    LOGIN_FIELD: "login"
    PASSWORD_FIELD: "password"
  extends:
    - .get_vault_creds
  script:
    - source vault.env
    - echo $LOGIN


use_vault_again:
  image: $YOUR_FAV_IMG
  stage: another_stage
  script:
    - source vault.env
    - echo $LOGIN
  cache:
    paths:
      - vault.env


# Use !reference for more flexibility:

get_another_secret:
  image: $YOUR_FAV_IMG
  stage: third_stage
  variables:
    VAULT_ADDRESS: "https://your.vault.path.com"
    VAULT_PATH_TO_SECRET: "kv"
    LOGIN_FIELD: "login"
    PASSWORD_FIELD: "password"
  before_script:  
    - !reference [.get_vault_creds, before_script]
  script:
    - source vault.env
    - echo $LOGIN


```
# Pipeline Modules for Determining Registry

## Description

This section contains a description of the stage for determining the target repository in the registry.

## GitLab Environment Variables Used in the Pipeline
```yaml
REGISTRY_HOST: "Your release docker registry"
SNAPSHOT_REGISTRY_HOST: "Your temp registry for snapshots"
```

## Integration into Your Project

```yaml
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: 'master'
    file:
      - '/define-registry/v1/define-registry.yml'

stages:
  - define_registry
```


# Usage Examples in .gitlab-ci.yml

```yaml
define_registry:
  extends: .define_registry
```
