# flows

Shared GitHub Actions workflows for New Graph Environment's R packages. A package's
workflow file shrinks to a call, so a runner image, a timeout or a step changes here once
instead of in every repository.

| workflow | does | inputs (default) |
|---|---|---|
| `r-cmd-check.yaml` | `R CMD check`, then prints every skipped-test block | `runs-on` (`ubuntu-24.04`), `timeout-minutes` (45), `error-on` (`'"warning"'`), `extra-packages` |
| `r-pkgdown.yaml` | builds the pkgdown site without `CLAUDE.md`, fails on an undeclared root page, deploys to `gh-pages` | `runs-on`, `timeout-minutes` (90), `allowed-pages` (`404 authors index LICENSE LICENSE-text`), `deploy` (`true`), `extra-packages` |
| `r-citation-cff.yaml` | regenerates `CITATION.cff` from `DESCRIPTION` and pushes it to `main`, serialized and rebased so a release's quick pushes cannot leave it stale | `runs-on`, `timeout-minutes` (15) |

## Calling one

Each workflow's header carries the full caller file. The shortest form:

```yaml
name: Update CITATION.cff
on:
  push:
    branches: [main]
    paths: ['DESCRIPTION']
  workflow_dispatch:
permissions:
  contents: write
jobs:
  update-citation:
    uses: NewGraphEnvironment/flows/.github/workflows/r-citation-cff.yaml@v1
```

Inputs go under `with:`:

```yaml
jobs:
  pkgdown:
    permissions:
      contents: write
    uses: NewGraphEnvironment/flows/.github/workflows/r-pkgdown.yaml@v1
    with:
      allowed-pages: 404 authors index LICENSE LICENSE-text NOTICE
```

The called job runs with the caller job's token, so the caller grants what the job needs:
`contents: write` for a pkgdown deploy and for the citation push.

A package that needs what the inputs cannot express (a matrix, an extra check step) keeps
its own workflow and says why in a comment beside the difference.

## Versions

Callers pin `@v1`, a tag that moves forward with every compatible change. A change that
would break a caller (a renamed or removed input, a new required permission) goes out as
`v2`, and callers move to it deliberately.

To release a compatible change, merge it to `main`, then move the tag:

```bash
git tag -f v1 && git push -f origin v1
```

## Runner image

The workflows name their image (`ubuntu-24.04`) instead of `ubuntu-latest`, which moved to
Ubuntu 26.04 in late 2026. To move every package to a new image, change the `runs-on`
default in the three workflows and move `v1`. Before doing so, run one package's workflow on
the new image with no cache and read its log: packages should install as binaries
(`Got <pkg>`), not compile (`Building <pkg>`).

## Reading a timed-out run

A job killed by `timeout-minutes` ends `cancelled`, not `failure`, and `--log-failed` is
empty. The only marker is the check run's annotation, "The job has exceeded the maximum
execution time of …":

```bash
gh api repos/<owner>/<repo>/check-runs/<job id>/annotations --jq '.[].message'
```

## License

MIT
