To adapt your Azure Pipelines setup for GitHub Actions with `semantic-release` for deploying a Vite React app to Azure, here’s how you can set it up:

### GitHub Actions Workflow (`.github/workflows/deploy.yml`)

```yaml
name: Deploy Vite App to Azure

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
          node-version: "16.x" # Use Node.js 16.x or change this to match your version

      - name: Install dependencies
        run: npm install --legacy-peer-deps

      - name: Build the project
        run: npm run build

      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: dist
          path: dist

  deploy:
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: dist

      - name: Deploy to Azure Static Web Apps
        uses: Azure/static-web-apps-deploy@v1
        with:
          app_location: "dist"
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }} # Set this secret in your GitHub repository
```

### Semantic Release Setup

1. **Install `semantic-release`:**
   ```bash
   npm install --save-dev semantic-release @semantic-release/commit-analyzer @semantic-release/release-notes-generator @semantic-release/changelog @semantic-release/npm @semantic-release/github @semantic-release/git
   ```

2. **Add a `.releaserc.json` configuration file for `semantic-release`:**

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

3. **Update GitHub Actions Workflow to Include Semantic Release:**

   Add the `semantic-release` step after the build is complete and before deployment.

   ```yaml
   jobs:
     build:
       runs-on: ubuntu-latest

       steps:
         # Previous steps ...

         - name: Run Semantic Release
           env:
             GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # GitHub token for creating releases
           run: npx semantic-release
   ```

### Steps Breakdown:
1. **Trigger on push to `main`** – The workflow is triggered on commits pushed to the `main` branch.
2. **Set up Node.js** – Installs Node.js 16.x (or any version specified).
3. **Install dependencies and build** – Installs packages and runs the build command.
4. **Upload and download build artifacts** – GitHub Actions' `upload-artifact` and `download-artifact` steps handle the build artifacts.
5. **Deploy to Azure Static Web Apps** – Uses `Azure/static-web-apps-deploy` GitHub action to deploy the `dist` folder to Azure Static Web Apps using a deployment token.
6. **Run Semantic Release** – Automates versioning and changelog generation based on commit messages, and pushes releases to GitHub.

### Notes:
- **Secrets**: Store the `AZURE_STATIC_WEB_APPS_API_TOKEN` and `GITHUB_TOKEN` in your GitHub repository's secrets.
- **Semantic Release**: Ensure your commit messages follow [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/).
