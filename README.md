# Provisioning my web server

## Overview

This repository is not designed to be flexible enough to be adapted by others,
but feel free to have a look at how I am setting things up.

Most of the work is done by several of [Jeff
Geerling](https://github.com/geerlingguy)'s Ansible roles. I have renamed the
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
- Munin

Later, I hope to add

- Memcache or Redis
- Varnish

and perhaps experiment with others such as

- Nginx
- PHP Farm
- Docker

## Directory Structure

I am still not sure this is a good idea, but I decided not to put my sites
under `/var/www/`.  The point is that I want to keep some things close to my
sites that are not web-accessible, such as private files, plain-text notes,
and so on.  It seems simpler, perhaps safer, not to change the access for the
standard `/var/www`.

For each site `foo`, I create the following directories:

- `/opt/drupal/files/foo`:  (coming soon) docroot for `files.foo.org`
- `/opt/drupal/repos/foo.git`:  a bare repository, so that I can push from
  local
- `/opt/drupal/sites/foo`:  a full repository; pull from the bare one, and set
  `.../foo/drupal` as the document root.

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

## Batteries Not Included

Before running Ansible, I need to create the file `ansible/vars/passwords.yml`
that looks something like this:
```
# This file should not be under version control.
---
munin_admin_password: XXXXXXXX
mysql_root_password: XXXXXXXX

mysql_users:
  - {name: 3mile, host: localhost, password: XXXXXXXX, priv: "3mile.*:ALL"}
  - {name: amcbostondev, host: localhost, password: XXXXXXXX, priv: "amcbostondev.*:ALL"}
```
(Note to self: without the quotation marks, there is a syntax error because of
the colon!)

## Managing sites

I will also use git to keep track of my apache config files.  I want to
customize them, so I do not plan to generate them from templates.  I will keep
them under version comtrol, but I have not yet decided whether to keep them
inside this repository or to create a separate one. I guess the question is
whether I am willing to share them on GitHub.

I will keep my Drupal sites in separate repositories, using git to sync them
up to the production server.  Perhaps I will eventually use Ansible for that
as well:  after updating the repository locally, I can push to the bare repo
and then have Ansible do a "git pull" and "drush updatedb; drush cc all"
without logging in to the server.

Before visiting a site, I need to do the following:

1. Copy a vhost file into `/etc/apache2/sites-available`.
2. Enable the site with `sudo a2ensite`.
3. Push the code from the local repo to the bare one.
4. Pull the code from the bare repo to the site directory.
5. Create the settings file (whether or not we use the Drupal install).
6. Create the files directory.
7. Import the database, or install Drupal from scratch.
8. Copy files to the server.
9. (Vagrant only): edit /etc/hosts on the VM so that the site can eat its own
   output. Or modify settings.php.  See https://www.drupal.org/node/811406

Soon, I will have to automate some of these steps.
