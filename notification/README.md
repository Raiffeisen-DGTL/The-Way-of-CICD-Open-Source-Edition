# Pipeline constructor for notification stages in the Mattermost channel after the deployment stage

## Description 

This section contains a pipeline constructor for notification stages in the Mattermost channel after the deployment stage.

## Usage
You can use the `extends` directive if you are satisfied with the described scenario in the existing pipeline.
If you need to extend an existing pipeline, you can use the constructor from `!reference [tags]` to add your own customized improvements.

## Mandatory conditions and preliminary steps

### Requirement:

1. Channel availability in Matter most;
2. The bot that the hook is installed on must be a member of the channel;
3. Created hook;

### Preliminary steps:

1. Creating a bot - [via Mattermost](https://developers.mattermost.com/integrate/reference/bot-accounts/);
2. Creating a channel
3. Creature hook ([instruction](https://developers.mattermost.com/integrate/webhooks/incoming/)) :
```yaml
# create webhook
brew install mmctl # install mmctl
mmctl auth login https://your.mattermost.server --name YOURNAME --access-token <bot_token> # auth to Mattermost as bot
mmctl webhook create-incoming  --user <bot_name> --channel <channel_id> --display-name <channel_name> # create webhook for channel

# nice commands
mmctl webhook list # list all webhooks which bot created
mmctl webhook delete <webhook_name> # delete webhook

# check webhook
curl -X POST -d 'payload={"text": "Check!"}' https://path.to.your.mettermost/hooks/<webhook_id>
```

## External variables used in templates
```yaml
.notification:
MATTERMOST_ADDRESS # Path to Mattermost server, the variable already specified, don't change it.
MATTERMOST_HOOK # Hide this in Settings -> CI/CD -> Variables
MATTERMOST_MESSAGE # Your custom to Mattermost
```

## Examples of use in .gitlab-ci.yml

```yaml
########################################
###      Include template jobs       ###
###         from this repo           ###
########################################
include:
  - project: 'path-to-gitlab-repo/the-way-of-cicd-open-source-edition'
    ref: master
    file:
      - 'notification/notification.yml'

########################################
###             Notification         ###
###       Using extends example      ###
########################################
notification:
  stage: notification
  extends:
    - .send_notification
  variables:
    MATTERMOST_HOOK: "<webhook_id>" # Hide it!
    MATTERMOST_MESSAGE: "Set up an alert!" # Get creative!
  needs:
    - job: deploy
      artifacts: false

########################################
###             Image scan           ###
###  Using !reference tags example   ###
########################################
notification:
  stage: notification
  image: !reference [.send_notification, image]
  variables:
    MATTERMOST_ADDRESS: path.to.your.mettermost/hooks/
    MATTERMOST_HOOK: "<webhook_id>" # Hide it!
    MATTERMOST_MESSAGE: "Set up an alert!" # Get creative!
  tags: !reference [.send_notification, tags]
  script: 
    - !reference [.send_notification, script]
  allow_failure: !reference [.send_notification, allow_failure]
