# labeler

A required workflow that labels pull requests by the files that they change, with
[`actions/labeler`](https://github.com/actions/labeler). The workflow lives in this repository
only. Each repository that wants labels adds one config file.

## Use it in a repository

1. Make the labels in the repository by hand. The workflow cannot create a label.
2. Add `.github/labeler.yml` to the default branch, in the
   [`actions/labeler` config format](https://github.com/actions/labeler#create-githublabeleryml).
   For example:

   ```yaml
   "plugin: gh":
     - changed-files:
         - any-glob-to-any-file: "plugins/gh/**"
   ```

The config is read from the base branch, so a new or changed rule applies to pull requests
opened after it merges. A repository with no `.github/labeler.yml` passes the check and gets
no labels.

With `sync-labels: true`, a label that the config names is removed when the diff no longer
matches it. A label that the config does not name is never touched.

## Deployment

An enterprise ruleset requires `.github/workflows/pr-labeler.yml` at `refs/heads/main` on the
default branch of every repository in every organization of the `bje` enterprise. It is the
same deployment as
[`bje-actions/conflict-label`](https://github.com/bje-actions/conflict-label) (see its ADR 0001).

- The ruleset is [`rulesets/enterprise-labeler.json`](rulesets/enterprise-labeler.json). It
  was created by hand with the API, and is not managed in terraform:

  ```bash
  gh api -X POST enterprises/bje/rulesets --input rulesets/enterprise-labeler.json
  ```

- This repository is public. A required workflow must live in a repository at least as
  visible as every repository it runs in, and the enterprise has public repositories.
- A ruleset workflow ignores the file's own event filters, so it runs on every pull request.
  With no config, it exits in a few seconds on `ubuntu-slim`.
- A change merged to `main` here applies to the next pull request in every repository. A
  defect here can block merges everywhere.
- A pull request that was open before the ruleset existed gets the check after its next push.
