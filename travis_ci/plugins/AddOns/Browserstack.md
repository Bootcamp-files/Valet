# Browserstack

## Travis input

```yaml
browserstack: true
```

### Transformed Github Action

```yaml
- uses: browserstack/github-actions/setup-env@master
  with:
    username: "${{ secrets.BROWSERSTACK_USERNAME }}"
    access-key: "${{ secrets.BROWSERSTACK_ACCESS_KEY }}"
- uses: browserstack/github-actions/setup-local@master
  with:
    local-testing: start
- uses: browserstack/github-actions/setup-local@master
  with:
    local-testing: stop
```

### Unsupported Options

- username
- access_key
- forecelocal
- only
- app_path
- proxyHost
- proxyPort
- proxyUser
- proxyPass

NOTE: There may be additional setup for this action required. Please visit the [Browserstack Action](https://github.com/browserstack/github-actions/tree/master/setup-local) page to complete setup of the action.
