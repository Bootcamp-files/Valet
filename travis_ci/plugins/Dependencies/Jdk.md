# JDK

## Travis input

```yaml
jdk: 2.1
```

## Transformed Github Action

```yaml
- uses: actions/setup-dotnet@v1
  with:
    dotnet-version: '3.1.x'
```

## Supported Distributions

- Zulu OpenJDK
- Adopt OpenJDK

## Supported Java Versions (from actions/setup-java)

- major versions: 8, 11, 15
- more specific versions: 11.0, 11.0.4, 8.0.232, 8.0.282+8
- early access (EA) versions: 15-ea, 15.0.0-ea, 15.0.0-ea.2, 15.0.0+2-ea
