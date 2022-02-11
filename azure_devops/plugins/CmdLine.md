# CmdLine task

## Azure DevOps input

```yaml
# Command line
# Run a command line script using Bash on Linux and macOS and cmd.exe on Windows
- task: CmdLine@2
  inputs:
    script: "echo Write your commands here."
    #workingDirectory: # Optional
    #failOnStderr: false # Optional
```

### Transformed Github Action

```yaml
- run: echo Write your commands here.
  #working-directory:
```

### Unsupported inputs

- failOnStderr
