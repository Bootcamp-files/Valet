# NPM task

## Azure DevOps input

```json
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
                              "packageType"            : { "id" : "npm", "name" : "npm" },
                              "skipextract"            : { "id" : "", "name" : "" },
                              "view"                   : { "id" : "", "name" : "" }
                            },
  "isRetained"          : false
}

```

### Transformed Github Action

```yaml
  # Ensure that you have a `.npmrc` file configured with the following: `@${{ secrets.PACKAGES_OWNER }}:registry=https://npm.pkg.github.com`
- name: Authenticate GitHub Packages
  run: echo '//npm.pkg.github.com/:_authToken=${{ secrets.PACKAGES_TOKEN }}' > ~/.npmrc
- name: Install NPM packages
  run: npm install @${{ secrets.PACKAGES_OWNER }}/revolution@2.0.0 --registry=https://npm.pkg.github.com/${{ secrets.PACKAGES_OWNER }}
```

### Unsupported inputs

- View
