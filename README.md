# README.md

This is a set of Ansible playbooks that configure a new server. 

The original idea came from Ryan Eschinger's gist at [https://gist.github.com/ryane/e0ea8e4a75b140bf799f](https://gist.github.com/ryane/e0ea8e4a75b140bf799f) 
Regrettably, that description seems to have been broken by non-backward-compatible changes to Ansible.

The inventory.ini file in this repo lists the hosts that these commands should apply to.

## Test connectivity

A test to see that this Ansible control computer can ping all the hosts in the inventory:

```
ansible all --inventory-file=inventory.ini --module-name ping -u deploy

```

## Run a command

Use `-i` in place of *--inventory-file*; `-m` in place of *--module-name*;  `command` is the default module; `-a` in place of *-args*;

```
ansible all -i "inventory.ini" -m command -a 'uptime' -u deploy

```

## Set Up Accounts 

Run this playbook only once on a server. 
It creates and sets up the `deploy` user with proper sudo access, 
forces ssh public key logins (no passwords),
and disables root logins totally. 

Since it doesn't rely on public keys for the initial login, 
you need to add additional arguments (`--ask-become-pass -k`)
to the command line. 

```
ansible-playbook -i "hostname," setup_accounts.yml --ask-become-pass -k
```

## Set Up Packages

This playbook uses the `deploy` user to set up the remainder of the basic packages needed by a VPS.

```
ansible-playbook -i "atl3.richb-hanover.com," setup_packages.yml  --ask-become-pass -u deploy
```

## Cool things about Ansible

* Ansible output from includes a `changed=` flag. If it's `1`, something changed; if it's `0`, nothing changed.
* Ansible invokes the playbook on the hosts specified by an  "inventory file" with `-i filename`. 
To invoke Ansible on a few devices, use `-i "comma-separated-list"`. 
For a single host, use `i "hostname,"` with a trailing comma.
* `ansible all -m setup -i "atl3.richb-hanover.com,"` displays all the "built-in" variables for the target.
* Note: atl.richb-hanover is stuck at 16.04 with old python - gives "Deprecated" warning. 
The `./ansible.cfg ` file has a `deprecation_warnings=False` to suppress this warning.
