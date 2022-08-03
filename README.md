Ansible Role: Guacamole Client
=========

This Ansible role will configure Apache's Guacamole Client on Debian 11 complete with MariaDB and LDAP auth plugins to allow an "exploded" deployment.

The Guacamole Client is a Tomcat application that connects to the Guacamole Server (`guacd`) and mysql for storing connection profiles (along with your favourite auth and 2FA). `guacd` is the component that then connects to your target RDP server. Note this role enables the guacamole client Tomcat to listen on :8080 on all interfaces, so having a reverse proxy and firewall protect this port is essential. We default to configuring guacamole to be proxied anyway.

A quick flow of how a typical "exploded" Guacamole deployment works:

User(Browser) -> haproxy:443 -> Tomcat(Guacamole war):8080 -> guacd:4822 -> RDP/SSH/Telnet connections :3389/:22/:23

Each of these are seperate machines, with a firewall in the way, allowing only the ports listed above. Guacamole client will also need :3306 access to mysql for example too. `guacd` doesn't need mysql access, it gets told what to do by the guacamole client only.

If you use the sol1-guacamole-mysql role (almost certainly need to!), that role will create a default user for this application of **guacadmin:guacadmin**, so immediately login and change the password for this account and store it in your favourite break-glass password manager. You may need this in case your other authentication methods don't work first time.

IMPORTANT: This role makes the guacamole war file the ROOT of your tomcat installation, meaning you can use / instead of /guacamole to access it for tomcat. This is to primarily make saml_strict mode work with guacamole 1.4, as per this mailing list post: https://www.mail-archive.com/user@guacamole.apache.org/msg09795.html. Please keep that in mind if you want to run other applications on the same Tomcat instance. It really isn't recommended you do that anyway - this tomcat should be just for the Guacamole client.

Requirements
------------

Works with Ansible 2.4.

Requires `become` or running as the `root` user. You can use `--ask-become-pass` when running. (often we use this with `-kKb`)

Role Variables
--------------
The following variables are set in `defaults/main`:

| Variable                 | Description                  |
|--------------------------|------------------------------|
|guacamole_version         | Guacamole version to install |
|guacamole_db_user         | Guacamole MariaDB username   |
|guacamole_db_password     | Guacamole MariaDB password   |
|guacamole_db_name         | Guacamole MariaDB database   |
|guacamole_proxy_address   | IP the reverse proxy is on   |
|ldap_auth_enable          | Enable LDAP authentication   |
|totp_2fa_enable           | Enable TOTP 2fa              |
|duo_2fa_enable            | Enable DUO 2fa               |
|saml_auth_enable          | Enable SAML authentication   |
|mysql_java_client_version | MySQL Java Client version    |
|guacamole_apt_install     | apt packages to install      |

There are lots of variables for the various features with 'enable' options which you can see in the guacamole.properties template.



Example Playbook
----------------

```
- hosts: guacamole-host
  become: yes
  roles:
    - sol1.guacamole-client
```

 Information
------------------

This role was based on work by [Alex Feigenson](https://github.com/alexfeig) and almost completely updated by [Sol1](https://sol1.com.au).

