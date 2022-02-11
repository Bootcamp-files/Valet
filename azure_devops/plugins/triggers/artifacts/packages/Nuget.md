# Nuget task

## Azure DevOps input

```JSON
{
  "sourceId"            : "97d2995f-4fe1-4175-9c24-effd87124bba/cb2b4bba-fcf7-4ea5-b953-129f0a2b7c97:revolution",
  "type"                : "PackageManagement",
  "alias"               : "_revolution",
  "definitionReference" :
                            {
                              "defaultVersionSpecific" : { "id" : "2.0.0", "name" : "2.0.0" },
                              "defaultVersionType"     : { "id" : "specificVersionType", "name" : "Specific version" },
                              "definition"             : { "id" : "revolution", "name" : "revolution" },
                              "feed"                   :
                                                          { "id" : "97d2995f-4fe1-4175-9c24-effd87124bba/cb2b4bba-fcf7-4ea5-b953-129f0a2b7c97", "name" : "begona_test_feed" },
                              "files"                  : { "id" : "**", "name" : "**" },
                              "packageType"            : { "id" : "nuget", "name" : "nuget" },
                              "skipextract"            : { "id" : "", "name" : "" },
                              "view"                   : { "id" : "", "name" : "" }
                            },
  "isRetained"          : false
}

```

### Transformed Github Action

```YAML
- name: Authenticate GitHub Packages
  run: dotnet nuget add source --username ${{ env.PACKAGES_USERNAME }} --password ${{ env.PACKAGES_TOKEN }} --store-password-in-clear-text --name github 'https://nuget.pkg.github.com/${{ env.PACKAGES_OWNER }}/index.json'
- name: Install Nuget packages
  run: nuget install revolution -Version 2.0.0
```

### Unsupported inputs

- View
