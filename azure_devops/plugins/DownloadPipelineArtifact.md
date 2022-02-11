# DownloadPipelineArtifact task

## Azure DevOps input

```yaml
# Download build and pipeline artifacts
- task: DownloadPipelineArtifact@2
  inputs:
    source: "current" # Options: current, specific
    artifact: "drop" # Optional
    path: "out"
    #project: # Required when source == Specific
    #pipeline: # Required when source == Specific
    #preferTriggeringPipeline: false # Optional
    #runVersion: 'latest' # Required when source == Specific# Options: latest, latestFromBranch, specific
    #runBranch: 'refs/heads/master' # Required when source == Specific && RunVersion == LatestFromBranch
    #runId: # Required when source == Specific && RunVersion == Specific
    #tags: # Optional
    #patterns: '**' # Optional
```

### Transformed Github Action

```yaml
- uses: actions/download-artifact@v2
  with:
    name: drop
    path: out
```

### Unsupported inputs

- source (specific)
- project
- preferTriggeringPipeline
- runVersion
- runBranch
- runId
- tags
- patterns
