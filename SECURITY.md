# Security

## The rule

`pr-labeler.yml` never checks out or runs pull request content. The guard step reads one
file from the base branch through the API. `actions/labeler` reads the config and the list
of changed files through the API, and sets labels.

That is why `pull_request_target` is safe here. On `pull_request`, the token is read-only
for a pull request from a fork, so the label write fails, and the required check fails with
it. `pull_request_target` runs in the base repository with a writable token. That is
dangerous only when a workflow builds or runs the pull request's code. This one does not.

A change to the workflow must keep these properties:

- No `actions/checkout` of the pull request head.
- No build, test or script run of pull request code.
- No untrusted input (branch names, titles, bodies, labels) in a shell.

## Token permissions

The job asks for `contents: read` and `pull-requests: write`, and nothing else. It does not
ask for `issues: write`, so it cannot create a label. Each repository makes its labels by
hand.

`actions/labeler` is pinned to a commit SHA. Dependabot proposes each new release.
