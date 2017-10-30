# Ansible Distributed Continuous Integration Vendor Documentation

## Overview

Distributed Continuous Integration (*DCI*) testing is a system for vendors who maintain Ansible Network modules to host test equipment (switches, routers, etc.) for their modules in their own testing environments while providing an API and web interface for sharing the results of those tests with the Ansible maintainers. Network partners already have the necessary expertise and hardware/firmware matrix in house to be able to ensure compatibility with Ansible. This allows network vendors to ensure that their modules are correctly tested by giving them the ability to contribute comprehensive integration testing and test results.

## Architecture

DCI has two main components:

1. A **control server** which provides a shared set of credentials, test results, and information about tests through a web application and REST API. This server is available as distributed-ci.io

2. **DCI agent** which sits behind the vendor's firewall and runs the integration tests.

The control server is in charge of taking snapshots of the Ansible Git repo a number of times a day via the *feeder* process. It is configured to post these snapshots to *topics* which represent different releases (currently devel, and 2.3). The control server is managed by Ansible, and aside from getting access to the API, vendors do not need to worry too much about this component.

"The DCI agent is in charge of invoking integration tests. Since integration tests are just playbooks they can be run from the DCI agent periodically via cron jobs, manually or however the vendor sees fit. When this happens, the agent reaches out to the control server, authenticates, and pulls the latest snapshot for whichever topic (version) is being tested. The agent then uses Ansible playbooks to provision the correct test environment, runs the configured tests and then pushes the test results back to the control server. Once results are ready contributors from Ansible and the vendor can log in to the control server and see what happened.

# Setup

## Provisioning the Agent

We have an Ansible Role ([ansible-dci-setup](/dci-agent-setup)) to facilitate the setup of the DCI agent. This role installs dci-ansible and configures it to connect to the central distributed-ci.io server. We will provide you with the URL for our control server as well the an API keys and their corresponding RemoteCIs which need to be added to in `defaults/main.yaml` before the playbook is run. More information on setting up the agent can be found [here](/docs/agent_setup.md).

### Cron Scripts

    # Update cache of Ansible source code
    30 22 * * * DCI_CS_URL='https://api.distributed-ci.io' DCI_LOGIN='admin' DCI_PASSWORD='REDACTED' bash -c "cd ~/dci-feeders && ansible-playbook playbook.yml"
    30 10 * * * DCI_CS_URL='https://api.distributed-ci.io' DCI_LOGIN='admin' DCI_PASSWORD='REDACTED' bash -c "cd ~/dci-feeders && ansible-playbook playbook.yml"

    # Run Tests
    00 18 * * * DCI_CS_URL='https://api.distributed-ci.io' DCI_CLIENT_ID='REDACTED' DCI_API_SECRET='REDACTED' bash -c "cd ~/dci-partner-ansible && ansible-playbook -i hosts playbook.yml -e platform='vyos' -e topic='Ansible-2.4'"
    15 18 * * * DCI_CS_URL='https://api.distributed-ci.io' DCI_CLIENT_ID='REDACTED' DCI_API_SECRET='REDACTED' bash -c "cd ~/dci-partner-ansible && ansible-playbook -i hosts playbook.yml -e platform='vyos' -e topic='Ansible-2.3'"



## Configuring Tests

DCI test playbooks contain several steps which correspond to various testing states. Broadly speaking, a DCI test playbook needs to register a new job with the control server, and then keep the control server up to date on the various steps of the testing process. Each step of the testing process is defined as a different play and keeps the DCI control server up to date on which step is being run by defining the dci_status variable.

To get an idea of how this all fits together, take a look at our current test setup here: [playbook.yaml](/playbook.yaml). This example playbook is what we are currently using to test the modules that are supported by the Ansible team. These tests can be run using the following command:

   `ansible-playbook -i hosts playbook.yml -e platform='vyos' -e topic='Ansible-2.4`

Our setup currently uses two hosts: *dciagent* and *nodepool* (shown below).

When `playbook.yml` is run the following happens:

1. (from dciagent) Schedule a new job, get all the attached information (such as components, tests) to this specific job
2. (from nodepool) Ensure the node in charge of running the integration tests is deleted and re-up and running before continuing
3. (from dciagent) Retrieve the version of ansible being tested from the Control Server and put it somewhere on the filesystem
4. (from nodepool) Deletes the old version of Ansible and copies the new version being tested from dciagent onto itself
5. (from nodepool) Setup Ansible by running  `source hacking/env-setup`. Run the integrations tests by executing a playbook against the matching nodepool spawned node and generated JUnit result.
6. (from dciagent) Retrieve the Junit result and upload it back to the DCI Control Server
7. (from nodepool) Ensure the node in which the test was run is destroyed

![agent workflow diagram](/docs/assets/agent-workflow.png)

More generalized information about setting up DCI tests [can be found here](/docs/creating_tests.md)

## Other provisioning systems

