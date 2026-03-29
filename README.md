# ansible-deploy-app

Ansible project for deploying a sample Kubernetes application using a Harbor registry pull secret.

## Overview

This repository contains an Ansible-based Kubernetes deployment workflow that:
- creates a namespace
- creates a Harbor pull secret
- deploys a `Deployment` and `Service`
- waits for the deployment to become available
- optionally rolls back a deployment

## Repository layout

- `inventory/hosts.ini` - Ansible inventory for local execution
- `playbooks/deploy_app.yml` - Main deployment playbook
- `playbooks/rollback_app.yml` - Rollback playbook
- `vars/app_vars.yml` - Deployment variables and registry settings
- `templates/deployment.yml.j2` - Kubernetes Deployment manifest template
- `templates/service.yml.j2` - Kubernetes Service manifest template
- `ansible-ee-workshop/` - Execution environment definition and dependency manifests

## Prerequisites

- Ansible installed locally
- `kubectl` configured to access the target Kubernetes cluster
- Access to the Harbor registry specified in `vars/app_vars.yml`
- Python packages defined in `ansible-ee-workshop/requirements.txt` if using the execution environment

## Variables

The deployment is configured in `vars/app_vars.yml`.
Important values to customize:
- `app_name`
- `app_namespace`
- `app_image`
- `app_tag`
- `app_replicas`
- `app_port`
- `app_node_port`
- `harbor_url`
- `harbor_user`
- `harbor_password`

> Note: `harbor_user` and `harbor_password` are expected to be defined via extra vars or environment variables when running the playbook.

## Deploying the application

From the repository root:

```powershell
ansible-playbook -i inventory/hosts.ini playbooks/deploy_app.yml -e "harbor_user=<USER> harbor_password=<PASSWORD>"
```

If you need to override variables, use `-e` with additional values:

```powershell
ansible-playbook -i inventory/hosts.ini playbooks/deploy_app.yml -e "app_tag=1.0.0 harbor_user=<USER> harbor_password=<PASSWORD>"
```

## Rolling back a deployment

```powershell
ansible-playbook -i inventory/hosts.ini playbooks/rollback_app.yml -e "harbor_user=<USER> harbor_password=<PASSWORD>"
```

## Execution environment

The `ansible-ee-workshop` directory contains an Ansible Execution Environment definition.

Build the EE image with `ansible-builder`:

```powershell
ansible-builder build -t ansible-ee-workshop:latest -f ansible-ee-workshop/execution-environment.yml
```

Run the playbook inside the execution environment:

```powershell
podman run --rm -v ${PWD}:/workspace -w /workspace ansible-ee-workshop:latest \
  ansible-playbook -i inventory/hosts.ini playbooks/deploy_app.yml -e "harbor_user=<USER> harbor_password=<PASSWORD>"
```

## Notes

- The playbook creates a Kubernetes Secret named `harbor-secret` in the target namespace.
- The service uses `NodePort` and exposes the app on `app_node_port`.
- The deployment includes readiness and liveness probes.
- If your environment uses a different package manager, adjust `ansible-ee-workshop/bindep.txt` and `execution-environment.yml` accordingly.