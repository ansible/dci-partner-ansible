# Agent Setup

We have an Ansible role to install the DCI Agent here: [dci-agent-setup](/dci-agent-setup).

At the moment the [DCI agent](https://github.com/redhat-cip/python-dciclient) is only officially supported on a CentOS 7 or RHEL 7 server, which we provide packages for. An Ansible role to install the DCI agent on Ubuntu is currently in development.

## dci-agent-setup Role
Before running the role to install the DCI agent, you need to set the following variables, which can be specified as host variables, using the `-e dci_user...` flag or using the `set_fact` module.

- `dci_user`: username for the dci server
- `dci_control_url`: DCI server (defaults to: https://api.distributed-ci.io)
- `dci_api_key`: password for the DCI user

We recommend storing these as a separate file and encrypting them with Ansible Vault.

Run the role against your server. It will perform the following actions:
- Enable the EPEL repository
- Install the DCI agent and DCI ansible libraries
- Install junit_xml and xunitmerge python packages
- Store credentials for the DCI server as environment variables
- Configures Ansible to use the DCI plugins and modules

After the role finishes, log in to the server and try the following command: `dcictl topic-list`. If your credentials are set up correctly this should return a list of Ansible versions which you can test against. If this command fails, try logging out and back in again. If that fails, check that the following environment variables are set:

- DCI_LOGIN
- DCI_PASSWORD
- DCI_CS_URL
