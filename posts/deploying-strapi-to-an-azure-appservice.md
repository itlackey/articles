---
title: Deploying Strapi to an Azure AppService
description: Deploying Strapi to an Azure AppService can be done with a few additional steps. This guide provides additional steps to get Strapi running on Azure AppService.
tags: 'webdev, devops, azure, strapi'
published: true
id: 1656904
date: '2023-11-04T16:06:19.040Z'
---
I have seen a lot of people have issues deploying a Strapi JS application on Azure AppServices. It is not an intuitive process to say the least. In this post, we'll walk through the challenges and configurations necessary to deploy Strapi on Azure AppServices.

## AppService Configuration
The following sections will discuss the changes needed to modify the Azure AppService deployment process to work with Strapi.

### The Deployment File

Azure AppServices will run some build steps during deployment. However, we can change the default process by including a `.deployment` file to the root of your Strapi project. This file works like a tells Azure what to do during deployment:

```plaintext
[config]
SCM_DO_BUILD_DURING_DEPLOYMENT = false
NODE_ENV = production
COMMAND = bash deploy.sh
```

The counterpart to the `.deployment` file is the `deploy.sh` script, which orchestrates the deployment process. Create this file in the root of your application and include this snippet:

```bash
#! /bin/bash
echo "Removing src, config"
rm -rf /home/site/wwwroot/src
rm -rf /home/site/wwwroot/config
rsync -arv --no-o --no-g --ignore-existing --size-only  ./ /home/site/wwwroot
echo "Source sync done."
```

This script ensures that any old, unwanted files are removed, preventing obsolete files from causing issues at runtime.

### Node Version and Start Command

Azure needs to know which version of Node.js to use and how to get your application running. On the AppService's Configuration page, under the General Settings tab, make sure the Major and Minor versions are set to match Node 18 LTS or the latest version that Strapi supports.

You will also want to make sure your `package.json` scripts include the path to the `strap.js` file:

```json
"start": "node node_modules/@strapi/strapi/bin/strapi.js start",
"build": "node node_modules/@strapi/strapi/bin/strapi.js build",
```

*Note:* The node version is likely to change. Please refer to the latest Strapi and Azure documentation to select the correct version.

### Disabling SCM/Oryx Builds

By default, Azure AppServices will run a build process with SCM/Oryx during deployment, but this isn't needed for the Strapi app since we provide a custom deployment script. Disable it by setting the following application settings:

```plaintext
ENABLE_ORYX_BUILD=false
SCM_DO_BUILD_DURING_DEPLOYMENT=false
```

### Disabling node_modules Compression

Microsoft updated AppServices to include a feature that will zip the `node_modules` folder and move it to the local container file system. This does provide a performance improvement for more applications. However, Strapi doesn't play nicely with the way Azure zips and relocates the `node_modules` folder. You will need to disable this feature to avoid issues with Strapi, disable this feature by updating this application setting:

```plaintext
BUILD_FLAGS=Off
```

More information about Oryx and Node.js can be found [here](https://github.com/microsoft/Oryx/blob/main/doc/hosts/appservice.md#nodejs).

### Optional: Extend Container Startup Time

If your Strapi application is taking too long to start, you might want to increase the amount off time Azure will wait for the container to start. Update the following application setting if needed:

```plaintext
WEBSITES_CONTAINER_START_TIME_LIMIT=1600
```

## GitHub Action Configuration

To enable automated deployments you will need to set up a GitHub action to package the Strapi application for Azure AppServices. This is pretty standard for a Node.js application but let's review a few important details.

### Specify Node.js Version

Ensure you're using the correct version of Node.js in your GitHub action workflow:

```yaml
steps:
  - uses: actions/checkout@v3

  - name: Set up Node.js version
    uses: actions/setup-node@v3
    with:
      node-version: 18
```
*Note:* The node version is likely to change. Please refer to the latest Strapi and Azure documentation to select the correct version.

### Build and Zip Strapi

These steps will install the necessary node modules and run the Strapi build process. Once those are done, the next step will create a zip file containing all the application, including the `.build` and `node_modules` folders. It then uploads the zip file to GitHub artifacts.

```yaml
- name: npm install, build, and test
  env:
      NODE_ENV: production
  run: |
      npm install
      npm run build

- name: Create Archive of application code
  run: |
      mkdir -p zip
      zip  -r zip/app.zip . -x@.zipignore

- name: Upload artifact for deployment job
  uses: actions/upload-artifact@v3
  with:
      name: dimm-city-data
      path: zip/app.zip
```

Don't forget to use a `.zipignore` file to exclude unnecessary files from the zip. Create the file in the root of the application and include these entries:

```plaintext
.editorconfig
.env*
.eslint*
.git/*
.github/*
.gitignore
.strapi-updater.json
.tmp/*
.vscode/*
.zipignore
README.md
tools/*
zip/*
```

## Wrapping Up

Deploying Strapi on Azure AppServices can be a bit challenging. Knowing which settings to adjust can take some trial and error until you wrap your head around the Azure deployment process. Hopefully this guide helps make the deploying your Strapi application to Azure a little less daunting.

Here are a few links for additional information:
- [Official Strapi Documentation for Azure Deployment](https://docs.strapi.io/dev-docs/deployment/azure)
- [Strapi Forum Discussions on Azure Deployment](https://forum.strapi.io/search?q=azure%20deploy)

Please feel free to post any questions or comments you have below, or reach out on social media.

Happy coding!
