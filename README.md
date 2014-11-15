# Provisioning my web server

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

This repository also contains a Vagrantfile, which I will use for testing.  I
will have to be careful not to configure anything insecurely while working on
the local VM.

I will also use git to keep track of my apache config files.  I do not plan to
install them using Ansible, and I have not yet decided whether to keep them
inside this repository or to create a separate one.

I will keep my Drupal sites in separate repositories, using git to sync them
up to the production server.  Perhaps I will eventually use Ansible for that
as well:  after updating the repository locally, I can push to the bare repo
and then have Ansible do a "git pull" and "drush updatedb; drush cc all"
without logging in to the server.
