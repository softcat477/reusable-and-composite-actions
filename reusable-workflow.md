# Reusable Workflow

🐟 How to use [Composite Action](composite-action.md)
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
---
### Use a public repo's reusable workflow in a job
https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_iduses

Put the location of the workflow in `jobs.<job id>.uses`. The syntax is `{owner}/{repo}/.github/workflows/{filename}@{ref}`

I created a tag v1.0.0 in the reusable workflow and put it in the `{ref}` part.
    
```yaml=
jobs:
  run-reuse-workflow:
    uses: softcat477/reuse-workflow/.github/workflows/reuse-workflow.yml@v1.0.0
```

### Pass input to the workflow
Pass inputs under `jobs.<job id>.with`.
```yaml=
jobs:
  run-reuse-workflow:
    with:
      input1: "This-is-input-1-from-caller-workflow-in-${{github.event.repository.name}}"
```
* The name of the input (`input1`) is defined in the reusable workflow.

### Pass secret to the workflow
Pass secrets under `jobs.<job id>.secrets`.
```yaml=
jobs:
  run-reuse-workflow:
    secrets:
      secret1: ${{ secrets.secret0 }}
```
* The name of the secret (`secret1`) is defined in the reusable workflow. 

* The called workflow does **not** have access to the secrets in the caller workflow. It's not going to work if the called workflow uses a secret and the caller workflow does not pass the secret.

This is quoted from https://docs.github.com/en/actions/using-workflows/reusing-workflows#overview
> When a reusable workflow is triggered by a caller workflow, the github context is always associated with the caller workflow. The called workflow is automatically granted access to github.token and secrets.GITHUB_TOKEN. For more information about the github context, see "Contexts."

* There's an exception for `${{secrets.GITHUB_TOKEN}}`. The called workflow **can** use `${{secrets.GITHUB_TOKEN}}` without a problem.

### Use the output of the workflow
```yaml=
jobs:
  run-reuse-workflow:
    ...
  show-output-from-reuse-workflow:
    ...
    needs: run-reuse-workflow
    steps:
      - name: Print single-line output
        run: echo '${{needs.run-reuse-workflow.outputs.reuse-workflow-out1}}'
      - name: Print multi-line output
        run: echo '${{needs.run-reuse-workflow.outputs.reuse-workflow-out2}}'
```

* Use `jobs.<job id>.needs` to ask GitHub action to run this job **after** the reuse workflow is done. **By default, jobs run parallel**, so we need `needs` to control the execution flow.

* The name of the outputs is defined in the reusable workflow. In this example, the outputs are named `reuse-workflow-out1` and `reuse-workflow-out2`. 

* Use `needs.<job id>.outputs.<output name>` to use the output.
