# Drupal on Azure App Service

This sample provides guidance and code to run a containerized Drupal image on [Azure Web Apps for Containers](https://azure.microsoft.com/en-us/services/app-service/containers/) with the Drupal database externalized to [Azure Database for MariaDB](https://docs.microsoft.com/en-us/azure/mariadb/).  The container image used is [Bitnami's Docker images for Drupal with NGINX](https://github.com/bitnami/bitnami-docker-drupal-nginx).

This guidance uses the [Bring Your Own Storage (BYOS) feature for Azure App Service](https://azure.github.io/AppService/2018/09/24/Announcing-Bring-your-own-Storage-to-App-Service.html).  This storage will be implemented as an Azure File Share, allowing multiple Drupal containers to access the file system simultaneously.  This also provides persistence of the Drupal files that would otherwise be lost when containers stop and/or restart.

> NOTE: The Azure App Service BYOS feature is currently in preview.

This guidance was tested and verified on the following platforms:
- Ubuntu 18.04 LTS

## Pre-Requisites

The following software needs to be installed on your local computer before you start.

- Azure Subscription (commercial) 
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli), v2.18 (or newer)

## Get Started

To get started, you need to perform the following tasks in order:
- Configure your environment
- Deploy the solution to Azure
- Verify your deployment

### Configure your environment
Set and save the environment variables listed below to a `.env` file to personalize your deployment.

```bash
# Azure resoure group settings
RG_NAME=                         # Resource group name, ie: 'drupal-rg'
LOCATION=eastus                  # az account list-locations --query '[].name'

# Azure App Service settings
ASP_NAME=                        # App Service Plan name, ie: 'drupal-asp'
ASP_SKU=P1V2                     # App Service Plan SKU
WEB_APP_NAME=                    # Must be globally unique, lowercase only, ie: 'drupal-web-<unique>'

# Azure storage account settings
STG_ACCT_NAME=                   # Must be globally unique, lowercase only, ie: 'drupalstg<unique>'
FILESHARE_NAME=drupal-data       # Lowercase only

# Drupal DB Settings
DB_SERVER_NAME=                  # Must be globally unique, ie: 'drupal-db-srv-<unique>'
DB_SERVER_SKU=GP_Gen5_2          # Azure Database for MariaDB SKU
DB_ADMIN_NAME=                   # Cannot be 'admin'.
DB_ADMIN_PASSWORD=               # Must include uppercase, lowercase, and numeric
DB_NAME=drupal_db
```

> NOTE: This guidance doesn't configure every feature in Drupal.  Instead, many default values are assumed.  For example, if you want to be able to administer users on your drupal site, you will need modify the `.env` and `docker-compose.yaml` file to include those settings.  You can find all the Drupal environment settings that are available [here](https://github.com/bitnami/bitnami-docker-drupal-nginx#environment-variables).

After you have updated and saved your changes to the `.env` file, open a terminal window and execute the following commands to load your environment.


```bash
# Login to your Azure Subscription
az login

# Source and export the environment variables
cd ./drupal-on-app-service
set -a  
source .env
set +a

# Perform variable subsitution on the docker-compose file that will be
# used later when the Drupal container image is deployed.
# NOTE: This is a temporary hack because Azure Web Apps doesn't currently
# support passing in an environment file when using it's docker compose feature.
envsubst < ./compose/docker-compose.yaml > ./compose/docker-compose-subst.yaml
```

### Deploy the solution to Azure

To deploy and configure the Azure resources that the Drupal workload will run on, just run `deploy.sh`.

> NOTE: This step will take about 3-4 minutes.

```bash
./deploy.sh
```

### Verify your deployment

The Drupal initialization process will take about 3 minutes to complete before your site can be accessed.  However, you can monitor the progress by streaming the logs from the web app:

```bash
az webapp log tail --resource-group $RG_NAME --name $WEB_APP_NAME
```

When the site is ready, open up your browser and navigate to your web app URL.

#### Log in to the site

The Bitnami image creates a default user with username `user` and password `bitnami`.  Click the *Log in* link in the upper right corner of the screen to log in with these credentials.

#### Create some content

After you are logged into Drupal, you should see an _Add content_ link.  Use this link to create a new page of content.

#### Verify data in the Azure FileShare

To verify your file share is where Drupal is persisting data, use Storage Explorer or the Azure Portal to explore the contents of the file share.

## Private Deployment in ASE V3

To get started, you need to perform the following tasks in order:
- Provision ASE V3 Environment
- Configure your environment
- Deploy the solution to Azure
- Verify your deployment

### Create App Service Environment V3
Follow this [tutorial](https://docs.microsoft.com/en-us/azure/app-service/environment/creation) to provision an ASE V3 while it is still in Preview.

### Configure your environment
Set and save the environment variables listed below to a `.env` file to personalize your deployment.

```bash
# Azure resoure group settings
RG_NAME=                         # Resource group name, ie: 'drupal-rg'
LOCATION=eastus                  # az account list-locations --query '[].name'

# Azure App Service settings
ASE_NAME=                        # Name of the ASE V3 env
ASP_NAME=                        # App Service Plan name, ie: 'drupal-asp'
ASP_SKU=I1v2                     # App Service Plan SKU
WEB_APP_NAME=                    # Must be globally unique, lowercase only, ie: 'drupal-web-<unique>'

# Azure storage account settings
STG_ACCT_NAME=                   # Must be globally unique, lowercase only, ie: 'drupalstg<unique>'
FILESHARE_NAME=drupal-data       # Lowercase only

# Drupal DB Settings
DB_SERVER_NAME=                  # Must be globally unique, ie: 'drupal-db-srv-<unique>'
DB_SERVER_SKU=GP_Gen5_2          # Azure Database for MariaDB SKU
DB_ADMIN_NAME=                   # Cannot be 'admin'.
DB_ADMIN_PASSWORD=               # Must include uppercase, lowercase, and numeric
DB_NAME=drupal_db
```

> NOTE: This guidance doesn't configure every feature in Drupal.  Instead, many default values are assumed.  For example, if you want to be able to administer users on your drupal site, you will need modify the `.env` and `docker-compose.yaml` file to include those settings.  You can find all the Drupal environment settings that are available [here](https://github.com/bitnami/bitnami-docker-drupal-nginx#environment-variables).

After you have updated and saved your changes to the `.env` file, open a terminal window and execute the following commands to load your environment.


```bash
# Login to your Azure Subscription
az login

# Source and export the environment variables
cd ./drupal-on-app-service
set -a  
source .env
set +a
```

### Deploy the solution to Azure

To deploy and configure the Azure resources that the Drupal workload will run on, just run `deploy.sh`.

> NOTE: This step will take about 3-4 minutes.

```bash
./deploy-private.sh
```

### Verify your deployment

The Drupal initialization process will take about 3 minutes to complete before your site can be accessed.  However, you can monitor the progress by streaming the logs from the web app:

```bash
az webapp log tail --resource-group $RG_NAME --name $WEB_APP_NAME
```

When the site is ready, login to a jumpbox and navigate to the URL

#### Jumpbox VM to View Site

> Info: You should not use Azure Bastion at this point since the VNet is linked to a private DNS Zone (https://docs.microsoft.com/en-us/azure/bastion/tutorial-create-host-portal#prerequisites)

Since this deployment is occurring within a Private vNet, you will need a VM connected to that VNet in order to access the site. 

#### Log in to the site

The Bitnami image creates a default user with username `user` and password `bitnami`.  Click the *Log in* link in the upper right corner of the screen to log in with these credentials.

#### Create some content

After you are logged into Drupal, you should see an _Add content_ link.  Use this link to create a new page of content.

#### Verify data in the Azure FileShare

To verify your file share is where Drupal is persisting data, use Storage Explorer or the Azure Portal to explore the contents of the file share.