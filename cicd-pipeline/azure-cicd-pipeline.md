# Building a CI/CD pipeline to Deploy a Static Website to Azure

To connect our pipeline up we'll use [GitHub Actions](https://docs.github.com/en/actions/get-started/understanding-github-actions), a continuous integration and continuous delivery (CI/CD) platform that allows you to automate your build, test, and deployment pipeline. 

You can create workflows that build and test every pull request to your repository, or deploy merged pull requests to production.

### Prepare Your Environment

- Create a GitHub repository
- Ensure your static website is already hosted in an Azure Storage account.
- Your code should also be in your GitHub repository.

### Get Your Deployment Token

1. Go to your Azure Static Website in the Azure Portal.
1. Navigate to `Settings` > `Deployment Token`.
1. Copy the token and save it securely.

### Add the Token to GitHub Secrets

- In your GitHub repository, go to `Settings` > `Secrets and variables` > `Actions`
- Click New repository secret.
- Name it `AZURE_STORAGE_KEY`.
- Paste the token you copied from Azure.
- Create a second secret called `AZURE_STORAGE_ACCOUNT`, in there store the name of your storage account as a plain text string.

### Create GitHub Actions Workflow

In your GitHub repo, create a directory called `.github`, in there one called `workflows`, and in there a file called `deploy.yml` with the following content:

```yaml
name: Deploy to Azure Static Website

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Upload files to Azure Storage
        uses: azure/CLI@v1
        with:
          inlineScript: |
            az storage blob upload-batch --account-name ${{ secrets.AZURE_STORAGE_ACCOUNT }} \
            --account-key ${{ secrets.AZURE_STORAGE_KEY }} \
            --destination '$web' \
            --source '.' \
            --overwrite
```

Breaking it down...

```yaml
name: Deploy to Azure Static Website
```
This sets the name of the workflow as it will appear in the GitHub Actions UI.

```yaml
on:
  push:
    branches:
      - main
```
This defines the trigger for the workflow. In this case it runs every time a push is made to the `main` branch.

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
```
Defines a job named deploy, which runs on the latest Ubuntu runner (a small VM) provided by GitHub.

#### Steps in the Job
```yaml
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
```
This step checks out the code from your GitHub repository so that the workflow can access the files

```yaml
      - name: Upload files to Azure Storage
        uses: azure/CLI@v1
        with:
          inlineScript: |
            az storage blob upload-batch --account-name ${{ secrets.AZURE_STORAGE_ACCOUNT }} \
            --account-key ${{ secrets.AZURE_STORAGE_KEY }} \
            --destination '$web' \
            --source '.' \
            --overwrite
```
Uses the official Azure CLI GitHub Action to run Azure CLI commands. `inlineScript` runs a `Bash` command to upload files to Azure Blob Storage.

```bash
az storage blob upload-batch --account-name ${{ secrets.AZURE_STORAGE_ACCOUNT }} \
            --account-key ${{ secrets.AZURE_STORAGE_KEY }} \
            --destination '$web' \
            --source '.' \
            --overwrite
```
This command uploads multiple files from a local directory to an Azure Blob Storage container, the parameters provided are as follows:

|Parameter|Explanation|
|---|---|
|`--account-name ${{ secrets.AZURE_STORAGE_ACCOUNT }}`|Uses a GitHub secret to specify the name of your Azure Storage account.|
|`--account-key ${{ secrets.AZURE_STORAGE_KEY }}`|Uses a GitHub secret to authenticate using the storage account key.|
|`--destination '$web'`|Targets the special `$web` container used for static website hosting in Azure Blob Storage.|
|`--source '.'`|Uploads all files from the root of the repository (current directory).|
|`--overwrite`|Ensures that existing files in the destination are replaced with the new ones.|

#### Required GitHub Secrets

|Secret Name|Description|
|---|---|
|AZURE_STORAGE_ACCOUNT|Your Azure Storage Account name|
|AZURE_STORAGE_KEY|The access key for your storage account|

### Summary

This workflow:

- Runs on every push to the main branch.
- Uploads your static website files to Azure Blob Storage.
- Uses the Azure CLI to automate deployment to the `$web` container.