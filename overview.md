# Ansible Distributed Continuous Integration Vendor Documentation

## Overview

Distributed Continuous Integration (*DCI*) testing is a system for vendors who maintain Ansible Network modules to host test equipment (switches, routers, etc.) for their modules in their own testing environments while providing an API and web interface for sharing the results of those tests with the Ansible maintainers. Network partners already have the necessary expertise and hardware/firmware matrix in house to be able to ensure compatibility with Ansible.This allows network vendors to ensure that their modules are correctly tested by giving them the ability to contribute comprehensive integration testing and test results.

## Architecture

DCI has two main components:

1. A **control server** which provides a shared set of credentials, test results, and information about tests through a web application and REST API. This server is available as distributed-ci.io

2. **DCI agent** which sits behind the vendor’s firewall and runs the integration tests.

The control server is in charge of taking snapshots of the Ansible Git repo a number of times a day via the *feeder* process. It is configured to post these snapshots to *topics* which represent different releases (currently devel, and 2.3). The control server is managed by Ansible, and aside from getting access to the API, vendors don’t need to worry too much about this component.

The DCI Agent, which triggers the tests locally. This can be done using cron jobs, or however the vendor sees fit. When this happens, the agent reaches out to the control server, authenticates, and pulls the latest snapshot for whichever topic (version) is being tested. The agent then uses Ansible playbooks to provision the correct test environment, runs the configured tests and then pushes the test results back to the control server. Once results are ready contributors from Ansible and the vendor can log in to the control server and see what happened.

## Setup

### Provisioning the Agent

The current version of the DCI agent requires an RPM based operating system (CentOS, RHEL). We have an Ansible Role to install [ansible-dci-roles](https://github.com/newswangerd/ansible-dci-roles) to facilitate the setup of the DCI agent. This role installs dci-ansible and configures it to connect to the central distributed-ci.io server. We will provide you with the URL for our control server as well as an API key and user which need to be added to in defaults/main.yaml before the playbook is run.

## Configuring Tests

DCI test playbooks contain several steps which correspond to various testing states. Broadly speaking, a DCI test playbook needs to register a new job with the control server, and then keep the control server up to date on the various steps of the testing process. Each step of the testing process is defined as a different play and keeps the DCI control server up to date on which step is being run by defining the dci_status variable.

## Provisioning test machines

### OpenStack

Source Repo: [dci-partner-ansible](https://github.com/ansible/dci-partner-ansible).

This setup uses two hosts: *dciagent* and *nodepool* (shown below).

FIXME add image

When `playbook.yml` is run the following happens:

1. (from dciagent) Schedule a new job, get all the attached information (such as components, tests) to this specific job
2. (from nodepool) Ensure the node in charge of running the integration tests is deleted and re-up and running before continuing
3. (from dciagent) Retrieve the version of ansible being tested from the Control Server and put it somewhere on the filesystem
4. (from nodepool) Deletes the old version of Ansible and copies the new version being tested from dciagent onto itself
5. (from nodepool) Setup Ansible by running  ‘source hacking/env-setup’. Run the integrations tests by executing a playbook against the matching nodepool spawned node and generated JUnit result.
6. (from dciagent) Retrieve the Junit result and upload it back to the DCI Control Server
7. (from nodepool) Ensure the node in which the test was run is destroyed

## Other provisioning systems

We encourage you to set up your tests using based on the existing code in [/hooks](https://github.com/ansible/dci-partner-ansible/tree/master/hooks) with what is needed for your environment.

# DCI Terminology

**User:** A User. Two possible roles, a team admin, with administrator privileges and a regular user.

**Team:** Users are part of a team. Distributed-CI is a multi-tenant platform, where each team is isolated. As of now, users can only be part of a single team.

**Remote CI:** This item represents your lab. It is an item that will hold the label that will be attached to job runs. Users of a team are associated with the Remote CIs they are allowed to interact with. Special Tests can be attached to specific Remote CIs

**Test:** Set of tests that will be run on the deployed platform. Tests can be specified at different level. The Topic level, the Team level, the Remote CI level.

**Job:** A run of a topic against a specific component.

**Topic:** Main item of focus. Example are Ansible 2.2, Ansible 2.3, Ansible 2.4, ...

**Component:** Topics are made of components. Components are items that will be tested during the deployment. In the case of Ansible 2.3 for example, components are Ansible 2.3 stable branch snapshots

**Jobstate:** A job goes through different phases and does different things during those phases. In DCI terminology, the name of those phases are strictly enforced. Jobs go from: new -> pre-run -> running -> post-run -> success|fail

**Feeders:** Fetches Ansible source code for the specific Topics and stored on dci-agent host, the applicable version is used in [/hooks/running.yml](https://github.com/ansible/dci-partner-ansible/blob/master/hooks/running.yml)

# On-Boarding Steps

This section outlines the steps needed to add a partner into the DCI system.


## Meeting with Partner

* Hi level explanation of where we are
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

* Create the team in www.distributed-ci.io
* Create the accounts for the member of the team that will use www.distributed-ci.io
* Create the necessary RemoteCIs (one per lab)
* Give the team members the necessary permissions

**dci-agent**

Partner to create a CentOS 7 or RHEL7 node within their lab environment.

* Firewall to allow outgoing access to `distributed-ci.io`, incoming access is not required
* Install [dci-agent role ](https://github.com/newswangerd/ansible-dci-roles)
* Install and adapt (dci-partner-ansible)[https://github.com/ansible/dci-partner-ansible] to the partner’s needs
* Schedule the jobs to run periodically from the dciagent node


