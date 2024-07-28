# Ansible Playbook Examples

A collection of minimalist Ansible playbooks for automating server setups.

## Playbook Structure

The playbooks contained in this repository were created for educational purposes, and should serve as a base for you to create your own playbooks and roles.

Although we opt to not use roles, our playbooks follow a distinctive structure to facilitate reuse while keeping them mostly self-contained and straightforward.

For instance, this is how the `lemp` playbook is structured:

```
lemp_ubuntu1804
├── files
│   ├── info.php.j2
│   └── nginx.conf.j2
├── vars
│   └── default.yml
├── playbook.yml
└── readme.md
```


- `files/`: directory containing templates and other files required by the playbook.
- `vars/`: directory to save variable files. A `default.yml` var file is included by default.
- `playbook.yml`: the playbook file.
- `readme.md`: instructions and links related to this playbook.

### Connection Test

From your local machine or Ansible control node, run:

```command
ansible all -m ping -u remote_user
```

If you're able to get a "pong" reply back from your node(s), your setup works as expected and you'll be able to run both ad-hoc commands and playbooks on your nodes, using Ansible.

