# Setting up Tests to Run in DCI

DCI tests do the following:

1. Register a new job run with the DCI server
2. Download the version of ansible being tested and set up an appropriate test environment to run it on.
3. Execute a command to run tests such as `ansible-playbook` or `ansible-test` which generate junit files with the results
4. Upload the junit results to the DCI server for viewing

If you haven't done so already, open up [our example playbook](/playbook.yml) to view the needed steps. This playbook constitutes a series of plays which represent different steps the test's life cycle: _new_, _pre-run_, _running_, _post-run_, and _success_.

This playbook is intended to be run using the dci callback plugin which should already be configured in `/etc/ansible/ansible.cg`. This plugin requires that a couple of variables are set in the playbook in order to function correctly.

- *job_informations*: this must be set to the returned value from the `dci_job` module. It contains the information required to post the results back to the DCI server.
- *dci_status*: must be set at the beginning of each play in the playbook. This tells the server which step the testing is currently on.
