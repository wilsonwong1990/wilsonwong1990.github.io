---
layout: default
---

# Some notes and thoughts on ansible build out
I'm finally taking the time to build up an ansible server. I had a few ideas for this project:

* [x] Centos/Ubuntu/Windows specific playbooks to update the servers, send a slack message to #update-notification when updates are ran, reboot the server, and then send a final slack message notifying that updates were done.
* [x] Utilize ssh-keys to login into all my servers for ansible to run scripts
* [x] Start using ansible vault to store my secrets.

## TODO List:
* [ ] Setup [Jenkins](./jenkins/notes.md) to  run playbooks
* [ ] Setup a second Ansible server (maybe AWX?) to update this ansible server.

## Update 10/23/19

Since majority of my servers are CentOS 7 based, I started there. I was able to quickly create an ansible playbook to copy an ansible specific ssh key to all a majority of my servers. This prevents me from having to save the password somewhere plus now I can rotate passwords with an ssh key. Luckily, ansible has a [module](https://docs.ansible.com/ansible/latest/modules/authorized_key_module.html) for exactly this so that was quick to put together.

Next I wanted to do a [yum update](https://docs.ansible.com/ansible/latest/modules/yum_module.html?highlight=yum) playbook that [reboots](https://docs.ansible.com/ansible/latest/modules/reboot_module.html?highlight=reboot) then sends a [slack notifications](https://docs.ansible.com/ansible/latest/modules/slack_module.html?highlight=slack). Since the slack notification requires a webhook token, I also took advantage of ansible vault to store that. I should just take advantage of [include_vars](https://docs.ansible.com/ansible/latest/modules/include_vars_module.html?highlight=include_vars) in general.

Everything is committed to my private ansible-playbook repo.

## Update 10/25/19

Took a while but the [Windows based](./ansible/windows_setup.md) playbook is ready to roll out. I was going to do an Ubuntu/Debian based one but I have zero Debian hosts at the moment so I'll edit the Yum playbook when the time comes to use apt.
