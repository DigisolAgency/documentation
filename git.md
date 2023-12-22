# Work with git

### Branches 

- `master/main`   - production server
- `dev`           - staging server ( dev.digisol.agency )

### Rules

Do not push directly into master/main or dev. Only via PR.

Every PR should contain description of changes you want to include in to the branch.

Every PR should be verified and approved. 

If changes will break other functionality in application, reviewer should use "request changes" to signal developer that new changes will break other functionality 
and this PR can't be merged.



### Github actions

PR in master/main or dev branches trigger github action to push changes on server, build application.

Github actions configs located in `./github/workflows`
