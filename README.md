# Ansible Playbook Examples

A collection of minimalist Ansible playbooks for automating server setups.

## Playbook Structure

The playbooks contained in this repository were created for educational purposes, and should serve as a base for you to create your own playbooks and roles.

Although we opt to not use roles, our playbooks follow a distinctive structure to facilitate reuse while keeping them mostly self-contained and straightforward.

For instance, this is how the `lemp` playbook is structured:

```
lemp_ubuntu2204
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

## Windows Subsystem for Linux

To test examples from this repository in Windows Subsystem for Linux and hosts based on virtual machines, please do the next steps:

1. Install Windows Subsystem for Linux (WSL) as described in official [documentation](https://learn.microsoft.com/en-us/windows/wsl/install) and install Ubuntu under WSL.

2. Install VirtualBox using th official download [source](https://www.virtualbox.org/wiki/Downloads).

3. In Ubuntu under WSL, install Vagrant

```bash
wget -O - https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install vagrant
vagrant plugin install virtualbox_WSL2
```

4. Add to `~/.bashrc` in your Ubuntu under WSL the next lines ath the end of the file. Do not forget to change `<YOUR_USERNAME>` to your real Windows user files folder name.

```
# Setup Vagrant variables
export VAGRANT_WSL_ENABLE_WINDOWS_ACCESS="1"
export VAGRANT_WSL_WINDOWS_ACCESS_USER_HOME_PATH="/mnt/c/Users/<YOUR_USERNAME>/"
export PATH="$PATH:/mnt/c/Program Files/VirtualBox"
```

5. Create a new virtual machine using Vagrant. To do so, prepare a file `Vagrantfile` with the next configuration in your Ubuntu WSL session:

```
Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/jammy64"
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.network "forwarded_port", guest: 80, host: 8888

  config.vm.provider "virtualbox" do |vb|
     vb.gui = false
     vb.customize ['modifyvm', :id, "--uart1", "off"]
     vb.customize ['modifyvm', :id, "--uart2", "off"]
     vb.customize ['modifyvm', :id, "--uart3", "off"]
     vb.customize ['modifyvm', :id, "--uart4", "off"]
  end
end
```

Run the virtual machine:

```bash
vagrant up
```

To get ssh configuration for the virtual machine:

```bash
vagrant ssh-config
```

6. Install Ansible in your WSL Ubuntu:

```bash
sudo apt update
sudo apt install software-properties-common
sudo add-apt-repository --yes --update ppa:ansible/ansible
sudo apt install ansible
```

7. Edit a system inventory file `/etc/ansible/hosts` adding there the next lines. Do not forget to add relevant values from SSH configuration data given by Vagrant earlier.

```
[myhosts]
myhost ansible_host=<IP address> ansible_port=<Port> ansible_user=vagrant ansible_ssh_private_key_file=<Path to ssh private key>
```
