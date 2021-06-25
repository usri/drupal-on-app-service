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

To get started, you can deploy based on the following deployment options:
- standard-app-service deployment
- private-ase-v3-deployment