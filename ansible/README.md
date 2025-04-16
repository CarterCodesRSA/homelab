# Ansible
The basis of all things ansible.

## Installation
```
brew install ansible
```

## Running ansible
NOTE: You will need access to all servers via ssh in order to run any ansible commands.

This will run the homelab inventory, calling the ping module, and using the ubuntu user.
```bash
ansible -i inventory homelab -m ping -u ubuntu
```

# Important commands
If you want to check available memory on all hosts, you can use the "arguments" flag `-a`.
```bash
ansible -i inventory homelab -m shell -a "free -m" -u ubuntu
```

# ansible.cfg
This file is used to overwrite the defaults used on ansible. Most useful is the ability to always define the desired inventory file.
```
[defaults]
INVENTORY = inventory
```
This means that any Ansible command you run will automatically use the inventory file described above.
