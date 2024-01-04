# Ansible Execution Environment

## Required
- Ansible Builder
- Docker
- Container Registry (Github, Gitlab, Quay.io, ...) 

## How-To Build
Start and wait until Docker is up and running. Open a terminal or command prompt on your system. Navigate to the directory where you have your EE files and then execute the following code:

```
cd awx_fortinet_ee
ansible-builder build --context ./context --container-runtime docker
```

If the build was successful, you will notice the "Dockerfile" in the context folder. You can upload this file to quay.io or any other Container Registry you prefer. 

## How-To Publish

### Gitlab Container Registry
If you intend to utilize GitLab as a Container Registry, create the ".gitlab-ci.yml" file in the main folder with the following content:

```
stages:
  - build

.build ee:
  image: docker:latest
  stage: build
  services:
    - docker:dind
  before_script:
    - docker login -u "\$CI_REGISTRY_USER" -p "\$CI_REGISTRY_PASSWORD" \$CI_REGISTRY
  script:
    - |
      if [[ "\$CI_COMMIT_BRANCH" == "\$CI_DEFAULT_BRANCH" ]]; then
        tag=""
        echo "Running on default branch '\$CI_DEFAULT_BRANCH': tag = 'latest'"
      else
        tag=":\$CI_COMMIT_REF_SLUG"
        echo "Running on branch '\$CI_COMMIT_BRANCH': tag = \$tag"
      fi
    - docker build --pull -t "\$CI_REGISTRY_IMAGE/\$CONTAINER_NAME\${tag}" "\$DOCKERFILE_PATH"
    - docker push "\$CI_REGISTRY_IMAGE/\$CONTAINER_NAME\${tag}"

awx_fortinet_ee:
  extends: .build ee
  variables:
    DOCKERFILE_PATH: awx_fortinet_ee/context/
    CONTAINER_NAME: awx_fortinet_ee
```

### Github Container Registry ghcr.io
If you want to use GitHub as a Container Registry, create the folder '.github' with the subfolder 'workflows' and then create the YAML file with the following content: (e.g., .github/workflows/publish.yml)

```
env:
  DOTNET_VERSION: '6.0.x'

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
      container_1:
        runs-on: ubuntu-latest
        defaults:
          run:
            working-directory: './awx_fortinet_ee/context'
        steps:
          - name: 'Checkout GitHub Action'
            uses: actions/checkout@main

          - name: 'Login to GitHub Container Registry'
            uses: docker/login-action@v1
            with:
              registry: ghcr.io
              username: thenetautomationquy
              password: ${{secrets.PAT}}

          - name: 'Build Inventory Image'
            run: |
              docker build . --tag ghcr.io/thenetautomationquy/awx_fortinet_ee:latest
              docker push ghcr.io/thenetautomationquy/awx_fortinet_ee:latest

```

## Packages on ghcr.io
The AWX EE 'awx_fortinet_ee' can be used freely and is accessible at https://ghcr.io/thenetautomationquy/awx_fortinet_ee:latest.