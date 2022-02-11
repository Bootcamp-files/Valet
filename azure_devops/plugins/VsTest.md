# VSTest task

## Azure DevOps input

```yaml
- task: VSTest@2
  inputs:
    testSelector: 'testAssemblies'
    testAssemblyVer2: |
      **\*Test*.dll
      !**\*TestAdapter.dll
      !**\obj\**
    searchFolder: '$(System.DefaultWorkingDirectory)'
    testFiltercriteria: 'ghc'
    platform: 'x64'
    publishRunAttachments: false

- task: VSTest@1
  inputs:
    testAssembly: |
      **\*Test*.dll
      !**\*TestAdapter.dll
      !**\obj\**
    codeCoverageEnabled: true
```


## Transformed Github Action

```yaml
- name: Setup VSTest Path
  uses: darenm/Setup-VSTest@v1
- name: Install Dependencies for VSTest
  shell: pwsh
  run: |-
    Set-PSRepository PSGallery -InstallationPolicy Trusted
    Install-Module VstsTaskSdk -AllowClobber
- name: Executing VSTest
  continue-on-error: true
  env:
    SEARCH_FOLDER: "${{ github.workspace }}"
    TEST_RESULTS_FOLDER: "${{ runner.temp }}/${{ github.run_id }}_resultsfolder"
    PATTERN: "**\\*Test*.dll;!**\\*TestAdapter.dll;!**\\obj\\**"
    ARGS: "/Logger:trx;/TestCaseFilter:ghc;/Platform:x64"
  shell: pwsh
  run: |-
    Import-Module -Name VstsTaskSdk
    $SearchPattern = $env:PATTERN.Split(';').Trim()
    Write-Host "Search Pattern: " $SearchPattern
    Write-Host "Finding files for VSTest to execute.."
    $files = Find-VstsMatch -DefaultRoot $env:SEARCH_FOLDER -Pattern $SearchPattern
    Write-Host "Files found:" $files
    $exe = "vstest.console.exe"
    $args = $env:ARGS.Split(';').Trim()
    Write-Host "Arguments are: " $args
    & $exe $files /ResultsDirectory:$env:TEST_RESULTS_FOLDER $args
- name: Upload Test Results
  env:
    RESULTS_FOLDER: "${{ runner.temp }}/${{ github.run_id }}_resultsfolder"
    uses: actions/upload-artifact@v2
  with:
    name: TestResultsFile
    path: "${{ env.RESULTS_FOLDER }}"
```

## Unsupported inputs and aliases
-testSelector:  Unsupported Options: testPlan, testRun
-testPlan: # Required when testSelector == TestPlan
-testSuite: # Required when testSelector == TestPlan
-testConfiguration: # Required when testSelector == TestPlan
-vstestLocationMethod: 'version' # Optional. Options: version, location
-vsTestVersion: 'latest' # Optional. Options: latest, 16.0, 15.0, 14.0,toolsInstaller
-distributionBatchType: 'basedOnTestCases' # Optional. Options: basedOnTestCases, basedOnExecutionTime, basedOnAssembly
-batchingBasedOnAgentsOption: 'autoBatchSize' # Optional. Options: autoBatchSize, customBatchSize
-customBatchSizeValue: '10' # Required when distributionBatchType == BasedOnTestCases && BatchingBasedOnAgentsOption == CustomBatchSize
-batchingBasedOnExecutionTimeOption: 'autoBatchSize' # Optional. Options: autoBatchSize, customTimeBatchSize
-customRunTimePerBatchValue: '60' # Required when distributionBatchType == BasedOnExecutionTime && BatchingBasedOnExecutionTimeOption == CustomTimeBatchSize
-dontDistribute: False # Optional
-failOnMinTestsNotRun: false # Optional
-minimumExpectedTests: '1' # Optional
-diagnosticsEnabled: false # Optional
-collectDumpOn: 'onAbortOnly' # Optional. Options: onAbortOnly, always, never
-rerunFailedTests: False # Optional
-rerunType: 'basedOnTestFailurePercentage' # Optional. Options: basedOnTestFailurePercentage, basedOnTestFailureCount
-rerunFailedThreshold: '30' # Optional
-rerunFailedTestCasesMaxLimit: '5' # Optional
-rerunMaxAttempts: '3' # Optional