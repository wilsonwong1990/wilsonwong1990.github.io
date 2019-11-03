# Running Ansible playbooks via Jenkins jobs
  Prerequisites:
    * Install the following packages:
      * ansible
      * python36-virtualenv
      * openssl-devel
      * libyaml-devel
      * krb5-devel
      * krb5-libs
      * openldap-devel
      * git
      * sshpass
      * python-winrm
        * If you are planning on running ansible jobs against Windows RM
  * Make sure that `jenkins` owns the playbook directory and has access to the vault-password file. Do not put that in the root's home dir.

## Setting up Jenkins Job
* Set up a jenkins job with:
  * Playbook path: _full path to playbook_
  * Inventory: file or host list: _fully path to inventory_
    * Or alternatively, you can run it against a list of hosts directly. Make sure to end the line with `,` or the list of hosts will not be accepted.
  * Credential: ssh key from jenkins vault
    * You can set this up under "Credentials"
  * Advanced ->  Additional parameter: `--vault-password-file=/secrets/passwords`
      * I put the vault password file in `/secrets/` directory and found it easier to just use the additional parameter instead of the `ansible vault` field.
