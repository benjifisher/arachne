# Provisioning my web server

## Overview

This repository is not designed to be flexible enough to be adapted by others,
but feel free to have a look at how I am setting things up.

Most of the work is done by several of [Jeff
Geerlin](https://github.com/geerlingguy)'s Ansible roles. I have renamed the
roles and edited the dependencies, but not tried to hide the origin.  I have
made a few tweaks, and if I think any of these are generally useful, then I
will send Jeff a pull request.

I plan to provision my new web server manually, setting up a few different
partitions:

- /var/log
- /var/lib/mysql
- /opt/drupal
- maybe others

The ansible directory contains everything needed to install the LAMP stack on
the server:

- Apache
- MySQL
- PHP

as well as

- Postfix
- Munin (coming soon)

Later, I hope to add

- Memcache or Redis
- Varnish

and perhaps experiment with others such as

- Nginx
- PHP Farm
- Docker

## Testing with Vagrant

This repository also contains a Vagrantfile, which I will use for testing.  I
will have to be careful not to configure anything insecurely while working on
the local VM.

With the included inventory file, the following commands work on the Vagrant
VM:

```
$ ansible vagrant -i ansible/hosts -m ping
$ ansible-playbook -i ansible/hosts -l vagrant ansible/playbook.yml
```

## Managing sites

I will also use git to keep track of my apache config files.  I do not plan to
install them using Ansible, and I have not yet decided whether to keep them
inside this repository or to create a separate one.

I will keep my Drupal sites in separate repositories, using git to sync them
up to the production server.  Perhaps I will eventually use Ansible for that
as well:  after updating the repository locally, I can push to the bare repo
and then have Ansible do a "git pull" and "drush updatedb; drush cc all"
without logging in to the server.
