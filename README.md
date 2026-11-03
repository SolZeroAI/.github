# .github

Shared GitHub workflows for SolZeroAI repositories.

## Reusable workflows

| Workflow | Use |
| --- | --- |
| `tegami-pr-preview.yml` | Builds the Tegami release preview for a pull request and uploads it as the `tegami-pr-preview` artifact. |
| `tegami-pr-comment.yml` | Downloads that artifact and posts it as a pull request comment. |

Both take two inputs:

- `runs-on`: the runner labels as JSON. The default is `"ubuntu-latest"`. A private repository on the self-hosted runners passes `'["self-hosted","arm64","linux"]'`.
- `setup-commands`: shell commands that run before `nub install`. Leave it empty when the repository needs none.

### Caller for the preview

```yaml
name: Tegami PR Preview

on:
  pull_request:
    branches: [master]
    types: [opened, synchronize, reopened, ready_for_review]

permissions:
  contents: read

concurrency:
  group: tegami-pr-preview-${{ github.event.pull_request.number }}
  cancel-in-progress: true

jobs:
  preview:
    if: github.event.pull_request.draft == false
    uses: SolZeroAI/.github/.github/workflows/tegami-pr-preview.yml@master
    with:
      runs-on: '"ubuntu-latest"'
```

### Caller for the comment

Keep the workflow name `Tegami PR Preview` in the preview caller. The comment caller watches it by name.

```yaml
name: Tegami PR Comment

on:
  workflow_run:
    workflows: [Tegami PR Preview]
    types: [completed]

permissions:
  actions: read
  contents: read
  pull-requests: write

jobs:
  comment:
    uses: SolZeroAI/.github/.github/workflows/tegami-pr-comment.yml@master
    with:
      runs-on: '"ubuntu-latest"'
```

## Rules

- This repository is public. Do not add secrets, internal hostnames, or process notes.
- A change on `master` reaches every caller on its next run. Keep inputs backward compatible.
- Use major tags for actions. Do not pin an action to a commit SHA.
