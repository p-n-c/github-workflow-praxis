# GitHub workflow praxis

Safe environment for experimenting with GitHub workflows and actions from [People and Code](https://people-and-code.com/).

## Personal Access Tokens (PATs) and Repository Secrets

### Create a PAT

Navigate to [Developer Settings](https://github.com/settings/tokens)  
Create a token or tokens to be used only for experimentation. Copy the token **at once**, you will need it for the next step!

### Create a Repository Secret

Navigate to repository settings/actions ~/settings/secrets/actions e.g. [repository secret](https://github.com/p-n-c/github-workflow-praxis/settings/secrets/actions)

## Create a workflow which bumps the project version

This action updates the version in three places:

- package.json
  `"version": "0.0.1"`
- package-lock.json
  `"version": "0.0.1"`
- index.html
  `<meta name="version" content="0.0.1" />`

### Pull requests

Check that PRs are permitted to create and approve pull requests: [~/settings/actions](https://github.com/organizations/p-n-c/settings/actions)

[ ] Allow GitHub Actions to create and approve pull requests
