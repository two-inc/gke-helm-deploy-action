# GitHub Action to deploy Helm chart to GKE

Sample workflow :

```yaml
name: Build and deploy to GKE

on:
  push:
    branches:
      - master
      - staging
      - experimental

jobs:
  build:
    name: Build and push docker image
    runs-on: ubuntu-latest
    steps:
      -
        name: checkout
        uses: actions/checkout@v2
        with:
          fetch-depth: 0 # For push events you need to include fetch-depth: 0 OR fetch-depth: 2
      -
        name: Login to Google Container Registry
        uses: docker/login-action@v1
        with:
          registry: eu.gcr.io
          username: _json_key
          password: ${{ secrets.GCLOUD_SERVICE_KEY }}
      -
        name: Docker meta
        id: meta
        uses: docker/metadata-action@v3
        with:
          images: eu.gcr.io/tillit-api/tillit-checkout-page
          tags: |
            type=sha
      -
        name: Run docker build
        id: build-base-image
        uses: docker/build-push-action@v2
        with:
          file: Dockerfile
          tags: ${{ steps.meta.outputs.tags }}
          push: true
      -
        name: Test yarn build
        id: yarn-build
        uses: eu.gcr.io/tillit-api/tillit-checkout-page:${{ steps.mesta.outputs.tags }}
        cmd: yarn build
      -
        name: Invoke GKE helm deploy action
        uses: tillit-dot-ai/gke-cluster-deploy-action@v0.1.0
        with:
          helm-chart: checkout-page
          github-ref: ${{ github.ref }}
          image-tag: ${{ needs.build.outputs.image_tag }}
          gke-sa-key: ${{ secrets.GKE_SA_KEY }}
          helm-ssh-key:  ${{ secrets.HELM_SSH_KEY }}
```
