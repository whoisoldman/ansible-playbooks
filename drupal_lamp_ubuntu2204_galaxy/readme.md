# Drupal on Ubuntu 22.04 LAMP

This playbook will install a Drupal website on top of a LAMP environment (**L**inux, **A**pache, **M**ySQL and **P**HP) on an Ubuntu 22.04 machine. Drupal will be configured with the options specified in the `vars/default.yml` variable file.

## Settings

- `drupal_install_site`:  Installs Drupal site.
- `drupal_build_composer_project`:  Uses Composer's create-project as a site deployment strategy.
- `drupal_composer_install_dir`:  Drupal Composer install directory.
- `drupal_core_path`:  Drupal site directory.
- `drupal_domain`: Your domain name.
- `drupal_db_user`: The name of the MySQL user that should be created for Drupal.
- `drupal_db_password`: The password for the new MySQL user.
- `drupal_db_name`: The name of the MySQL database that should be created for Drupal.
- `drupal_version`: Version of Drupal.
- `drush_launcher_version`: Version of Drush Launcher.
- `drupal_composer_project_package`: Details of Drupal Package for Composer.
- `drupal_composer_project_options`: Options of Drupal Package for Composer.
- `apache_vhosts`: Details of an vhost for the Drupal site.
- `apache_remove_default_vhost`: Deletes default virtual host.
- `mysql_databases`: Details for the MySQL database.
- `mysql_users`: Details for the MySQL user.
- `mysql_root_password`: The desired password for the **root** MySQL account.
- `php_version`:  Version of PHP.
- `php_enable_php_fpm`:  Use php-fpm.
- `php_packages_extra`:  Packages for PHP.

## Running this Playbook

Quickstart guide for those already familiar with Ansible:

### 1. Obtain the playbook
```shell
git clone https://github.com/Bumeranghc/ansible-playbooks.git
cd ansible-playbooks/drupal_lamp_ubuntu2204_galaxy
```

### 2. Customize Options

```shell
nano vars/default.yml
```

```yml
---
#System Settings
php_modules: [ 'php-curl', 'php-gd', 'php-mbstring', 'php-xml', 'php-xmlrpc', 'php-soap', 'php-intl', 'php-zip' ]

#MySQL Settings
mysql_root_password: "mysql_root_password"
mysql_db: "wordpress"
mysql_user: "sammy"
mysql_password: "password"

#HTTP Settings
http_host: "your_domain"
http_conf: "your_domain.conf"
http_port: "80"
```

### 3. Run the Playbook

```command
ansible-playbook -l [target] -i [inventory file] -u [remote user] playbook.yml
```
