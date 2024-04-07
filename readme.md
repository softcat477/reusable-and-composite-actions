# How to use reusable workflow and composite action

## Reusable Workflow
🐟 More details: [Reusable Workflow](reusable-workflow.md)
```yaml=
# .github/workflows/driver.yml
name: Trigger reusable workflow in a public repo

on:
  pull_request:
    branches:
      - main
  workflow_dispatch:
  
env:
  env1: "An environment variable in caller workflow in ${{github.event.repository.name}}"
  
jobs:
  run-reuse-workflow:
    # https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_iduses
    # {owner}/{repo}/.github/workflows/{filename}@{ref}
    uses: softcat477/reuse-workflow/.github/workflows/reuse-workflow.yml@v1.0.0
    with:
      input1: "This-is-input-1-from-caller-workflow-in-${{github.event.repository.name}}"
    secrets:
      secret1: ${{ secrets.secret0 }}

  show-output-from-reuse-workflow:
    runs-on: ubuntu-latest
    needs: run-reuse-workflow
    steps:
      - name: Print single-line output
        run: echo '${{needs.run-reuse-workflow.outputs.reuse-workflow-out1}}'
      - name: Print multi-line output
        run: echo '${{needs.run-reuse-workflow.outputs.reuse-workflow-out2}}'
```

## Composite Action
🐟 More details: [Composite Action](composite-action.md)
```yaml=
# .github/workflows/composite-action.yml
name: Use composite action in a public repo

on: 
  pull_request:
    branches:
      - main
  workflow_dispatch:

env:
  CALLER_ENV: "An environment variable in the caller workflow."

jobs:
  use-composite-action:
    name: Use Composite Action in the same Repo
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#example-using-a-public-action
      # uses: {owner}/{repo}/{path}@{ref}
      - name: Use composite action
        id: composite-action
        uses: softcat477/composite-action/.github/composite-action@v1.0.0
        env:
          ASSIGNED_ENV: "An environment variable assigned to composite actions"
        with:
          input1: "An input assigned to composite actions"

      - name: Print the single-line from the composite action
        run: echo "The single-line output is:${{steps.composite-action.outputs.composite-action-out1}}"

      - name: Print the multi-line from the composite action
        run: echo 'The multi-line output is:${{steps.composite-action.outputs.composite-action-out2}}'
```