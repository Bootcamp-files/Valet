# Ansible Playbook

## Designer pipeline

This plugin is not supported in Designer pipelines.

## Jenkinsfile pipeline

### Jenkins input

```groovy
steps {
  steps {
        ansiblePlaybook(credentialsId: 'private_key', inventory: 'inventories/a/hosts', playbook: 'my_playbook.yml')
    }
  }
```

### Transformed Github Action

```yaml
- name: run ansible playbook
  shell: bash
  run: ansible-playbook playbook my_playbook.yml -i inventories/a/hosts
```

### Unsupported Options

- Ensure the `ansible-playbook` ssh credentials are present on the runner.
