# Composite Action

* [Overview](#Overview)
* [Use a composite action](#Use-a-composite-action-in-a-public-repo)
* [Pass inputs](#Pass-inputs)
* [Pass environment variables](#Pass-environment-variables)
* [Pass secrets](#How-about-secrets?)
* [Use outputs](#Use-outputs)

🐟 How to use [Reusable Workflow](reusable-workflow.md)

## Overview
Unlike reusable workflow, which is used as a job, composite action is used in **steps** of a job.

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

---
## Use a composite action in a public repo
https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#example-using-a-public-action
```yaml=
# .github/workflows/composite-action.yml
- name: Use composite action
  uses: softcat477/composite-action/.github/composite-action@v1.0.0
```

The syntax to use a composite action is `jobs.<job_id>.steps[*].uses: {owner}/{repo}/{path}@{ref}`. In this example, I'm using [this one](https://github.com/softcat477/composite-action/tree/main/.github/composite-action):
* the `owner` is myself
* the `repo` name is `composite-action`
* the `path` is `.github/composite-action`. This should be the path to the `action.yml` file but `excluding` the file itself.
* the `ref` is the `v1.0.0` tag.


## Pass inputs
```yaml=
# .github/workflows/composite-action.yml
- name: Use composite action
  with:
    input1: "An input assigned to composite actions"
```
Use the `jobs.<job_id>.steps[*].with` syntax to pass inputs to the composite action. In this example, the name of the input is `input1`. It should match with the inputs of the composite action.

This is what's in the composite action.
```yaml=
# softcat477/composite-action/blob/main/.github/composite-action/action.yml
inputs:
  input1: # Match with this one!
    description: "The first input to composite action"
    required: true
```

Well it did match with the caller workflow (`.github/workflows/composite-action.yml`)

## Pass environment variables
Composite action **has** access to environment variables in the caller workflow. Use `#{{env.env_variable_name}}` to access the variables.

But you can still use `jobs.<job_id>.steps[*].env` to pass environment variables.

## How about secrets?
Secrets are **not going to work**, including `${{secrets.GITHUB_TOKEN}}`. Composite action can't access secrets in the caller workflow. Check `.github/workflows/composite-action.yml`. It fails.

## Use outputs
Assign and id the the step using composite action, and use `steps.<id>.outputs.<output name>` to read the output. The `<output name>` should match with what's specified in the composite action.

These are the outputs of the composite action.
```yaml=
# softcat477/composite-action/blob/main/.github/composite-action/action.yml
outputs:
  composite-action-out1:
    description: "The first output from composite action"
    value: ${{steps.step-singleline.outputs.output1}}
  composite-action-out2:
    description: "The second output from reuse-workflow"
    value: ${{steps.step-multiline.outputs.output2}}
```

so in the workflow, assign an `id` the the composite action.
```yaml=
# .github/workflows/composite-action.yml
- name: Use composite action
  id: composite-action
```

and read the output with `steps.<id>.outputs.<output name>`
```yaml=
# .github/workflows/composite-action.yml
- name: Print the single-line from the composite action
  run: echo "The single-line output is:${{steps.composite-action.outputs.composite-action-out1}}"
 - name: Print the multi-line from the composite action
   run: echo 'The multi-line output is:${{steps.composite-action.outputs.composite-action-out2}}'
```