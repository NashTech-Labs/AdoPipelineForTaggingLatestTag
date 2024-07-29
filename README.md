# Azure DevOps Pipeline for Building and Pushing Docker Images

This repository contains an Azure DevOps pipeline configuration for building a Docker image, tagging it with the build number and `latest`, and pushing it to an Azure Container Registry (ACR).

## Pipeline Configuration

### Trigger

The pipeline is configured to trigger on changes to the `main` branch. You can change the branch if needed.

```yaml
trigger:
- main  # or the branch you want to trigger the build on
```

### Variables

Define the following variables in the pipeline:

- `imageName`: The name of your Docker image.
- `acrName`: The name of your Azure Container Registry.
- `buildId`: The unique ID for the build, provided by Azure DevOps.

```yaml
variables:
  imageName: your-image-name  # Replace with your image name
  acrName: your-acr-name  # Replace with your ACR name
  buildId: $(Build.BuildId)
```

### Stages and Jobs

The pipeline consists of a single stage `BuildAndPush` with one job `Build`.

#### Build Job

1. **Install Docker**

   This step installs Docker on the build agent.

   ```yaml
   - task: DockerInstaller@0
     displayName: Install Docker
   ```

2. **Login to ACR**

   This step logs in to the Azure Container Registry using the Azure CLI.

   ```yaml
   - task: AzureCLI@2
     inputs:
       azureSubscription: 'YourServiceConnection'  # Replace with your service connection name
       scriptType: 'bash'
       scriptLocation: 'inlineScript'
       inlineScript: |
         az acr login --name $(acrName)
   ```

3. **Build Docker Image**

   This step builds the Docker image and tags it with both the build number and `latest`.

   ```yaml
   - script: |
       docker build -t $(acrName).azurecr.io/$(imageName):$(buildId) .
       docker tag $(acrName).azurecr.io/$(imageName):$(buildId) $(acrName).azurecr.io/$(imageName):latest
     displayName: Build Docker Image
   ```

4. **Push Docker Image**

   This step pushes both tags of the Docker image to the Azure Container Registry.

   ```yaml
   - script: |
       docker push $(acrName).azurecr.io/$(imageName):$(buildId)
       docker push $(acrName).azurecr.io/$(imageName):latest
     displayName: Push Docker Image
   ```

## Usage

1. **Set up Azure DevOps Service Connection**

   - Go to your Azure DevOps project settings.
   - Under "Pipelines", click on "Service connections".
   - Click on "New service connection", choose "Docker Registry", and select "Azure Container Registry".
   - Name the service connection (e.g., `YourServiceConnection`).

2. **Update the Pipeline Configuration**

   Replace the placeholder values in the pipeline YAML file with your actual values:

   - `imageName`: The name of your Docker image.
   - `acrName`: The name of your Azure Container Registry.
   - `azureSubscription`: The name of your Azure DevOps service connection.

3. **Commit and Push**

   Commit the pipeline YAML file to your repository and push the changes to the branch specified in the trigger.

4. **Run the Pipeline**

   Go to Azure DevOps, navigate to Pipelines, and run the pipeline. The Docker image will be built, tagged, and pushed to your ACR.

## Example

Here's an example of the complete pipeline configuration:

```yaml
trigger:
- main  # or the branch you want to trigger the build on

variables:
  imageName: my-app  # Replace with your image name
  acrName: myacr  # Replace with your ACR name
  buildId: $(Build.BuildId)

stages:
- stage: BuildAndPush
  displayName: Build and Push Docker Image
  jobs:
  - job: Build
    displayName: Build
    pool:
      vmImage: 'ubuntu-latest'
    steps:
    - task: DockerInstaller@0
      displayName: Install Docker

    - task: AzureCLI@2
      inputs:
        azureSubscription: 'YourServiceConnection'  # Replace with your service connection name
        scriptType: 'bash'
        scriptLocation: 'inlineScript'
        inlineScript: |
          az acr login --name $(acrName)

    - script: |
        docker build -t $(acrName).azurecr.io/$(imageName):$(buildId) .
        docker tag $(acrName).azurecr.io/$(imageName):$(buildId) $(acrName).azurecr.io/$(imageName):latest
      displayName: Build Docker Image

    - script: |
        docker push $(acrName).azurecr.io/$(imageName):$(buildId)
        docker push $(acrName).azurecr.io/$(imageName):latest
      displayName: Push Docker Image
```

By following these instructions, you will set up an Azure DevOps pipeline that builds and pushes your Docker images to ACR automatically.
