# Slack Notification

## Designer pipeline

This plugin is not supported in Designer pipelines.

## Jenkinsfile pipeline

### Jenkins input

```groovy
    slackSend channel: '#general',
        color: COLOR_MAP[currentBuild.currentResult],
        message: "Hello World!!"

### Transformed Github Action

```yaml
- name: Slack Notification
  uses: rtCamp/action-slack-notify@v2
  env:
     SLACK_WEBHOOK: "${{ secrets.SLACK_WEBHOOK }}"
     SLACK_CHANNEL: "#general"
     SLACK_MESSAGE: Hello World!!

### Unsupported Options

- SLACK_MSG_AUTHOR
- SLACK_ICON
- SLACK_ICON_EMOJI
- SLACK_COLOR
- SLACK_TITLE
- SLACK_FOOTER
- MSG_MINIMAL
