# Ansible Execution Environment

## Required
- Ansible Builder
- Docker
- Container Registry (Github, Gitlab, Quay.io, ...) 

## How-To
Start and wait until Docker is up and running. Open a terminal or command prompt on your system. Navigate to the directory where you have your EE files and then execute the following code:

```
cd awx_fortinet_ee
ansible-builder build --context ./context --container-runtime docker
```

If the build was successful, you will notice the "Dockerfile" in the context folder. You can upload this file to quay.io or any other Container Registry you prefer. If you want to use GitLab as a Container Registry, then check out the .gitlab-ci.yml file.