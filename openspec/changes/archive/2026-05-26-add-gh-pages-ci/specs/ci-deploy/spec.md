## ADDED Requirements

### Requirement: Automatic deployment on push to main
The CI pipeline SHALL automatically publish the contents of the `html/` directory to GitHub Pages whenever a commit is pushed to the `main` branch.

#### Scenario: Push to main triggers deploy
- **WHEN** a commit is pushed to the `main` branch
- **THEN** a GitHub Actions workflow runs that uploads `html/` as a Pages artifact and deploys it via `actions/deploy-pages`
- **AND** the deployed site reflects the new commit within minutes of a successful workflow run

#### Scenario: Push to a non-main branch does not deploy
- **WHEN** a commit is pushed to any branch other than `main`
- **THEN** the deploy workflow MUST NOT run

### Requirement: Manual re-deploy via workflow_dispatch
The workflow SHALL be re-runnable on demand from the GitHub Actions UI without requiring a new commit.

#### Scenario: Manual trigger from Actions tab
- **WHEN** a maintainer clicks "Run workflow" on the deploy workflow in the GitHub Actions UI
- **THEN** the workflow runs against the current `main` and republishes the site

### Requirement: Only the html/ directory is published
The workflow SHALL upload only the `html/` directory as the Pages artifact. Other top-level directories (e.g. `encyclic/`, `openspec/`, `.claude/`, `.github/`) MUST NOT be included in the published site.

#### Scenario: Source material is not served
- **WHEN** the workflow runs successfully
- **THEN** files under `encyclic/` and other non-`html/` directories are absent from the deployed Pages site
- **AND** requesting `<pages-url>/encyclic/...` returns 404

#### Scenario: Site root is the html index
- **WHEN** a visitor requests the Pages site root
- **THEN** the response body is the content of `html/index.html`

### Requirement: Safe concurrent deploys
The workflow SHALL use a single concurrency group for Pages deployments so that overlapping runs do not corrupt the published site.

#### Scenario: Two pushes land in quick succession
- **WHEN** two commits are pushed to `main` within seconds of each other
- **THEN** the first deploy is allowed to complete (it is not cancelled mid-flight)
- **AND** the second deploy runs after the first and publishes the later commit

### Requirement: Least-privilege workflow permissions
The workflow SHALL request only the permissions required to deploy to Pages: `contents: read`, `pages: write`, and `id-token: write`. It MUST NOT request `contents: write` or other write scopes.

#### Scenario: Permissions block in workflow
- **WHEN** the workflow file is inspected
- **THEN** its top-level `permissions:` block lists exactly `contents: read`, `pages: write`, `id-token: write`

### Requirement: One-time GitHub Pages repo configuration documented
The change SHALL include human-readable instructions for the maintainer covering the one-time GitHub Pages setup that cannot be automated from a workflow file alone.

#### Scenario: Maintainer follows the instructions
- **WHEN** the maintainer reads the instructions in `tasks.md`
- **THEN** they can locate the Settings → Pages screen, set Source to "GitHub Actions", and verify the first deploy via the URL surfaced in the workflow run summary, without further guidance
