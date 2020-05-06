# README.md

These [Ansible playbooks](https://www.ansible.com/resources/get-started)
configure a new server with default accounts and an initial set of packages.
The playbooks set up a `deploy` account that has sudo ability, and then rely on that account (alone) for further configuration.
This is best practice for managing a fleet (or even a couple) of computers that you want to keep updated and synchronized.

Ansible runs on your "control computer" (your laptop/desktop) and configures one or more "target computers".
No special software need be installed on the target:
Ansible connects via SSH and invokes commands on the target. 
This makes it convenient to apply the same configuration to multiple computers in the "inventory".

The original idea for these "new VPS" scripts came from Ryan Eschinger's gist at [https://gist.github.com/ryane/e0ea8e4a75b140bf799f](https://gist.github.com/ryane/e0ea8e4a75b140bf799f) 
Regrettably, that playbook seems to have been broken by non-backward-compatible changes to Ansible.


## Install Ansible

The [Ansible Get Started](https://www.ansible.com/resources/get-started) page
tells how to install Ansible on your laptop/desktop (the control computer).
There are installers for Linux, Windows, macOS.

## Set Up Accounts on Target Server

Run this playbook (on your control computer) *one time* for every new VPS server. 
It directs the target computer(s) to create a `deploy` user, 
set up proper sudo access with your public key, 
force ssh public key logins (no passwords permitted),
and disable root logins. 
Since it doesn't rely on public keys for this initial login, you will be prompted for the root password (this one time only). 
You will also be prompted to enter a *sudo* password to be used with the `deploy` account.

*Note:* If you see an error message mentioning "host fingerprint", you should `ssh root@hostname` one time. 
You don't even need to log in - just make the SSH connection and accept the fingerprint for the target machine. 
Then re-run the command below:

```
ansible-playbook -i "hostname," setup_accounts.yml -k
```

## Set Up Packages on Target Server

This playbook directs the `deploy` user to set up the remainder of the basic packages needed by a VPS.
Currently, the following packages are installed/configured:

* ufw
* fail2ban
* unattended-upgrades
* logwatch
* mosh
* vim
* Postfix (to relay mail)
* cron job to send daily summary of activity

```
ansible-playbook -i "hostname," setup_packages.yml --ask-become-pass -u deploy
```

## Test connectivity

After installing Ansible on your control computer, you can test it with computers in your inventory.
Run the following command on your control computer to ping all the hosts named in the inventory file:

```
ansible all --inventory-file=inventory.ini --module-name ping -u deploy

```
Note: The command above requires that you already set up a `deploy` user. (See the **Set Up Accounts on Target Server** script above.)

## Run uptime command

Use `-i` in place of *--inventory-file*; `-m` in place of *--module-name*;  `command` is the default module; `-a` in place of *-args*;

```
ansible all -i "inventory.ini" -m command -a 'uptime' -u deploy

```

## Cool things to know about Ansible

As I played around to get these playbooks working, I learned the following. I put them here so that I'll remember...

* Ansible playbooks are designed to be run and then re-run without causing problems. 
This makes it convenient to refine configuration playbooks and then to re-run them on the entire fleet of servers. 
* Ansible output from includes a `changed=` flag. If it's `1`, something changed; if it's `0`, nothing changed.
Color of the output also show success, change, or failure.
* Ansible invokes the playbook on the hosts specified by an  "inventory file" with `-i filename`. 
To invoke Ansible on a few devices, use `-i "comma-separated-list"`. 
For a single host, use `i "hostname,"` with a trailing comma.
* `ansible all -m setup -i "hostname,"` displays all the "built-in" variables for the target.
* One of my VPS's is stuck at 16.04 with old python, and gives a "Deprecated" warning. 
The `./ansible.cfg ` file has a `deprecation_warnings=False` to suppress this warning.
* To discover Ansible variables from the target, add `gather_facts: yes` to the playbook.
This adds considerably to the time to run the playbook, so you may choose to leave it when those facts are not required.
* These scripts do arithmetic/text processing on variables, as described in this
[Stack Overflow](https://stackoverflow.com/questions/33505521/how-to-use-arithmetic-when-setting-a-variable-value-in-ansible) article.
