## Why

The site is currently a static set of HTML/CSS/JS files with no deployment automation. Each push to the repo should immediately publish the latest version of `html/` to GitHub Pages so contributors can see changes live without manual steps. This removes friction and prevents stale/forgotten deploys.

## What Changes

- Add a GitHub Actions workflow that triggers on every push to the default branch (`main`).
- The workflow uploads the `html/` directory as a Pages artifact and deploys it via the official `actions/deploy-pages` action.
- No build step is required (project is plain static files per `CLAUDE.md`).
- Provide written instructions to the maintainer for the one-time GitHub Pages repo configuration (Settings → Pages → Source: "GitHub Actions").
- Allow manual re-runs via `workflow_dispatch`.

## Capabilities

### New Capabilities
- `ci-deploy`: Automated GitHub Actions pipeline that publishes the static site in `html/` to GitHub Pages on every push to `main`.

### Modified Capabilities
<!-- None - first capability for this project -->

## Impact

- New file: `.github/workflows/deploy.yml`.
- New permissions required on the workflow: `pages: write`, `id-token: write`, `contents: read`.
- One-time manual step required by the repo owner in GitHub Settings → Pages.
- No changes to the website's source files, no new runtime dependencies, no impact on `file://` local previews.
