   * [NRCAN API](#nrcan-api)
   * [Configuration](#configuration)
      * [CircleCI](#circleci)
      * [Docker Hub](#docker-hub)
      * [Azure Web App for Containers](#azure-web-app-for-containers)
      * [Setup](#setup)
      * [Enabling Continuous Deployment](#enabling-continuous-deployment)
      * [Troubleshooting](#troubleshooting)

NRCAN API
=========

rough docs.


Configuration
==========

CircleCI
--------
We're using Circle CI to run continuous integration and continuous deployment of the NRCAN API repository. There's a job called `deploy` in `.circle/config.yml` that controls the build and deploy of the Docker image.  For each commit to master, the `deploy` job will build a docker image and push it to Docker Hub with tag "latest" as well as a version number as defined by $VERSION in `.circleci/config.yml`

Docker Hub
----------
When a new image arrives at Docker Hub, a webhook is sent to Azure and the Azure App Service for Containers will download and deploy the "latest" image.

Azure Web App for Containers
----------------------------

Setup
-----
The Azure Web App for Containers is created using an ARM template called deployazure.json. The template should be executed as follows:
```
az group deployment create -n ContainerName --resource-group ResourceGoupName --template-file deployazure.json
```
The template will ask for `App Service Plan ID`, `App Name`, `Docker Image`, `Collection Name`, `Connection String`, `DB Name`, and `API Key`

Enabling Continuous Deployment
------------------------------
`az webapp deployment container config -n nrcan123252637 -g templatenrcan -e true`

```{
  "CI_CD_URL": "https://$nrcan123252637:PASSWORDHERE@nrcan123252637.scm.azurewebsites.net/docker/hook",
  "DOCKER_ENABLE_CI": true
}```

In case you lose the CI/CD url, you can find it using this url:

`az webapp deployment container show-cd-url -n nrcan123252637 -g templatenrcan`

```{
  "CI_CD_URL": "https://$nrcan123252637:PASSWORDHERE@nrcan123252637.scm.azurewebsites.net/docker/hook",
  "DOCKER_ENABLE_CI": true
}
```

* Add the CI_CD_URL URL to Docker Hub.
** On your Docker Hub Repository, click Webhooks and add your new webhook URL.


Troubleshooting
---------------

