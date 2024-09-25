Here’s how you can adapt the Azure Pipelines instructions to GitHub Actions for deploying a Next.js app (with TypeScript and Tailwind) to Azure, including using `semantic-release`:

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

  deploy:
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Download build artifacts
        uses: actions/download-artifact@v3
        with:
          name: next-build

      - name: Deploy to Azure Static Web Apps
        uses: Azure/static-web-apps-deploy@v1
        with:
          app_location: "." # Root of the Next.js project
          output_location: ".next"
          azure_static_web_apps_api_token: ${{ secrets.AZURE_STATIC_WEB_APPS_API_TOKEN }} # Set this secret in your GitHub repository
```

### Semantic Release Setup

1. **Install `semantic-release`:**

   Run this command to install the necessary dependencies:
   ```bash
   npm install --save-dev semantic-release @semantic-release/commit-analyzer @semantic-release/release-notes-generator @semantic-release/changelog @semantic-release/npm @semantic-release/github @semantic-release/git
   ```

2. **Add a `.releaserc.json` configuration file:**

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

   Add the `semantic-release` step after the build and before the deployment.

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

1. **Trigger on `main` branch** – The workflow triggers when code is pushed to the `main` branch.
2. **Install Node.js** – Installs Node.js (version 16.x in this case) on the runner.
3. **Install Dependencies and Build** – Installs the required packages and builds the Next.js app.
4. **Upload and Download Build Artifacts** – Uses GitHub Actions' artifact steps to store and retrieve the build output (`.next` folder).
5. **Deploy to Azure Static Web Apps** – Deploys the Next.js app to Azure Static Web Apps using a deployment token.
6. **Run Semantic Release** – Automates versioning, changelog generation, and release publication based on Conventional Commits.

### Notes:

- Set up the `AZURE_STATIC_WEB_APPS_API_TOKEN` and `GITHUB_TOKEN` as GitHub repository secrets.
- Ensure commit messages follow the [Conventional Commits](https://www.conventionalcommits.org/) standard for `semantic-release`.
