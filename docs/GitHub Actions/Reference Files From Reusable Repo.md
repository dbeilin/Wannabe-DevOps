# Use Files From Another Repo
## Background
While I was trying to implement a [workflow that templates an automatic PR comment](Comment%20on%20PR%20from%20Markdown%20file.md) from a Markdown file, I stumbled upon an issue where I couldn't call my `.md` file which ewas supposed to sit in anothr repo.
We have a dedicated repository that holds all of our reusable Github Workflows, which we then reference in each of our services as needed.
If I simply put my Markdown file in the same repo as our reusable workflows, the file would not be found when some repo calls this workflow.

## Example
The following would only work if `test.md` was located in the caller repo:
```yaml
name: PR Welcome Comment
on:
  workflow_call:

jobs:
  comment:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout repository
      uses: actions/checkout@v4

    - name: Read .md file and set output
      shell: bash
      run: |
        echo 'README<<EOF' >> $GITHUB_ENV
        cat ./test.md >> $GITHUB_ENV
        echo 'EOF' >> $GITHUB_ENV

    - name: Comment on PR
      uses: peter-evans/create-or-update-comment@v2
      with:
        issue-number: ${{ github.event.pull_request.number }}
        body: ${{ env.README }}
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## Workaround
In the reusable workflow, we can create 2 files:

- Composite
- Github Action

Create the composite workflow using the same logic:
```yaml
name: pr-comment
description: pr-comment

runs:
  using: "composite"
  steps:
    - name: Get markdown file
      shell: bash
      run: |
        echo 'README<<EOF' >> $GITHUB_ENV
        cat ${{ github.action_path }}/test.md >> $GITHUB_ENV
        echo 'EOF' >> $GITHUB_ENV
```

And then call it like so in the reusable workflow:
```yaml
name: PR Welcome Comment
on:
  workflow_call:

jobs:
  comment:
    runs-on: ubuntu-latest
    steps:
    - name: Fetch Markdown
      uses: dbeilin/PrimateCI/composite/common/pr-comment@main

    - name: Comment on PR
      uses: peter-evans/create-or-update-comment@v2
      with:
        issue-number: ${{ github.event.pull_request.number }}
        body: ${{ env.README }}
```

!!! note
    Both files should sit inside the "reusable" repo.

### Calling the Workflow
You can now call the workflow like so in any of your repos:
```yaml
name: common-trigger

on:
  pull_request:
    branches: [main]
    types: [opened, edited, synchronize, reopened]

jobs:
  pr-comment:
    name: pr-comment
    uses: dbeilin/PrimateCI/.github/workflows/pr.yml@main
    if: ${{ github.event_name == 'pull_request' && github.event.action == 'opened' }}
```

This way, you don't have to keep a `.md` file in your regular repos and it can sit along your reusable workflows.
