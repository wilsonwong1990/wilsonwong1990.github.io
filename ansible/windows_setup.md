## Setting up Windows for ansible

### Links:
* [Enable WinRM](https://www.ansible.com/blog/connecting-to-a-windows-host)
* [Create a vault file for secrets and set up the playbook](https://www.ansible.com/blog/windows-updates-and-ansible)
* [win_updates module](https://docs.ansible.com/ansible/latest/modules/win_updates_module.html)

### TODO
* [x] Create a vault for Windows username and password
* [x] Set up WinRM on a test Windows host
* [x] Test the playbook against the host
* [ ] Set up some sort of script for new Windows hosts

### 10/25/19 Update

The vault creation wasn't too bad. It ended up looking like this:

```
ansible_connection: winrm
ansible_user: [SOMEADMINUSER]
ansible_password: [SECRIT]
ansible_winrm_cert_validation: ignore
slacktoken: [TOKEN]
```

With that done, I moved on to getting winRM enabled on my Windows VMs. First, I logged into each box and ran `Set-Execution Policy Bypass` in PowerShell as an admin since the [script](https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1) provided by Ansible wasn't signed.

Next, I opened up ports `5985/TCP` for WINRM HTTP and `5986/TCP`for WINRM HTTPS on my firewall between my networks per this [Microsoft doc](https://docs.microsoft.com/en-us/windows/win32/winrm/installation-and-configuration-for-windows-remote-management#windows-firewall-and-winrm-20-ports).

With all that out of the way, it was time to put together the playbook. It ended up looking like this:

```
- name: Update packages
  hosts: all
  gather_facts: no
  tasks:
  - name: Get windows vault
    include_vars: /ansible-playbooks/vault/win.yml

  - name: Perform win updates
    win_updates:
      category_names:
      - SecurityUpdates
      - CriticalUpdates
      - UpdateRollups
      reboot: yes
      reboot_timeout: 3600
    register: winoutput

  - name: send to slack
    slack:
      token: "{{ slacktoken }}"
      color: good
      parse: full
      msg: "Host: {{ inventory_hostname }}\nUpdate Results: ```{{ winoutput }}```"
    delegate_to: 127.0.0.1
```
_NOTE:_ The theme is doing some weird formatting and the `msg:` line isn't coming through completely. Take a look at the [md file](https://github.com/wilsonwong1990/wilsonwong1990.github.io/blob/master/ansible/windows_setup.md) directly instead.
A couple of things took trial and error:

* Finding the best way to get an output from the `win_updates` task took me a little while. It helps to use a *debug task* to see what gets outputted to the **winoutput** variable. For example, I put this after the _Perform win updates_ task:

```
  - debug:
      var: winoutput
```

When I ran my task on the commandline, I was able to see what was being stored in **winoutput**. With this, the output is very verbose and would look something like:

```
{u'filtered_updates': {u'f5666069-f9a7-46cf-8375-647f9f201ed2': {u'kb': [], u'title': u'Intel - LAN - Intel(R) Ethernet Connection I217-LM', u'filtered_reason': u'category_names', u'installed': False, u'id': u'f5666069-f9a7-46cf-8375-647f9f201ed2', u'categories': [u'Drivers', u'Windows Server 2012 R2  and later drivers']}, u'a73b4728-01aa-47e6-bfd8-9d4d780d68bb': {u'kb': [], u'title': u'Intel - Other hardware - Intel(R) C226 Series Server Advanced SKU LPC Controller - 8C56', u'filtered_reason': u'category_names', u'installed': False, u'id': u'a73b4728-01aa-47e6-bfd8-9d4d780d68bb', u'categories': [u'Drivers', u'Windows Server Drivers']}, u'a50d8dc2-c5b0-4c21-aef5-c13a58fc4ff0': {u'kb': [], u'title': u'Intel - Other hardware - Intel(R) Xeon(R) processor E3 - 1200 v3/4th Gen Core processor DRAM Controller - 0C00', u'filtered_reason': u'category_names', u'installed': False, u'id': u'a50d8dc2-c5b0-4c21-aef5-c13a58fc4ff0', u'categories': [u'Drivers', u'Windows Server 2016 and Later Servicing Drivers']}, u'029539b5-2dd6-41f8-b3b1-9b9a5f4309d0': {u'kb': [u'4519979'], u'title': u'2019-10 Cumulative Update for Windows Server 2016 for x64-based Systems (KB4519979)', u'filtered_reason': u'category_names', u'installed': False, u'id': u'029539b5-2dd6-41f8-b3b1-9b9a5f4309d0', u'categories': [u'Updates', u'Windows Server 2016']}, u'f9a7ad7f-8be9-42b9-9aef-a78187208ff0': {u'kb': [], u'title': u'Intel - Other hardware - Intel(R) 8 Series/C220 Series PCI Express Root Port #1 - 8C10', u'filtered_reason': u'category_names', u'installed': False, u'id': u'f9a7ad7f-8be9-42b9-9aef-a78187208ff0', u'categories': [u'Drivers', u'Windows Server 2016 and Later Servicing Drivers']}, u'64218d22-ff91-4027-92f6-5d8c82c8b686': {u'kb': [], u'title': u'Intel - Other hardware - Intel(R) 8 Series/C220 Series PCI Express Root Port #2 - 8C12', u'filtered_reason': u'category_names', u'installed': False, u'id': u'64218d22-ff91-4027-92f6-5d8c82c8b686', u'categories': [u'Drivers', u'Windows Server 2016 and Later Servicing Drivers']}}, u'installed_update_count': 0, u'changed': False, u'reboot_required': False, 'failed': False, u'updates': {}, u'found_update_count': 0}
```

Which is a bit hard on the eyes, but I figured since I'm sending it to slack to notify me, I rather have more details than less.

* Slack originally would fail out and not send a notification because it was trying to use python on the Windows box to send it. This was due to Windows not having python installed and not being in the `usr/bin/python` path anyways. What tipped me off was the error and this [Ansible GitHub repo issue comment](https://github.com/ansible/ansible/issues/51604#issuecomment-462724706). By setting it to `delegate_to:localhost`, the slack notification comes from the Ansible host instead.

With that, slack notifications are rolling in after Windows updates and I'm ready to move on figuring out how to use Jenkins to run my Ansible playbooks on a schedule.
