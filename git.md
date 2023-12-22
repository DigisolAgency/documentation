# Work with git

### Branches

- `master/main` - Production Server
- `dev` - Staging Server (dev.digisol.agency)

### Rules

- **Do not push directly into master/main or dev. Only via PR.**
- **Every PR should include a description of the changes you want to merge
into the branch.**
- **Every PR should be verified and approved.**

If the changes are anticipated to break other functionalities within the
application, the reviewer must utilize the "request changes" option to
signal the developer that the new changes will have an adverse effect on
existing functionalities, and as a result, the PR cannot be merged.

### GitHub Actions

Pull requests in the master/main or dev branches trigger GitHub actions to
push changes on the server and build the application. The GitHub Actions
configurations are located in ./github/workflows.

Additionally, it's crucial to adhere to these guidelines to maintain a smooth
and efficient development process. Clear communication through detailed PR
descriptions and through verification processes helps ensure the stability
of our production and staging environments.
