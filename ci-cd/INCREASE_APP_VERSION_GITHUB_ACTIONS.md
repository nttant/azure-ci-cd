To adapt these instructions for GitHub Actions and `semantic-release` to automatically handle versioning in your CI/CD pipeline, here’s how you can achieve the same result. `semantic-release` will handle the version bumping, creating git tags, generating changelogs, and publishing the release. You do not need to manually run `npm version` in this setup.

### GitHub Actions Workflow (`.github/workflows/deploy.yml`)

```yaml
name: Deploy Next.js App to Azure

on:
  push:
    branches:
      - main # Trigger on push to the main branch

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v3

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "16.x" # Adjust Node.js version as needed

      - name: Install dependencies
        run: npm install --legacy-peer-deps

      - name: Build the project
        run: npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: next-build
          path: .next

      - name: Deploy to Azure Static Web Apps
        uses: Azure/static-web-apps-deploy@v1
        with:
          app_location: "." # Root of the Next.js project
          output_location: ".next"
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }} # Set this secret in your GitHub repository

  release:
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Checkout code
        uses: actions/checkout@v3
        with:
          fetch-depth: 0 # Full git history is required for semantic-release

      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "16.x"

      - name: Install dependencies
        run: npm ci

      - name: Run Semantic Release
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # GitHub token to push commits/tags
          NPM_TOKEN: ${{ secrets.NPM_TOKEN }}       # Token for npm package publishing (if needed)
        run: npx semantic-release
```

### Semantic Release Configuration (`.releaserc.json`)

1. Install the necessary dependencies for `semantic-release`:

   ```bash
   npm install --save-dev semantic-release @semantic-release/commit-analyzer @semantic-release/release-notes-generator @semantic-release/changelog @semantic-release/npm @semantic-release/github @semantic-release/git
   ```

2. Create a `.releaserc.json` file for `semantic-release` configuration:

   ```json
   {
     "branches": ["main"],
     "plugins": [
       "@semantic-release/commit-analyzer",
       "@semantic-release/release-notes-generator",
       "@semantic-release/changelog",
       "@semantic-release/npm",
       "@semantic-release/github",
       "@semantic-release/git"
     ]
   }
   ```

### Key Parts Explained:

1. **`semantic-release` handles versioning**:
   - You don’t need to manually increment the version in `package.json`. `semantic-release` will handle it based on your commit messages (using the [Conventional Commits](https://www.conventionalcommits.org/) format). The version will automatically be bumped and committed, and the release will be tagged.
2. **GitHub Actions tokens**:
   - `GITHUB_TOKEN` is required for creating the release on GitHub and pushing the new version tags/commits.
   - `NPM_TOKEN` is required if you want to publish to npm (you can omit it if you're not publishing to npm).
3. **Full Git history for `semantic-release`**:
   - The `fetch-depth: 0` is necessary in the `checkout` step to ensure `semantic-release` has access to the full git history for proper analysis of version bumps.

### Notes:
- Ensure the `AZURE_STATIC_WEB_APPS_API_TOKEN`, `GITHUB_TOKEN`, and `NPM_TOKEN` (if applicable) are set up as secrets in your GitHub repository.
- `semantic-release` will handle the changelog generation, versioning, and release automation entirely based on your commit messages, so make sure your commits follow Conventional Commits format (`feat:`, `fix:`, etc.).
