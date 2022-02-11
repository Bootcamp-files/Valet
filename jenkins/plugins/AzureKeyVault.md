# AzureKeyVault

## Designer pipeline

This plugin is not supported in Designer pipelines.

## Jenkinsfile pipeline

### Jenkins input

```groovy
options {
    azureKeyVault([[envVariable: 'MY_SECRET', name: 'my-secret', secretType: 'Secret']])
}            
steps {
    echo 'Found $MY_SECRET'
}
```

### Transformed Github Action

```yaml
- uses: Azure/login@v1
  with:
    creds: "${{ secrets.AZURE_CREDENTIALS }}"
- uses: Azure/get-keyvault-secrets@v1
  with:
    keyvault: "${{ secrets.AZURE_KEYVAULT }}"
    secrets: my-secret
  id: azure_keyvault
- name: set job envs
  shell: bash
  run: echo 'MY_SECRET=${{ steps.azure_keyvault.outputs.my-secret }}' >> $GITHUB_ENV
- name: checkout
  uses: actions/checkout@v2
- name: echo message
  run: echo Found $MY_SECRET
```

### Unsupported Options

- None
