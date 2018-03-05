   * [NRCAN API and Endpoint](#nrcan)
      * [Prerequisites](#prerequisites)
   * [Configuration](#configuration)
      * [CircleCI](#circleci)
      * [Docker Hub](#docker-hub)
      * [Azure Web App for Containers](#azure-web-app-for-containers)
      * [Azure Function App](#azure-function-app)
      * [DNS](#dns)
   * [Troubleshooting](#troubleshooting)

NRCAN
=========

Installation documentation for the NRCan API and Endpoint.


Prerequisites
-------------

The following documentation assumes you have set up the following:

* A Resource Group to place the application in
* A Service Plan
* Access to the Azure CLI (on your desktop or a server with network access to Azure)

Configuration
==========

CircleCI
--------
We're using Circle CI to run continuous integration and continuous deployment of the NRCAN API repository. There's a job called `deploy` in `.circle/config.yml` that controls the build and deploy of the Docker image.  For each commit to master, the `deploy_api` and `deploy_etl` jobs will build a docker image and push it to Docker Hub with tag "latest" as well as the SHA from the last commit.  

Docker Hub
----------
When a new image arrives at Docker Hub, a webhook is sent to Azure and the Azure App Service for Containers will download and deploy the "latest" image.

The relevant images are:
`docker.io/cdssnc/nrcan_api:latest`
and
`docker.io/cdssnc/nrcan_etl:latest`

When a new image arrives at Docker Hub, Docker will send a webhook to either Azure WEb App for Containers or Azure Function App.  

Azure Web App for Containers (for API only)
----------------------------


Create a Resource Group:

```
az group create --name MyGroup -l canadaeast
```

Create a Service Plan

```
az appservice plan create -g NRCanGroup -n webapplinux --is-linux -l canadaeast
```

The Azure Web App for Containers is created using an ARM template called deploy_api.json. The template should be executed as follows:
```
az group deployment create -n AppName --resource-group ResourceGoupName --template-file deploy_api.json
```
The template will ask for `App Service Plan ID`, `App Name`, `Docker Image`, `Collection Name`, `Connection String`, `DB Name`, and `API Key`

* Enabling Continuous Deployment

`az webapp deployment container config -n AppName -g ResourceGroupName -e true`

```
{
  "CI_CD_URL": "https://$nrcan123252637:PASSWORDHERE@nrcan123252637.scm.azurewebsites.net/docker/hook",
  "DOCKER_ENABLE_CI": true
}
```

In case you lose the CI/CD url, you can find it using this url:

`az webapp deployment container show-cd-url -n nrcan123252637 -g templatenrcan`

```
{
  "CI_CD_URL": "https://$nrcan123252637:PASSWORDHERE@nrcan123252637.scm.azurewebsites.net/docker/hook",
  "DOCKER_ENABLE_CI": true
}
```

Add the CI_CD_URL URL to Docker Hub. On your Docker Hub Repository, click Webhooks and add your new webhook URL

Azure Function App
------------------

The Exndpoint Extractor runs as an Azure Function App. There's an ARM template called `deploy_etl.json` that will set up the function service for you. You need to specify the the app name, docker image, and the storage connection string.

The ETL also runs as an Azure Function App. There's an ARM template called `deploy_etl.json` that will set up the function app.  You need to specify the app name and docker image.


DNS
===
Configure a custom DNS name: In the Azure portal, open the API resource, click "Custom domains", click "+ Add hostname", enter your custom hostname. You must create CNAME pointing to the Azure DNS name.

(this should be added to the ARM template....)

Troubleshooting
===============

`az webapp log show --resource-group Group-Name --name App-Name`

