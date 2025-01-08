# CI/CD Pipeline with Docker and Helm on Azure DevOps

### This README explains how to set up a CI/CD pipeline in Azure DevOps to:

- Build a Docker image from your application source.
- Push that image to an Azure Container Registry (ACR).
- Deploy it to an Azure Kubernetes Service (AKS) cluster using Helm.

# Prerequisites

### Azure DevOps Project

An existing project in Azure DevOps (or create a new one).

### Azure Subscription

- An Azure Container Registry (ACR) already provisioned (e.g., myacr.azurecr.io).
- An Azure Kubernetes Service (AKS) cluster already created.

### Service Connections in Azure DevOps

- An Azure Resource Manager service connection (with permission to deploy to AKS).
- A Docker Registry service connection (pointing to your ACR).

### Repository with the Following
- A Dockerfile (for building your container image).
- A Helm chart (for deploying your application to AKS, e.g., in a helm/ directory).

## Step-by-Step Guide

1. Create or Import Your Application Repository
- You can use Azure DevOps Repos or GitHub.
- Ensure that your source code, Dockerfile, and Helm chart are committed to this repository.

2. Add a Dockerfile
- Place a Dockerfile at the root of your project (or wherever is convenient).

3. Add a Helm Chart
- Create a folder named helm in your repo.
- Provide the minimum Helm files:

4. Create Azure DevOps Pipeline YAML
- Create a YAML file (e.g., .ado-pipelines/ci-cd-pipeline.yaml) in your repo:

### Important

- Replace <YOUR_ACR_SERVICE_CONNECTION> with the actual name of your Docker Registry service connection.
- Replace <YOUR_AZURE_SUBSCRIPTION_SERVICE_CONNECTION>, <YOUR_AKS_RESOURCE_GROUP>, and <YOUR_AKS_CLUSTER_NAME> with the appropriate values.

# Configuring Azure DevOps Service Connections
1. Docker Registry (ACR) Service Connection
- In Azure DevOps, go to Project Settings > Service Connections.
- Click New service connection > Docker Registry.
- Provide your ACR details (e.g., myacr.azurecr.io) and authentication (Service Principal or admin user).
2. Azure Resource Manager (ARM) Service Connection
- In Azure DevOps, go to Project Settings > Service Connections.
- Click New service connection > Azure Resource Manager.
- Grant the connection the necessary permissions to manage/deploy to your AKS cluster.

# How the Pipeline Works
1. Trigger: On each commit or PR merge to the main branch (or whichever branch you configure), the pipeline starts.
2. Build Stage:
- Docker@2 task uses buildAndPush to build your image from the Dockerfile and push it to your ACR.
- The image is tagged with $(Build.BuildId), ensuring uniqueness per build.
3. Deploy Stage:
- AzureCLI@2 (optional): Logs in to ACR if needed.
- HelmDeploy@0: Connects to AKS via your ARM service connection and runs helm upgrade --install on the chart:
- The image.repository and image.tag are dynamically overridden to match the newly built image in ACR.

# Best Practices & Tips
1.Tagging Strategy: Use build IDs, Git commit SHAs, or semantic versions for your Docker images.
2. Environment Separation: Use multiple stages or separate pipelines for dev, staging, and prod.
3. Secrets: Store sensitive data (e.g., database passwords) in Azure Key Vault or Kubernetes Secrets, not in plain text.
4. Helm Values: Use separate values.*.yaml files for different environments if needed, and override them in your pipeline.
5. Scalability: Adjust replica counts or advanced settings in your Helm chart for production-scale deployments.


# Summary
By following this guide:

- Build: On code changes, a new Docker image is built and pushed to your ACR.
- Deploy: A Helm release is updated in your AKS cluster with the new image tag.

This setup provides a reliable CI/CD workflow, ensuring your application is always up-to-date in Kubernetes.

# Questions or Issues?

If you run into any issues:

- Check the Azure DevOps pipeline logs for build or deployment errors.
- Ensure your service connections are configured correctly and have the right permissions.
- Verify your Helm chart paths and overrides are correct.
