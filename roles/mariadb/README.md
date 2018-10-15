MariaDB Server
============

This roles helps to install MariaDB Server across Debian variants.
Apart from installing the MariaDB Server, it applies basic hardening, like
securing the root account with password, and removing test databases. 

The role can also be used to add databases to the MariaDB Server and create 
users in the database. It also supports configuring the databases for 
replication --both master and slave can be configured via this role.

Requirements
------------

This role requires Ansible 2.4 or higher, and platform requirements are listed
in the metadata file.

Role Variables
--------------

The variables that can be passed to this role and a brief description about
them are as follows:

    # version to be installed
    mariadb_version: '10.2' # 10.1, 10.2, 10.3

    # root user/pass
    mariadb_root_user: 'root'
    mariadb_root_password: 'root'

    # when defining users in replication make sure you have only one user tagged as master: true
    # this configuration allows you to change replication user if needed
    mariadb_users: {}
      login:
      - name: user
        password: pass
        state: present|absent
      replication:
      - name: user
        password: pass
        state: present|absent 
      - name: user_new
        password: pass_new
        master: true


    # list of databases to be added/dropped
    # replicate: yes, will be listed in: binlog_do_db
    mariadb_databases: {}
    - name: test_01
      replicate: no
      init_script: files/test_01-init.sql
      state: present

    # server-id is auto generated from assigned IP address on host.
    # any modifications here will be ignored
    mariadb_server_id: 1 # default: 1

    # set mariadb server role
    mariadb_replication_role: master|slave

    # group name of hosts defined in inventory to obtain necessary information
    mariadb_replication_master: hostname

    # connections
    mariadb_host: '127.0.0.1'
    mariadb_port: '3306'
    mariadb_socket: '/run/mysqld/mysqld.sock'

    # Enter the IP address of server you want MariaDB to monitor for public connection
    mariadb_bind_address: '0.0.0.0' # allow from all IPs.

    # mariadb configuration sections as dictionary. default includes minimum configuration
    # all configuration parameters defined here will be merged with `_defaults`
    # some parameters requires duplicating in configuration file. by default binlog_do_db
    # and binlog_ignore_db in list which can be extented via `mariadb_configuration_multiple_params`.
    # keywords added to list will be merged with `_defaults`.
    # enter configuration parameters here as shown.
    # all configurations will be written to /etc/mysql/my.cnf. all configurations goes in single file.
    # see: https://mariadb.com/kb/en/library/server-system-variables

    mariadb_configuration:
      client:
        host: "{{ mariadb_host }}"
        port: "mariadb_port"
      
      client_server: {}
      mysql: {}
      mysqld:
        log-error: "/var/log/mysql/mariadb-error.log"
        binlog_do_db: [
          "test", "test12", "test3"
        ]      

Examples
--------

1) Install MariaDB Server and set the root password, but don't create 
   any database or users.

- hosts: all
  roles:
  - role: mariadb
    mariadb_root_password: root

2) Install MariaDB Server and create 2 databases, one database with init-script
   and 2 users

---
- hosts: all
  roles:
  - role: mariadb
    mariadb_root_password: root
    mariadb_databases:
    - name: test_01
      init_script: files/test_01-init.sql
    - name: test_02
    mariadb_users:
    login:
    - name: test_01
      password: test_01
      priv: "*.*:ALL"
    - name: test_02
      password: test_02

Note: If users are specified and password/privileges are not specified, then
default values are set.

3) Install MySQL Server and create 2 databases and 2 users and configure the
   database as replication master with one database configured for replication.

---
- hosts: all
  roles:
  - role: mariadb
    mariadb_root_password: root
    mariadb_databases:
    - name: test_01
      replicate: yes
      init_script: files/test_01-init.sql
    - name: test_02
      replicate: no
    mariadb_users:
    login:
    - name: test_01
      password: test_01
      priv: "*.*:ALL"
    - name: test_02
      password: test_02
    replication:
    - name: repl_01
      password: repl_01
      master: true

4) A fully installed/configured MySQL Server with master and slave
replication.

---
- hosts: test_mariadb_master
  roles:
  - role: mariadb
    # mariadb_port: 3306
    mariadb_replication_role: master
    # mariadb_bind_address: "{{ groups['test_mariadb_master'][0] }}"
    mariadb_databases:
    - name: test_01
      state: absent
    - name: test_02
      state: absent
    - name: test_01
      replicate: yes
      init_script: files/test_01-init.sql
      state: present
    - name: test_02
      replicate: no
      init_script: files/test_02-init.sql
      state: present
    mariadb_users:
      login:
      - name: test_01
        password: test_01
      replication:
      - name: test_01
        password: test_01
        master: true
    mariadb_configuration: # master
      mysqld:
        log_basename: master
        binlog_format: row
        log_bin: /var/log/mysql/mariadb-bin.log
        log_bin_index: /var/log/mysql/mariadb-bin.index
        binlog_do_db: [ "test_01" ]
        binlog_ignore_db: [ "test_02" ]

- hosts: test_mariadb_slave
  roles:
  - role: mariadb
    # mariadb_port: 3306
    mariadb_replication_role: slave
    mariadb_replication_master: test_mariadb_master
    mariadb_replication_master_ip: "{{ groups[mariadb_replication_master][0] }}"
    # mariadb_replication_master_ip: {{ hostvars[groups[mariadb_replication_master][0]].ansible_default_ipv4.address }}
    # mariadb_replication_master_port: 3306
    mariadb_databases:
    - name: test_01
      state: absent
    - name: test_02
      state: absent
    - name: test_01
      init_script: files/test_01-init.sql
      state: present
    - name: test_02
      init_script: files/test_02-init.sql
      state: present
    mariadb_users:
      login:
      - name: test_01
        password: test_01
      - name: test_01
        password: test_01
      replication:
      - name: test_01
        password: test_01
        master: true
    mariadb_configuration: # slave
      mysqld:
        read_only:
        relay-log: /var/log/mysql/mariadb-relay-bin
        relay-log-index: /var/log/mysql/mariadb-relay-bin.index


Note: When configuring the full replication please make sure the master is
configured via this role and the master is available in inventory and facts
have been gathered for master. The replication tasks assume the database is
new and has no data.

For turning existing databases into master-slave mode, you need to setup 
your master server first then take sql dump of desired database(s) and 
import them on slaves. during dump/import, make sure you have locks. 
don't forget to unlock!

Dependencies
------------
none

License
-------
BSD

Author Information
------------------
Fatih Piristine