We encourage you to set up your tests using playbooks based on the existing code in [/hooks](https://github.com/ansible/dci-partner-ansible/tree/master/hooks), changed to comply with what is needed for your environment.

# DCI Terminology

**User:** A User, in exactly one team. In one of the following roles:

  * **User:** Basic privilege account.

  * **Admin:** Has the ability to create users within their team.

  * **Product Owner:** Has the ability to create teams. User must be in the `Product Team`.


**Product Team:** The "top-level" team, in this case Ansible.

**Partner Team:** Network Partner team. For companies that have multiple products in and development teams we split this into groups, such as "company-product", e.g. `cisco-ios`, `cisco-nxos`. Otherwise a single team is used, such as `Juniper`.

**Remote CI:** This item represents your lab. It is an item that will hold the label that will be attached to job runs. Users of a team are associated with the Remote CIs they are allowed to interact with. Special Tests can be attached to specific Remote CIs. Remote CIs are owned by a "team", so the same name can be reused, e.g. Remote CI `net` can exist under "Ansible" and "cisco-nxos".

**Test:** Set of tests that will be run on the deployed platform. Tests can be specified at different levels. The Topic level, the Team level, the Remote CI level.

**Job:** A run of a topic against a specific component.

**Topic:** Main item of focus. Example are ``Ansible-2.3``, ``Ansible-2.4``, ``Ansible-devel``. The topics are created and owned by the "Product Team" (Ansible) so they can be common and sharded between "Partner Teams". If the topics are not visible please ask for them to be shared.

**Component:** Topics are made of components. Components are items that will be tested during the deployment. In the case of Ansible 2.3 for example, components are Ansible 2.3 stable branch snapshots

**Jobstate:** A job goes through different phases and does different things during those phases. In DCI terminology, the name of those phases are strictly enforced. Jobs go from: new -> pre-run -> running -> post-run -> success|fail

**Feeders:** Fetches Ansible source code for the specific Topics and stored on dci-agent host, the applicable version is used in [/hooks/running.yml](https://github.com/ansible/dci-partner-ansible/blob/master/hooks/running.yml)

# On-Boarding Steps

This section outlines the steps needed to add a partner into the DCI system.


## Meeting with Partner

* High level explanation of where we are
* Show DCI interface
    * Global State
    * Result runs
    * runs
* Show how the states corresponding to [/hooks/](https://github.com/ansible/dci-partner-ansible/tree/master/hooks)

   * Running: Source Ansible
   * Ansible setup uses NodePool, so machines are already available - they are deleted and recreated in `teardown.yml` so on next run we have machines ready

## Setup and integration

The following is the list of steps to be completed:

**www.distributed-ci.io**

* Create new Partner Team (Red Hat)

  * [DCI Teams](https://www.distributed-ci.io/#!/admin/teams)
  * If Ansible modules exist for multiple platforms for this company, then we create one team per product, using the Ansible platform prefix, see existing teams for examples.

* Give Parner Team access to Topics (Red Hat)

  * [DCI Topics](https://www.distributed-ci.io/topics)
  * For each of the appropriate topics, edit and add the new team to the "Topic's teams" list


* Create Admin user for Partner Team (Red Hat)

  * [DCI Users](https://www.distributed-ci.io/#!/admin/users)
  * Create new user

    * Login: GitHub name
    * Full Name
    * Email
    * Password: Enter a temporary password
    * Team: Select their Partner Team
    * Role: Admin


* Email Partner Team Admin (Red Hat)

Send an email to the new Partner Team Admin with the following

Subject: Distributed CI (DCI) account details for $team

```
Hi,
We've just created an account for you on https://www.distributed-ci.io

The account has "Admin" privileges, meaning you can create standard "user" accounts for the other members of your team.

Please see LINK TO THIS PAGE for details.

Any issues reply to this email, or feel free to use the `#ansible-network` channel on Freenode.

Username: FIXME

Tempoarary Password: FIXME

Please update your password by visiting https://www.distributed-ci.io/#!/password before proceding further.



```

* Create Users for Team (Partner Admin)

  * [DCI Users](https://www.distributed-ci.io/#!/admin/users)
  * Create new user

    * Login: GitHub name
    * Full Name
    * Email
    * Password: Enter a temporary password
    * Team: Select their Partner Team
    * Role: User

* Create Remote CI for each product (Partner Admin)

  * [DCI Remote CI](https://www.distributed-ci.io/#!/admin/remotecis)
  * Create new Remove CI

    * Name: Generally the Ansible module prefix. As the Remote CI exists within the "Team", names do not need to be globally unique.

    * Team: Select your Team
    * Allow upgrade: Keep default (unchecked)
    * Click "Create"
  * Make a note of the details from the `dcirc.sh` file, you will need these when setting up the `dci-agent`


**dci-agent**

Partner to create a CentOS 7 or RHEL7 node within their lab environment. Other Operating systems are not supported.

* Firewall to allow outgoing access to `distributed-ci.io`, incoming access is not required
* Install [dci-agent-setup role ](https://github.com/ansible/dci-partner-ansible/tree/master/dci-agent-setup/) which will enable EPEL
* Install and adapt [dci-partner-ansible](https://github.com/ansible/dci-partner-ansible) to the partner's needs
* Schedule the jobs to run periodically from the dciagent node
