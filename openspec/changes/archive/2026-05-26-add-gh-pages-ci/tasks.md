## 1. Author the GitHub Actions workflow

- [ ] 1.1 Create `.github/workflows/` directory at the repo root.
- [ ] 1.2 Add `.github/workflows/deploy.yml` with:
  - `name: Deploy to GitHub Pages`
  - `on: push: branches: [main]` and `workflow_dispatch:`
  - Top-level `permissions:` block: `contents: read`, `pages: write`, `id-token: write` (only these).
  - `concurrency: group: pages, cancel-in-progress: false`.
  - A `build` job on `ubuntu-latest` that:
    - runs `actions/checkout@v4`
    - runs `actions/configure-pages@v5`
    - runs `actions/upload-pages-artifact@v3` with `path: ./html`
  - A `deploy` job on `ubuntu-latest` that:
    - declares `environment: name: github-pages, url: ${{ steps.deployment.outputs.page_url }}`
    - `needs: build`
    - runs `actions/deploy-pages@v4` with `id: deployment`
- [ ] 1.3 Pin each action to a major-version tag (`@v4`, `@v5`, `@v3`) and avoid floating `@main`.
- [ ] 1.4 Confirm the repo's default branch is named `main`; if not, update the `on.push.branches` value to match.

## 2. One-time GitHub Pages configuration (manual, repo owner)

These steps cannot be done from a workflow file — the repo owner must do them once in the GitHub web UI.

- [ ] 2.1 Push the workflow to `main` (or merge the PR that adds it). The first push may fail at the deploy step until step 2.2 is done — that's expected.
- [ ] 2.2 In GitHub, open the repo → **Settings** → **Pages** (left sidebar).
- [ ] 2.3 Under **Build and deployment** → **Source**, choose **GitHub Actions**. (Do NOT choose "Deploy from a branch".) No further fields need to be filled in on this screen.
- [ ] 2.4 Open the repo → **Settings** → **Actions** → **General** → **Workflow permissions**. Confirm "Read and write permissions" is NOT required — this workflow declares its own permissions, so the default ("Read repository contents and packages permissions") is sufficient and safer.
- [ ] 2.5 If the repo is private, confirm your GitHub plan allows Pages on private repos (Free tier supports public repos only; private repos need Pro/Team/Enterprise). For a public repo, no action.
- [ ] 2.6 (Optional) In **Settings** → **Environments** → `github-pages`, optionally restrict the environment to the `main` branch under "Deployment branches".

## 3. Verify the first deploy

- [ ] 3.1 Push a trivial commit to `main` (or click **Run workflow** on the Deploy workflow in the **Actions** tab) to trigger a run.
- [ ] 3.2 In the **Actions** tab, open the latest run. The `deploy` job summary shows a **page_url** like `https://<user>.github.io/<repo>/`. Open it.
- [ ] 3.3 Confirm the home page is the content of `html/index.html`.
- [ ] 3.4 Confirm `https://<user>.github.io/<repo>/encyclic/` returns 404 (source material must not be served).
- [ ] 3.5 Confirm linked CSS, images under `html/assets/`, and the game page load over HTTPS without mixed-content warnings.

## 4. Document for future contributors

- [ ] 4.1 Add a short "Deployment" section to `CLAUDE.md` (or a `README.md` if/when one exists) noting: "Pushing to `main` auto-deploys `html/` to GitHub Pages via `.github/workflows/deploy.yml`. To re-deploy without a commit, use the Actions tab → Deploy to GitHub Pages → Run workflow."
