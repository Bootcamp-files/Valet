# Powershell task

## Azure DevOps input

```yaml
# Run a PowerShell script on Linux, macOS, or Windows
- task: PowerShell@2
  inputs:
    filePath: ./build.ps1 # Required when targetType == FilePath
    arguments: '-Name someName -Path -Value "Some long string value"' # Optional
    #targetType: 'filePath' # Optional. Options: filePath, inline
    #script: '# Write your PowerShell commands here.Write-Host Hello World' # Required when targetType == Inline
    #errorActionPreference: 'stop' # Optional. Options: stop, continue, silentlyContinue
    #failOnStderr: false # Optional
    #ignoreLASTEXITCODE: false # Optional
    #pwsh: false # Optional
    #workingDirectory: # Optional
```

### Transformed Github Action

```yaml
- run: ./build.ps1 -Name someName -Path -Value \"Some long string value\"
  shell: powershell
```

### Unsupported inputs

- failOnStderr
