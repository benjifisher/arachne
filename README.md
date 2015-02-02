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
$ ansible-playbook --syntax-check -i ansible/hosts ansible/playbook.yml
$ ansible-playbook -i ansible/hosts -l vagrant --start-at-task="Enable the vhost files." -vvv ansible/playbook.yml
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

Steps 1-4 and 9 are now handled by Ansible.  Soon, I will have to automate some
of the remaining steps.

To install a new site, there are several options.

1. Use `drush make` and a makefile (.info or .yml format) to create a site from scratch.
2. If I already have a codebase (a git repository) then use `drush site-install`:  for example, `$ drush si --account-mail=benji@FisherFam.org --db-url=mysql://200doc:200doc@localhost/200doc --site-mail=benji@FisherFam.org --site-name="200 Days of Code"`
3. If I have a codebase and an existing site, then I can import the database with `drush sql-sync` or `drush sqlc < dump.sql`. I can use `drush rsync` to manage the files directory.

## Bugs

- The `munin-node` service does not restart. Do `$ sudo service munin-node restart` manually in order to see the OPCache graph (in the nginx section, another todo).
- The first time I SSH to the new host, I have to tell openssl to add the server's key to my `known_hosts` file. If that happens while Ansible is doing a `git push`, then the command fails. Run it again and it should be OK.
