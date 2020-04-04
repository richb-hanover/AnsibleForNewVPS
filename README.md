# README.md

The five_minute.yml file comes from Ryan Eschinger's gist at:

https://gist.github.com/ryane/e0ea8e4a75b140bf799f

The inventory.ini file lists the files that this should apply to:

Test commands:

```
# First check to see that this site can ping all the inventory hosts
ansible all --inventory-file=inventory.ini --module-name ping -u deploy

```