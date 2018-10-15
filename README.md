ansible-roles
=============

.ssh/config
-----------
avoid host key checking on test machines.

Vagrantfile
-----------
fires up three vmachines used in testing.

roles
======
each role has same host configuration file to start with.

mariadb
-------
'configure first' and install to mariadb server 10.x. 

nodejs
------
install and configure nodejs with selected npm packages on Debian servers.

wkhtmltox
---------
install and configure whtmltopdf on Debian servers using .deb or .tar packages
with specific versions when needed (especially with odoo)
