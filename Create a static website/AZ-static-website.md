# Deploy a Static Website

### Create an Azure Storage Account
1. Go to the Azure Portal. Click `Create a resource > Storage > Storage account`.
2. Configure the required settings:
- Choose the Subscription and Resource Group.
- Enter a unique Storage Account name.
- Select a Region and set Performance to Standard.
- Choose `Azure Blob Storage...` for the Primary Service.
3. Select `LRS` for Redundancy
4. Click `Review + Create`, then `Create`.

### Enable Static Website Hosting
1. Open your newly created storage account. Navigate to Static website (under Data Management), click `Enable`.
2. Enter the name of your homepage file (e.g., index.html) and error page (error.html).
3. Click `Save` Azure will generate a Primary Endpoint URL for your website.

### Upload Your Website Files
1. Navigate to Storage Explorer or go to Containers in the Azure Portal.
2. Open the $web container.
3. Click Upload and select all your static website files (HTML, CSS, JavaScript). Click `Upload`.

### Access Your Static Website
- Copy the Primary Endpoint URL from the Static Website settings.
- Open the URL in your browser to see your deployed site live.