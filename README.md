NowFap -  NOn Web Federated Authentication Portal
======

This project deploys COmanage using Ansible as a portal to handle
various non-web SSO scenarios. The portal needs an IdP for external
authentication. The IdP must adhere to the SAML 2.0 Web-SSO profile. The
EPPN is used to identify remote users and thus must be included in the
SAML response.

# Ansible playbooks and roles to install COmanage virtual machine.

This codebase contains all the roles/playbooks and templates to
configure a COmanage VMs.  The target Linux distribution for COmanage
server is Ubuntu Server 16.04

Machines are provisioned as follows:

```
ansible-playbook -i <local_inventory> comanage.yml
```

You are expected to provide your own inventory file as stated in the
above line. We will provide an example inventory file, so that you are
able to create your own.

## Example inventory
The below inventory specifies the different hosts that need to be
provisioned in order to get a working system. There are four major
parts:
1. the portal itself;
2. an LDAP server;
3. SSH daemon capable of accessing the LDAP server for SSH key
   retrieval;
4. LDAP management (this component is not necessary, but can be
   convenient to have).

A typical host file would look like:
```
[portal]
172.16.10.1  ansible_user=ubuntu

[ldap:children]
ldap-server
ssh-access

[ldap-server]
172.16.10.1 ansible_user=ubuntu

[ssh-access]
192.168.15.1 ansible_user=gerben

[phpldapadmin]
172.16.10.1  ansible_user=ubuntu
```
The `ldap-server` and `'ssh-access` hosts are placed in the same group,
as they share the IP address of the LDAP server.

Together with the above hosts file, `group_vars` are created as well.
Below an example of the group variables is given:

### Example of group_vars
Inside the `group_vars` directory a number of group files can be found:

1. **portal.yml**
   ```
   ---

   certificate: /etc/ssl/certs/portal.example.org.pem
   certificate_key: /etc/ssl/private/portal.examaple.org.key

   sp_hostname: portal.example.org
   ```
2. **ldap.yml**
   ```
   ---

   ldap_server: ldap://172.16.10.1
   ```
3. **ldap-server.yml**
   ```
   ---

   ldap_admin_passwd: BigSecret
   ldap_basedn: dc=portal,dc=example,dc=org
   ```
4. **ssh-access.yml**
   ```
   ---

   ldap_admin_passwd: BigSecret
   ldap_basedn: dc=portal,dc=example,dc=org
   ```

Please note that in the above hosts and group_vars files examples,
private IP addresses were used as well as unsecure password and other
dummy strings for certificates and host name.

# Roles
The playbook executes the following roles:

| Role       | Description |
| ---------- | ----------- |
apache       | The portal is build with Apache. |
auth_mellon  | Install and configure an Apache module for external SAML Authentication. |
comanage     | COmanage provides a portal in which the use case are implemented. |
common       | A number of command task, like package installation. |
ldap         | LDAP is used to store attributes (ASP, OTP, SSH Pub key, etc) collected by COmanage. |
mysql        | The framework requires a database. |
php          | Php 5.6 is necessary for the underlying framework. |
phpldapadmin | Web interface to an LDAP instance. |
sshd         | Setting up an SSH daemon that is able to retrieve SSH keys from an LDAP instance. |
surfconext   | Necessary metadata is fetched and verified in order to make the SAML flow work. |

# Caveats
There are number of caveats that you should be aware of.

## Install certificates yourself
The playbook currently does not setup certificates for you. Instead, it
is expected that certificates are already installed on the target
machine.  To support this, two variables are defined:
- certificate
- certificate_key

The first one, `certificate` points to the installed certificate, while
`certificate_key` points to the corresponding key. Use the `portal.yml`
group vars file to change the values, otherwise you will fallback to
default values.

| Variable | Default value |
| -------- | ------------ |
| certificate     | /etc/ssl/cert/portal.example.org.pem    |
| certificate_key | /etc/ssl/private/portal.example.org.key |

## Do not forget to specify parameters for mod_mellon
In order for mod_mellon to do the right thing, it needs to know a few
things first. You need to do this for each host, as these values are
unique to each one. Well, at least the sp_hostname is. So, please define
the following in your `portal.yml` group vars file:

| Variable | Description | Default value |
| -------- | ----------- | ------------- |
| sp_protocol | Protocol to be used | https:// |
| sp_hostname | Name of the host where mod_mellon is being used | portal.example.org |
| sp_path     | Path that follows the host name to create the full URI | 'registry/auth/sp' |
