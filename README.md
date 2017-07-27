# dci-ansible-partner

The integration playbook for Distributed CI (DCI) and Ansible networking plugins

## How to run me ?

### Prerequisites

1. Have an account on https://www.distributed-ci.io
2. Have a valid RemoteCI created
3. Install `epel-release` and `https://packages.distributed-ci.io/dci-release.el7.noarch.rpm` packages
4. Install the `dci-ansible` package


### Usage

Create a file called `dcirc.sh` that looks like the following:

```
export DCI_LOGIN='alogin'
export DCI_PASSWORD='apassword'
export DCI_CS_URL='https://api.distributed-ci.io'
```

Then `source` this file. `git clone` this $repository and create the appropriate `hosts` file.

Then simply run the command `ansible-playbook -i hosts playbook.yml`.


## Variables

The playbook can be driven through variables. The two most important one being `platform` and `topic`

  * `platform`: The platform to run integration test for (openvswitch, vyos, cisco, ...).
  * `topic`: The topic to run the integration test against (Ansible-2.3, Ansible-2.4, ...).

One can try any matrices one wants:

  * `ansible-playbook -i hosts playbook.yml -e platform='openvswitch' -e topic='Ansible-2.3'`
  * `ansible-playbook -i hosts playbook.yml -e platform='openvswitch' -e topic='Ansible-2.4'`
  * `ansible-playbook -i hosts playbook.yml -e platform='vyos' -e topic='Ansible-2.3'`
  * `ansible-playbook -i hosts playbook.yml -e platform='vyos' -e topic='Ansible-2.4'`

## Questions ?

Freenode: `#ansible-network`

Distributed-CI Team:  <distributed-ci@redhat.com>
