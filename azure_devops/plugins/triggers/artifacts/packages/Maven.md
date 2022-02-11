# Maven task

## Azure DevOps input

```json
 {
  "sourceId"            : "97d2995f-4fe1-4175-9c24-effd87124bba/8c510b21-db6b-491e-ada0-2d90aa61423f:1",
  "type"                : "PackageManagement",
  "alias"               : "_maven_artifact",
  "definitionReference" :
                            {
                              "defaultVersionSpecific" : { "id" : "1.0.0", "name" : "1.0.0" },
                              "defaultVersionType"     : { "id" : "latestType", "name" : "Latest" },
                              "definition"             : { "id" : "fix_world", "name" : "fix_world" },
                              "feed"                   : { "id" : "97d2995f-4fe1-4175-9c24-effd87124bba/8c510b21-db6b-491e-ada0-2d90aa61423f", "name" : "some-feed" },
                              "files"                  : { "id" : "**", "name" : "**" },
                              "packageType"            : { "id" : "maven", "name" : "Maven" },
                              "skipextract"            : { "id" : "", "name" : "" },
                              "view"                   : { "id" : "f797dfb5-a51c-46c6-9783-806a346a8aec", "name" : "Local" }
                            },
  "isRetained"          : false
}

```

### Transformed Github Action

```yaml
- name: Authenticate GitHub Packages
  uses: whelk-io/maven-settings-xml-action@v20
  repositories: '[{ \"id\": \"fix_world\", \"url\": \"https://maven.pkg.github.com/${{ env.PACKAGES_OWNER }}/${{ env.PACKAGES_REPOSITORY }}\" }]'\nservers: '[{ \"id\": \"fix_world\", \"username\":${{ env.PACKAGES_USERNAME }} \"password\": ${{ secrets.PACKAGES_TOKEN }} }]'
  # # Ensure the following is included in your pom.xml file
  # <dependencies>
  #   <dependency>
  #     <groupId>fix_world</groupId>
  #     <artifactId>fix_world</artifactId>
  #     <version>1.0.0</version>
  #   </dependency>
  # </dependencies>
- name: Install Maven packages
  run: mvn install -s ~/.m2/settings.xml
```

### Unsupported inputs

- View
