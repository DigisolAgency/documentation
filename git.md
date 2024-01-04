# Work with git

### Branches

- `master/main` - Production Server
- `dev` - Staging Server

### Rules

- **Do not push directly into master/main or dev. Only via PR.**
- **Every PR should include a description of the changes you want to merge
into the branch.**
- **Every PR should be verified and approved.**

If the changes are anticipated to break other functionalities within the
application, the reviewer must utilize the "request changes" option to
signal the developer that the new changes will have an adverse effect on
existing functionalities, and as a result, the PR cannot be merged.

### Commits

Use [conventional commits guide](https://www.conventionalcommits.org/en/v1.0.0/) 
to properly name your changes. Based on commits names semantic versioning will be
calculating.

New feature commit example:
```
feat: add new feature
```
This commit will trigger changing version, for example, from 0.1.0 to 0.2.0

Patch or fix:
```
fix: change page title
```
This commit will trigger changing version, for example, from 0.2.0 to 0.2.1

New major version:
```
release: New application major version
```
This commit will trigger changing version, for example, from 0.2.0 to 1.0.0.

All other conventional commits prefixes will not affect on versioning change.

**Note:** In large scale applications prefixes can have `scope` sufix like this:
```
feat(auth): add cookie based authorisation
fix(api): update oathkeeper configuration files.
```
Or:
```
feat: (auth) add cookie based authorisation
fix: (api) update oathkeeper configuration files.
```
Depends on project guidelines.

#### Commits description

Instead of long commit title, you can use commit description:
```bash
git commit -m "Title" -m "Description ..........";
```

### GitHub Actions

Pull requests in the master/main or dev branches trigger GitHub actions to
push changes on the server and build the application. The GitHub Actions
configurations are located in ./github/workflows.

Additionally, it's crucial to adhere to these guidelines to maintain a smooth
and efficient development process. Clear communication through detailed PR
descriptions and through verification processes helps ensure the stability
of our production and staging environments.
