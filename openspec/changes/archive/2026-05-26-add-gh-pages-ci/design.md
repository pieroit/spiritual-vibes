## Context

`spiritual-vibes` is a plain static site (HTML/CSS/JS, no build) intended to be served from GitHub Pages. The serveable content lives in `html/`. There is currently no deploy automation — publishing requires a manual step, which slows iteration and risks the live site drifting from `main`.

GitHub Pages now supports two source modes: the legacy "Deploy from a branch" mode and the modern "GitHub Actions" mode. The latter is the recommended path and integrates cleanly with the official Pages actions.

## Goals / Non-Goals

**Goals:**
- Every push to `main` redeploys the contents of `html/` to GitHub Pages.
- The workflow is minimal, uses official GitHub-maintained actions, and requires no third-party tokens.
- The site root on Pages serves `html/index.html` as `/index.html`.
- Maintainer can manually re-trigger the workflow from the Actions UI.

**Non-Goals:**
- No build/bundling step, no minification, no asset pipeline.
- No preview deployments for PRs (could be added later).
- No custom domain configuration (out of scope; can be done in repo settings later).
- No tests/linting in CI (the project has none).

## Decisions

### Decision 1: Use the official `actions/deploy-pages` workflow, not branch-based publishing
- **Choice**: Use `actions/configure-pages`, `actions/upload-pages-artifact`, and `actions/deploy-pages` with `pages: write` + `id-token: write` permissions.
- **Why**: It is the GitHub-recommended path, avoids committing built artifacts to a `gh-pages` branch, and supports the modern Pages environment with deployment URL output.
- **Alternative considered**: `peaceiris/actions-gh-pages` pushing to a `gh-pages` branch. Rejected — adds a third-party dependency and clutters git history.

### Decision 2: Upload only the `html/` subdirectory, not the repo root
- **Choice**: `actions/upload-pages-artifact` with `path: ./html`.
- **Why**: `CLAUDE.md` is explicit that `html/` is the serveable folder and `encyclic/` is read-only source material that must not be served. Uploading only `html/` keeps the Pages site clean and prevents leaking source material URLs.
- **Alternative considered**: Serve the whole repo. Rejected — would expose `encyclic/`, `openspec/`, `.claude/`, etc.

### Decision 3: Trigger on push to `main` + `workflow_dispatch`, single concurrency group
- **Choice**: `on: push: branches: [main]` plus `workflow_dispatch`, with `concurrency: group: pages, cancel-in-progress: false`.
- **Why**: The concurrency group is the pattern recommended in GitHub's Pages docs — it prevents two deploys racing while still letting an in-flight deploy finish (so we don't end up with a half-published site).
- **Alternative considered**: `cancel-in-progress: true`. Rejected — risks killing a near-complete deploy and republishing a stale one.

### Decision 4: Two jobs (`build` + `deploy`), not one
- **Choice**: A `build` job uploads the artifact; a `deploy` job depends on it and calls `actions/deploy-pages`. This mirrors GitHub's published example.
- **Why**: The `deploy` job needs a dedicated `environment: github-pages` declaration to surface the deployment URL and to gate on environment protection rules; combining jobs makes that awkward.

## Risks / Trade-offs

- **Risk**: Pages source in repo settings is not switched to "GitHub Actions" → workflow runs succeed but nothing is published. **Mitigation**: tasks.md includes explicit one-time configuration steps for the maintainer, and a verification step (open the deployment URL from the Actions run summary).
- **Risk**: A future contributor adds secrets or private content into `html/`. **Mitigation**: out-of-scope for this change, but flagged in the spec — only the `html/` folder is published, so anything outside it stays unpublished by default.
- **Trade-off**: No PR preview deployments. Acceptable for a small project; can be added later without breaking this workflow.
- **Risk**: The default branch is named something other than `main`. **Mitigation**: tasks.md asks the maintainer to confirm the default branch name and adjust the workflow trigger if needed.

## Migration Plan

1. Land the workflow file on `main`.
2. Maintainer flips Settings → Pages → Source to "GitHub Actions" (one-time).
3. First run kicks off automatically on the merge commit; verify the deployment URL.
4. Rollback: revert the workflow file commit; the live site will simply stop updating (last successful deploy stays live).
