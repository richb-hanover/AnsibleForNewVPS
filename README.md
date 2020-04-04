# README.md

This is a set of Ansible playbooks that configure a new server. 

The original idea came from Ryan Eschinger's gist at [https://gist.github.com/ryane/e0ea8e4a75b140bf799f](https://gist.github.com/ryane/e0ea8e4a75b140bf799f)

The inventory.ini file in the repo lists the hosts that these commands should apply to.

## Test connectivity

```
# First check to see that this site can ping all the inventory hosts
ansible all --inventory-file=inventory.ini --module-name ping -u deploy

```

*Note: atl.richb-hanover is stuck at 16.04 with old python - gives "Deprecated" warning. It's OK.*


## Run a command

Use `-i` in place of *--inventory-file*; `-m` in place of *--module-name*;  `command` is the default module; `-a` in place of *-args*;

```
ansible all -i "inventory.ini" -m command -a 'uptime' -u deploy

```

## Server First Time Playbook 

Run this playbook only once on a server. 
It creates and sets up the `deploy` user with proper sudo access, 
forces ssh public key logins (no passwords),
and disables root logins totally. 

Since it doesn't rely on public keys for the initial login, 
you need to add additional arguments (`--ask-become-pass -k`)
to the command line. 

```
ansible-playbook -i "hostname," server_first_time.yml --ask-become-pass -k
```

## Five-Minute script for new VPS's

This playbook uses the `deploy` user to perform the remainder of the setups:

** *Provisional command...* **

```
ansible-playbook -i "atl3.richb-hanover.com," setup_server.yml  --ask-become-pass  -u deploy
```

## Cool things about Ansible

* Ansible output from includes a `changed=` flag. If it's `1`, something changed; if it's `0`, nothing changed.
* Ansible invokes the playbook on the hosts specified by an  "inventory file" with `-i filename`. To invoke Ansible on a few devices, use `-i "comma-separated-list"`. For a single host, use `i "hostname,"` with a trailing comma.