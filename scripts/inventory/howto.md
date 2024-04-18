# How to add GLPI as dynamic inventory for Ansible Automation Platform

## Create Execution Environment on Ansible Automation Hub

Using Python requirements.txt from [here](https://github.com/chornberger-c2c/ansible-collection-glpi/blob/master/requirements.txt).

execution-environment.yml:
```
---
version: 1

build_arg_defaults:
  EE_BASE_IMAGE: 'quay.io/ansible/awx-ee'
  #EE_BUILDER_IMAGE: 'registry.redhat.io/ansible-automation-platform-24/ansible-builder-rhel9'

dependencies:
  #galaxy: requirements.yml
  python: requirements.txt
  # system: bindep.txt
```
Steps to be done on local Automation Hub:
```
podman pull quay.io/ansible-automation-platform-24/ee-supported-rhel8
podman pull quay.io/ansible/awx-ee
podman login -u admin --tls-verify=false automationhub.local
ansible-builder build -t quay.io/mynamespace/custom-ee
podman push quay.io/mynamespace/custom-ee
podman tag quay.io/mynamespace/custom-ee automationhub.local/mynamespace/custom-ee
podman push --tls-verify=false automationhub.local/mynamespace/custom-ee
```

## Create project in Ansible Automation Platform

Project must point to this [repository](https://github.com/chornberger-c2c/ansible-collection-glpi) or a fork.

The new execution environment must be used.

## Create Inventory in Ansible Automation Platform

### Sources
`Source` -> Sourced from a Project -> use project we just created

`Execution Environment` -> use execution environment we created on local automationhub that contains Python requirements

`Inventory file` -> `scripts/inventory/glpi-api.py` -> config file `glpi-api.yml` is located next to it in the repository

`Source variables`
```
---
ANSIBLE_GLPI_URL: http://glpi.local/apirest.php/
ANSIBLE_GLPI_USERTOKEN: 123abc
ANSIBLE_GLPI_APPTOKEN: 456def
```

#### How to get the correct values:
```
ANSIBLE_GLPI_URL: URL to the platform
ANSIBLE_GLPI_USERTOKEN: User token (Administration -> Users -> <USER> -> ALL -> Remote access keys -> API token)
ANSIBLE_GLPI_APPTOKEN: API client token (Setup -> General -> API -> <CLIENT> -> Application token (app_token))

(Setup -> General -> API -> Enable Rest API)
```
