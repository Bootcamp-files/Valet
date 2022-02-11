# DockerHub task

## DockerHub input

```json
  {
    "sourceId"            : "71f5a46f-09b2-4647-b72f-0c2324a46de1:Docker",
    "type"                : "DockerHub",
    "alias"               : "_Docker",
    "definitionReference" :
                              {
                                "connection"         : { "id" : "71f5a46f-09b2-4647-b72f-0c2324a46de1", "name" : "dummy_docker_hub" },
                                "defaultVersionType" : { "id" : "latestType", "name" : "Latest" },
                                "definition"         : { "id" : "Docker", "name" : "Docker" },
                                "namespaces"         : { "id" : "this_namespace", "name" : "this_namespace" }
                              },
    "isRetained" : false
  }

```

### Transformed Github Action

```yaml
- name: Login to Docker Hub
  uses: docker/login-action@v1
  with:
    username: "${{ secrets.DOCKER_USERNAME }}"
    password: "${{ secrets.DOCKER_PASSWORD }}"
    logout: true
    registry: this_namespace
- name: Run docker pull image
  run: docker pull this_namespace/Docker:latest
```

### Unsupported inputs

- None
