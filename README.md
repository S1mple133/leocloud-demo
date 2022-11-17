# Deployment of a Basic Quarkus Application to Leocloud

## Setup

### Github Secrets

In order to deploy this application, you need to set the ``KUBECONFIG`` secret. It should contain the contents of 
the kubeconfig file you use to connect to LeoCloud. 

But, you need to add following 2 lines under ``users.oidc.user.exec.args``

This is needed, because you cannot open a browser window in a gh-runner to log in and the LeoCloud uses OIDC (Keycloak) for authentication.

```
- --username=[LEOCLOUD EMAIL]
- --password=[LEOCLOUD PASSWORD]
```

You also have to set te ``LEOCLOUD_NAMESPACE`` secret. It should contain your namespace (`student-v-nachname` ex. `student-d-pavelescu`)


### Helm Values

You can configure your deployment under `helm/values.yml`. 

#### Replacing the Deployment URL

You should replace the deployment URL of your application in the helm values file (`ingress.path`). It should start with your 
LeoCloud Username (/v.nachname/) and then it can be followed by anything you want.

#### Replacing the Image Name

The Github Action Build Job compiles and pushes the image to the Github Container Registry. The name of the image is based on your username and repository name. 
You need to set your own in the helm values file. (`image.repository`)

## Note

If you use this repository as a template for your own deployment, make sure the access to the ghcr images is public (by default in public Github Repositories). 
If your repository is private, you can still make the images public by opening the `Package page`, then clicking on `Package Settings > Change Visibility` in GitHub.

If you wish to use private ghcr images, then you have add a Secret containing your GitHub Token and username, which is described [here](https://dev.to/asizikov/using-github-container-registry-with-kubernetes-38fb)

## Description

Every time a push event is triggered on the main branch (merge, commit, ...), the quarkus application is compiled
using a multi-stage Docker image.

It is then pushed to the `Github Container Registry (ghcr.io)`.

Afterwards, the Deploy job starts and it sets up kubectl and helm. It then updates the helm deployment to update 
changes in the k8s manifests and helm `values` files

As a last step, it triggers a rollout restart, which restarts the leocloud-demo deployment (without downtime) and pulls 
the new latest image from the Github Container Registry.

In a normal production environment, you should not use "latest" as an image tag. Then, you would just have to update the tag on 
every release (using helm values overrides). Doing so, you do not need the last step where we rollout restart the deployment 
because the helm upgrade step would notice the change in the image tag and automatically pull the new image.