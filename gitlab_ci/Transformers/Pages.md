# pages

## GitLab input

```yaml
pages:
  stage: deploy
  script:
    - echo 'Nothing to do...'
  artifacts:
    paths:
      - public
    expire_in: 1 day
```

### Transformed Github Action

```yaml
- uses: JamesIves/github-pages-deploy-action@4.1.5
  with:
    branch: gh-pages
    folder: public
```

### Unsupported Options

None
