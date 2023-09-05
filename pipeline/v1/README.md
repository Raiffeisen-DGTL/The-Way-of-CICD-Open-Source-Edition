# Pipeline Modules for Performing Various Tasks: Notifications, Pipeline Time Tracking, Certificate Management, etc.

## Description

This section contains various auxiliary templates for the pipeline. Here's what functionality is available:

- The `.slack` template is designed to send messages to Slack in the `after_script` stage. By default, the message is formed according to the template `${SLACK_UID_TAG}$STATUS_ICON $CI_PROJECT_TITLE ($CI_COMMIT_REF_SLUG) $CI_JOB_NAME $CI_JOB_STATUS`, where the value for the `$SLACK_UID_TAG` variable will be found by searching for the email address of the user who started the pipeline in the `SLACK_UIDS` variable. Additionally, by default, the message will only be sent in the event of a job failure in which the script is executed. To send a message to Slack for any job status, use the `SLACK_FAIL_ONLY` variable with the value `false`.

- Time tracking in the pipeline `.time_tracker`
- Obtaining certificates within the pipeline for Unix servers and more `.get_certificate`
Vaultbot is used to obtain certificates.

## External Variables Used in Templates

```yaml
.slack:
SLACK_UIDS
SLACK_WEBHOOK
```

## Variables with Default Values That Can Be Overridden
```yaml
.slack:
SLACK_FAIL_ONLY: "true"
SLACK_MESSAGE: "${SLACK_UID_TAG}$STATUS_ICON $CI_PROJECT_TITLE ($CI_COMMIT_REF_SLUG) $CI_JOB_NAME $CI_JOB_STATUS"
SYS_PROXY: "YOUR.PROXY.IF.EXISTS:8080"
```

## GitLab Environment Variables Used in the Pipeline
```yaml
.slack:
CI_COMMIT_REF_SLUG
CI_JOB_NAME
CI_JOB_STATUS
CI_PIPELINE_URL
CI_PROJECT_TITLE
GITLAB_USER_EMAIL
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
      - '/pipeline/v1/pipeline.yml'

########################################
###   Create job in your pipeline    ###
###     extending included job       ###
########################################
sonarqube:
  stage: code-scan
  extends:
    - .time_tracker
    - .sonarqube-maven
    - .slack # this line only

########################################
###    Or construct your own job     ###
###       using reference tags       ###
########################################
sonarqube:
  stage: code-scan
  before_script:
    - !reference [.time_tracker, before_script]   # Add time-tracking
  ...
  after_script: !reference [.slack, after_script]

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
```
