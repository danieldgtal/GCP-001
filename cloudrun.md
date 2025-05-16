# Simple Deployment to Google Cloud Run

This document outlines the steps to deploy a sample application to Google Cloud Run.

## Prerequisites

- You have a Google Cloud Platform account.
- The Google Cloud CLI (`gcloud`) is installed and configured.

## Deployment Steps

1.  **Set the Google Cloud Project:**

    ```bash
    gcloud config set project YOUR_GCP_PROJECT_ID
    ```

    _(Replace `YOUR_GCP_PROJECT_ID` with your actual GCP project ID.)_

2.  **Enable the Cloud Run API and configure your shell environment:**

    ```bash
    gcloud services enable run.googleapis.com
    ```

    ### Set compute region location

    ```bash
    gcloud config set compute/region "us-central1-a"
    ```

    ### Create a LOCATION environment variable

    LOCATION="us-central1-a"

3.  **Write the sample application:**

    ```bash
      mkdir helloworld && cd helloworld

      nano package.json

      {
        "name": "helloworld",
        "description": "Simple hello world sample in Node",
        "version": "1.0.0",
        "main": "index.js",
        "scripts": {
          "start": "node index.js"
        },
        "author": "Google LLC",
        "license": "Apache-2.0",
        "dependencies": {
          "express": "^4.17.1"
        }
      }


    ```

    ### Create a simple WebServer Application

    nano index.js
    <code>
    const express = require('express');
    const app = express();
    const port = process.env.PORT || 8080;

        app.get('/', (req, res) => {
          const name = process.env.NAME || 'World';
          res.send(`Hello ${name}!`);
        });

        app.listen(port, () => {
          console.log(`helloworld: listening on port ${port}`);
        });

    </code>

4.  **Containerize: application and upload to Artifact Registry**

    ### Create Dockerfile

    ```bash
    # Use the official lightweight Node.js 12 image.

    # https://hub.docker.com/_/node

    FROM node:12-slim

    # Create and change to the app directory.

    WORKDIR /usr/src/app

    # Copy application dependency manifests to the container image.

    # A wildcard is used to ensure copying both package.json AND package-lock.json (when available).

    # Copying this first prevents re-running npm install on every code change.

    COPY package\*.json ./

    # Install production dependencies.

    # If you add a package-lock.json, speed your build by switching to 'npm ci'.

    # RUN npm ci --only=production

    RUN npm install --only=production

    # Copy local code to the container image.

    COPY . ./

    # Run the web service on container startup.

    CMD [ "npm", "start" ]

    ```

    ### build image

    ```bash
    gcloud builds submit --tag gcr.io/$GOOGLE_CLOUD_PROJECT/helloworld
    ```

    ### List built images

    ```bash
    gcloud container images list
    ```

    ### Register gcloud as the credential helper for all Google-supported Docker registries:

    ```bash
    gcloud auth configure-docker
    ```

    ### To run and test the application locally from Cloud Shell, start it using this standard docker command:

    ```bash
    docker run -d -p 8080:8080 gcr.io/$GOOGLE_CLOUD_PROJECT/helloworld
    ```

5.  **Deploy to Cloud Run**

    ```bash
    gcloud run deploy --image gcr.io/$GOOGLE_CLOUD_PROJECT/helloworld --allow-unauthenticated --region=$LOCATION
    ```

    ### Clean up

    ```bash
    gcloud container images delete gcr.io/$GOOGLE_CLOUD_PROJECT/helloworld

    gcloud run services delete helloworld --region="REGION"
    ```
