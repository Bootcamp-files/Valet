# Bash task

## Azure DevOps input

```yaml
steps:
- task: ShellScript@2
  inputs:
    scriptPath:
    # args: # Optional
    # disableAutoCwd: false # optional
    # cwd:  # Optional
    # failOnStderr: false # Optional
```

```yaml
# Bash
# Run a Bash script on macOS, Linux, or Windows
- task: Bash@3
  inputs:
    #targetType: 'filePath' # Optional. Options: filePath, inline
    #filePath: # Required when targetType == FilePath
    #arguments: # Optional
    #script: '# echo Hello world' # Required when targetType == inline
    #workingDirectory: # Optional
    #failOnStderr: false # Optional
    #noProfile: true # Optional
    #noRc: true # Optional
```

### Transformed Github Action

```yaml
- run: sudo npm install -g appcenter-cli@1.1.20
  shell: bash
```

### Unsupported inputs

- failOnStderr
