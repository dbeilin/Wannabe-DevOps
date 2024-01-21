# Github Pagination
We recently needed to fetch a list of all of our repos and their branches for an internal tool we're working on and I wanted to document how we handled pagination.

## Prepare
Create a `.env` file in your directory with:

- Your [PAT](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens).
- Organization Name

## Usage
Let's start with the basics:
```python
import os
import requests
from dotenv import load_dotenv

load_dotenv()

github_token = os.getenv('GITHUB_TOKEN')
organization = os.getenv('GITHUB_ORGANIZATION')
repo_name = "your-repo-name"
```

We can now make our GET request:
```python
url = f"https://api.github.com/repos/{organization}/{repo_name}/branches?per_page=10"
headers = {'Authorization': f'token {github_token}',
            'Accept': "application/vnd.github+json"}
res = requests.get(url, headers=headers)
```

`res` will print out a list of repos:
```
[{'name': '39d1f185c83a44538781',
  'commit': {'sha': 'commit-sha',
   'url': 'https://api.github.com/repos/org/repo/commits/commit-sha'},
   ...
   ...
```

Next, we can see that `res.links` fetches the following answer:
```
{'next': {'url': 'https://api.github.com/repositories/repo/branches?per_page=10&page=2',
  'rel': 'next'},
 'last': {'url': 'https://api.github.com/repositories/repo/branches?per_page=10&page=9',
  'rel': 'last'}}
```

We loop over `res.links` while `next` exists and fill the `branches` list until there are no more pages:
```python
branches = []
res = requests.get(url, headers=headers)
branches.extend([branch['name'] for branch in res.json()])

while 'next' in res.links.keys():
    res = requests.get(res.links['next']['url'], headers=headers)
    branches.extend([branch['name'] for branch in res.json()])
```

That's pretty much it.

## Notes
A little after writing this document we started using GraphQL to query the repos and branches. It's a lot faster and uses less requests to Github's endpoint. I'll cover that in the future :)
