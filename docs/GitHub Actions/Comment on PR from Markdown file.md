# Comment on PR using Markdown
We recently implemented a "message of the day" type of concept in some of our workflows. Here's how we template a nice message in our PRs:

```yaml
name: PR Welcome Comment
on:
  pull_request:
    types: [opened]

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

Make sure to change the path `./test.md` to your markdown file.
The result:
![](images/gha-pr-comment.png)

That's it, pretty fun option to have in your environment 🫡
