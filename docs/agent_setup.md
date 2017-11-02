# Agent Setup

We have an Ansible role to install the DCI Agent here: [dci-agent-setup](/dci-agent-setup).

At the moment the [DCI agent](https://github.com/redhat-cip/python-dciclient) is only officially supported and tested on CentOS 7 and RHEL 7 servers, which we provide packages for. If vendors prefer to use an Ubuntu host, the `dci-agent` role can also be used to install the DCI agent on Ubuntu.

## Role Setup
Before running the role to install the DCI agent, configure the RemoteCIs that you wish to install credentials for in [roles/dci-agent/defaults/main.yaml](/dci-agent-setup/roles/dci-agent/defaults/main.yaml). Each platform (RemoteCI) that you are testing requires a separate API key, and are configured using a list of dictionaries like so:

```
remotecis:
  - name: platform1
    remoteci_id: REMOTE_CI_ID
    dci_control_url: https://api.distributed-ci.io
    remoteci_api_key: REMOTE_CI_API_KEY

  - name: platform2
    remoteci_id: REMOTE_CI_ID_2
    dci_control_url: https://api.distributed-ci.io
    remoteci_api_key: REMOTE_CI_API_KEY_2
```

The credentials for your *Remote CI* can be found at [DCI Remote CI](https://www.distributed-ci.io/admin/remotecis) and downloading the configuration file.


Each entry in this list will be saved as a shell script in `/etc/dci` which can be activated using `source /etc/platform1.sh` in order to set up the environment variables required to connect to the DCI control server. The scripts in this directory will be readable by any operating system use that belongs to the group specified in `dci_group_name` variable. Running DCI tests requires that the DCI_CS_URL, DCI_CLIENT_ID and DCI_API_SECRET environment variables be set, which are all set by these scripts.

We recommend encrypting [dci-agent/defaults/main.yaml](/dci-agent-setup/roles/dci-agent/defaults/main.yaml) using Ansible Vault to protect DCI credentials.


## Running the dci-agent role

Running the `dci-agent` role on your server will perform the following actions:
- Enable the EPEL repository (on CentOS/RHEl)
- Install the DCI agent and DCI ansible libraries
- Install junit_xml and xunitmerge python packages
- Store credentials for the RemoteCIs in `/etc/dci`
- Configures Ansible to use the DCI plugins and modules

After the role finishes, log in to the server and try the following command: `source /etc/dci/platform1.sh && dcictl topic-list`. If your credentials are set up correctly this should return a list of Ansible versions which you can test against.
