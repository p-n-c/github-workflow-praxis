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

package-lock.json is not updated manually but programmatically e.g.

  ```json
  npm version ${{ steps.version.outputs.new_version }} --no-git-tag-version --force
  npm install
  ```

### Pull requests

Check that PRs are permitted to create and approve pull requests: [~/settings/actions](https://github.com/organizations/p-n-c/settings/actions)

[x] Allow GitHub Actions to create and approve pull requests

## Detailed breakdown of the .yaml workflow

### Triggers

#### workflow_dispatch

This allows manual triggering of this workflow in GitHub Actions.  
The optional input `version_override` lets us specify a custom version.

#### Push

A push to release-branch `release-branch` triggers the workflow.

### Jobs

The GitHub Action `create-release` wraps the GitHub Release APIso we can use leverage GitHub Actions to [create releases](https://github.com/actions/create-release).

#### Steps within the create-release job

- Checkout Code
  - Checks out the repository, ensuring the `release-branch` is used, with full commit history (`fetch-depth: 0`).
- Setup Node.js
  - Installs and configures Node.js version 18.x.
- Get Current Version
  - Retrieves the current version from `package.json`.
  - Sets it as an output `current_version` for subsequent steps.
- Set New Version
  - Determines the new version:
    - If `version_override` is provided, uses it.
    - Otherwise, auto-increments the patch number (`build`).
- Update `package.json`
  - Updates the `package.json` version and installs dependencies.
- Inject Version into HTML
  - Updates the `<meta name="version">` tag in `src/index.html` with the new version.
- Commit Changes
  - Configures Git, commits the updated files (`package.json`, `package-lock.json`, `index.html`), and pushes changes.
- Create or Update Pull Request
  - Checks if a pull request exists between `release-branch` and `main`.
  - Creates one if none exists and enables auto-merge.
- Create GitHub Release
  - Creates a GitHub release with the new version tag.
  
## .yaml syntax

Three examples of syntax drawn from the workflow.

### Example Output 1

```yaml
run: |
  CURRENT_VERSION=$(node -p "require('./package.json').version")
  echo "current_version=${CURRENT_VERSION}" >> $GITHUB_OUTPUT
```

A `run` block contains shell commands to be executed.  
`|`: Indicates a multi-line string for commands. Commands are executed in sequence.  

The command:

`CURRENT_VERSION=$(node -p "require('./package.json').version")`

Can be broken down:

- `CURRENT_VERSION`=: A shell variable is being assigned.
- `$(...)`: Subshell syntax to execute a command and use its output.
- `node -p "..."`:
  - `node`: Executes a Node.js command.
  - `-p`: Prints the result of the executed JavaScript code.
  - `"require('./package.json').version"`:
    - `require('./package.json')`: Reads and parses the package.json file as a JavaScript object.
    - `.version`: Accesses the version property from the object.
  
If `package.json` contains:

```json
{
  "name": "my-project",
  "version": "1.2.3"
}
```

Then `CURRENT_VERSION` will be assigned the value `"1.2.3"`.

### Example Output 2

```bash
echo "current_version=${CURRENT_VERSION}" >> $GITHUB_OUTPUT
```

- `echo`: Prints a string to standard output.
- `>> $GITHUB_OUTPUT`:
  - `$GITHUB_OUTPUT` is a special file used in GitHub Actions to define step outputs.
  - Appending `current_version=<value`> to this file registers the variable `current_version` as an output of this step.
- The value of `current_version` (e.g., `1.2.3`) can be referenced in subsequent steps using:
  - `${{ steps.current_version.outputs.current_version }}`

### Example Output 3

`sed -i "s/<meta name=\"version\" content=\".*\" \/>/<meta name=\"version\" content=\"${VERSION}\" \/>/" ./src/index.html`

- `sed` is a stream editor for filtering and transforming text
`-i` means edit the file in place
`s/pattern/replacement/` is sed's substitution command
  - First part `<meta name=\"version\" content=\".*\" \/>` matches the existing meta tag
  - `.*` matches any characters between `content="` and `"`
  - Second part replaces with the same tag but using `${VERSION}`
- `./src/index.html` is the file to edit