# Sql Azure Dacpac Deployment task

## Azure DevOps input

```yaml
- task: SqlAzureDacpacDeployment@1
  inputs:
    azureConnectionType: 'ConnectedServiceName'
    azureClassicSubscription: 'azure-temp'
    AuthenticationType: 'connectionString'
    ConnectionString: '$(connection_string)'
    deployType: 'DacpacTask'
    DeploymentAction: 'Publish'
    DacpacFile: 'AwesomeCompany.dacpac'
    IpDetectionMethod: 'AutoDetect'
```

## Transformed Github Action

```yaml
- uses: Azure/login@v1
  with:
    creds: "${{ secrets.AZURE_CREDENTIALS }}"
# if 'Allow Azure Services and resources to access this server' is ON the Azure/login step above can likely be removed
- name: Azure SQL Deploy
  uses: Azure/sql-action@v1
  with:
    server-name: "${{ env.SERVER_NAME }}"
    connection-string: "${{ secrets.AZURE_SQL_CONNECTION_STRING }}"
    dacpac-package: AwesomeCompany.dacpac
  env:
    SERVER_NAME: UPDATE_ME
```

## Unsupported inputs and aliases
- deploymentAction: Extract, Export, Import, Script, DriftReport, and DeployReport
- publishProfile
- authenticationType: server, aadAuthenticationPassword, aadAuthenticationIntegrated, and servicePrincipal

## Additional Information
- Connection to Azure is handled by the Azure/login action and can be removed if not required to access Server
- Only supported authentication type is connectionString

