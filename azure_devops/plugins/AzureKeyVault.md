# Azure Key Vault

## Azure DevOps input

```yaml
- task: AzureKeyVault@2
  inputs:
    azureSubscription: 'Your-Azure-Subscription'
    KeyVaultName: 'Your-Key-Vault-Name'
    SecretsFilter: '*'
    RunAsPreJob: false
```

### Transformed Github Action

```yaml
- uses: Azure/login@v1
  with:
    creds: '${{ secrets.AZURE_CREDENTIALS }}'

- uses: Azure/get-keyvault-secrets@v1
  with:
    keyvault: 'Your-Key-Vault-Name'
    secrets: '*'
```

### Unsupported inputs and aliases

- azureSubscription
- ConnectedServiceName
