## What is ansible-phpfpm? [![Build Status](https://secure.travis-ci.org/nickjj/ansible-phpfpm.png)](http://travis-ci.org/nickjj/ansible-phpfpm)

It is an [Ansible](http://www.ansible.com/home) role to install and configure
php-fpm.

##### Supported platforms:

- Ubuntu 16.04 LTS (Xenial)
- Debian 8 (Jessie)

### What problem does it solve and why is it useful?

I'll be honest with you, I don't really use PHP but a project I'm working on
requires deploying an app made with it. This role fills the gap on being able
to quickly get php-fpm running and configured as per your application's requirements.

This role will not set up Apache. It is expected you proxy your application
with nginx.

## Role variables

Below is a list of default values along with a description of what they do.

```
---

# Which version of php-fpm and php-cli do you want to install?
phpfpm_version: '7.0'

# Default PHP support packages that will be installed by default.
# phpX-fpm and phpX-cli are both included regardless of this list.
phpfpm_default_support_packages:
  - 'php7.0-curl'
  - 'php7.0-mysql'

# Additional PHP support packages that you may want to install.
phpfpm_support_packages: []

# Custom settings that override the php.ini settings. An example on adding
# a few custom multi-line commands is provided later on in this README.
phpfpm_php_ini_settings: ''

# Default settings for all of the php-fpm.conf values.
phpfpm_fpm_config_error_log: '/var/log/php7.0-fpm.log'
phpfpm_fpm_config_syslog_facility: 'daemon'
phpfpm_fpm_config_syslog_ident: 'php-fpm'
phpfpm_fpm_config_log_level: 'notice'
phpfpm_fpm_config_emergency_restart_threshold: 0
phpfpm_fpm_config_emergency_restart_interval: 0
phpfpm_fpm_config_process_control_timeout: 0
phpfpm_fpm_config_process_max: 0
phpfpm_fpm_config_daemonize: 'yes'
phpfpm_fpm_config_rlimit_files: 1024
phpfpm_fpm_config_rlimit_core: 0
phpfpm_fpm_config_events_mechanism: 'epoll'
phpfpm_fpm_config_systemd_interval: 10

# Create your own FPM pools, at least one needs to exist. Optionally you may
# add additional pools, just define them all in this variable.
phpfpm_fpm_pools: |
  [www]
  user = www-data
  listen = /var/run/php7.0-fpm.sock
  listen.owner = www-data
  pm = dynamic
  pm.max_children = 5
  pm.start_servers = 2
  pm.min_spare_servers = 1
  pm.max_spare_servers = 3

# The APT GPG key id used to sign the PHP package.
phpfpm_apt_keys:
  Debian:
    id: 'DF3D585DB8F0EB658690A554AC0E47584A7A714D'
  Ubuntu:
    id: '14AA40EC0831756756D7F66C4F4EA0AAE5267A6C'

# The OS distribution and distribution release, thanks https://github.com/debops.
# Doing it this way doesn't depend on having lsb_release installed.
phpfpm_distribution: '{{ ansible_local.core.distribution
                         if (ansible_local|d() and ansible_local.core|d() and
                             ansible_local.core.distribution|d())
                         else ansible_distribution }}'
phpfpm_release: '{{ ansible_local.core.distribution_release
                                 if (ansible_local|d() and ansible_local.core|d() and
                                     ansible_local.core.distribution_release|d())
                                 else ansible_distribution_release }}'

# Address of the PHP repository.
phpfpm_repositories:
  Debian: 'deb https://packages.sury.org/php/ {{ ansible_distribution_release }} main'
  Ubuntu: 'ppa:ondrej/'
```

## Example playbook

For the sake of this example let's assume you have a group called **app** and
you have a typical `site.yml` file.

To use this role edit your `site.yml` file to look something like this:

```
---

- name: Configure app server(s)
  hosts: app
  become: True

  roles:
    - { role: nickjj.phpfpm, tags: phpfpm }
```

Let's say you want to configure a few php.ini settings and define your own 2nd
FPM pool, start by opening or creating `group_vars/app.yml` which is located
relative to your `inventory` directory and then making it look like this:

```
---

phpfpm_php_ini_settings: |
  memory_limit = 256M
  max_execution_time = 60

phpfpm_fpm_pools: |
  [www]
  user = www-data
  listen = /var/run/php7.0-fpm.sock
  listen.owner = www-data
  pm = dynamic
  pm.max_children = 5
  pm.start_servers = 2
  pm.min_spare_servers = 1
  pm.max_spare_servers = 3

  [auth]
  user = www-data
  listen = /var/run/php7.0-fpm-auth.sock
  listen.owner = www-data
  pm = dynamic
  pm.max_children = 5
  pm.start_servers = 2
  pm.min_spare_servers = 1
  pm.max_spare_servers = 3
```

## Installation

`$ ansible-galaxy install nickjj.phpfpm`

## Ansible Galaxy

You can find it on the official
[Ansible Galaxy](https://galaxy.ansible.com/nickjj/phpfpm/) if you want to
rate it.

## License

MIT
