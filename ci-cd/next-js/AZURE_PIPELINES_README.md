Here’s a CI/CD pipeline script for deploying a Next.js app (created with TypeScript and Tailwind using `npx create-next-app`) to Azure. This example assumes you’re deploying to Azure Static Web Apps or Azure App Service.

### Azure Pipelines YAML (`azure-pipelines.yml`)

```yaml
# Azure Pipelines configuration file for Next.js with TypeScript and Tailwind
trigger:
  branches:
    include:
      - main # Trigger pipeline on changes to main branch

pool:
  vmImage: "ubuntu-latest" # Use the latest Ubuntu VM

steps:
  - task: NodeTool@0
    inputs:
      versionSpec: "16.x" # Ensure Node.js 16.x is installed (adjust if needed)
    displayName: "Install Node.js"

  - script: |
      npm install --legacy-peer-deps
      npm run build
    displayName: "Install dependencies and build"

  - task: CopyFiles@2
    inputs:
      SourceFolder: ".next" # Build output directory for Next.js is '.next'
      Contents: "**"
      TargetFolder: "$(Build.ArtifactStagingDirectory)/.next"
    displayName: "Copy build artifacts"

  - task: PublishBuildArtifacts@1
    inputs:
      PathtoPublish: "$(Build.ArtifactStagingDirectory)/.next"
      ArtifactName: "drop"
    displayName: "Publish build artifacts"

  - task: AzureStaticWebApp@0 # If deploying to Azure Static Web Apps
    inputs:
      app_location: "." # Root of the Next.js project
      output_location: ".next" # Path to the build output
      azure_static_web_apps_api_token: $(deployment_token) # Azure Static Web Apps deployment token
    displayName: "Deploy to Azure Static Web Apps"
```

### Steps Breakdown:

1. **Trigger on the `main` branch** – The pipeline triggers on any commits to the `main` branch.
2. **Use Node.js** – Ensures Node.js 16.x is installed (or modify based on your Node version).
3. **Install Dependencies and Build** – Installs the required packages (including TypeScript and Tailwind) and runs `npm run build` to build the Next.js app. The build output is stored in the `.next` folder.
4. **Copy and Publish Artifacts** – Copies the `.next` build folder and publishes it as build artifacts.
5. **Deploy to Azure Static Web Apps** – Deploys the Next.js app using the deployment token. Adjust the app and output locations as necessary if you’re deploying to Azure Static Web Apps.

### Notes:

- Replace `$(deployment_token)` with your Azure Static Web Apps deployment token or set up an Azure service connection for App Service deployment.
- If you are using Azure App Service, replace the last task with the `AzureWebApp` task:

```yaml
- task: AzureWebApp@1
  inputs:
    azureSubscription: "<azure-service-connection>"
    appName: "<your-app-name>"
    package: "$(Build.ArtifactStagingDirectory)/.next"
```
