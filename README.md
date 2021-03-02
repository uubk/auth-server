# Auth-Server
[![CircleCI status](https://img.shields.io/circleci/project/github/uubk/auth-server/master.svg?style=shield)](https://circleci.com/gh/uubk/auth-server/tree/master)
![License](https://img.shields.io/github/license/uubk/auth-server.svg?style=popout)

Set up 389ds and MIT Kerberos to authenticate users. Tested on Debian 9.

## Description
This role sets up 389ds in multi-master mode.

## Configuration
| Name | Default value | Description |
| ---- | ------------- | ----------- |
| `auth_ldap_domain` | `False` | Which base domain should be used for this role? |
| `auth_ldap_domain_ldap` | `False` | `auth_ldap_domain` in LDAP format (dc=...) |
| `auth_ldap_domain_suffix` | `False` | The first domain part of `auth_ldap_domain`) |
| `auth_ldap_admin_pwd` | `False` | The LDAP administrator password |
| `auth_ldap_sync_pwd` | `False` | The LDAP syncrepl user password |
| `auth_ldap_group` | `core` | The group of hosts this role is applied to |
| `auth_ldap_init_source` | `False` | The name of the host that should be used as a LDAP data source when adding new hosts. |
| `auth_kerberos_ldap_password` | `False` | The kerberos LDAP service account password |
| `auth_kerberos_database_master_key` | `False` | The initial kerberos database master key |
| `auth_kerberos_enctypes` | `aes256-cts-hmac-sha384-192 aes128-cts-hmac-sha256-128` | Which encryption modes to enable? The default is for recent versions of Kerberos and no Windows clients only. |
| `auth_ldap_have_tls` | `True` | Whether to enable SSL/TLS support in openLDAP |
| `auth_ldap_ssl_cert_path` | `/etc/ldap/server.pem` | Path to openLDAP's certificate |
| `auth_ldap_ssl_key_path` | `/etc/ldap/server.key` | Path to openLDAP's certificate's key|
| `auth_ldap_ssl_ca_path` | `/etc/ldap/ca.pem` | Path to the CA certificate of openLDAP's certificate |
| `auth_ldap_users` | `[]` | User accounts to create, see below |
| `auth_ldap_allow_read` | `[]` | DNs of users that should be granted read permissions on users and groups |
| `auth_ldap_services` | `[dns, ldap, host]` | Service account containers to create |
| `auth_ldap_service_bases` | (see defaults/main.yml) | LDAP containers to create for services |
| `auth_ldap_service_accounts` | (see defaults/main.yml) | Kerberos services to generate. This will also write out a keytab for each service. |
| `auth_ldap_permissions` | (see defaults/main.yml) | ACIs to set on the directory |
| `auth_kerberos_admin_privs` | `[]` | Kerberos principals to grant administrative permissions to (see defaults/main.yml for format) |
| `auth_ldap_store_pam` | `True` | Whether to actually store the generated 389ds PAM config. Useful if you want to customize it using another role |
|`auth_kerberos_curves` | `edwards25519` | Curves to use for kerberos SPAKE |
| `auth_ldap_use_memberof_plugin` | `False` | Whether to enable the `memberOf` LDAP plugin. |

Users can be created by putting them into `auth_ldap_users` as a dict with the following format:
```
auth_ldap_users:
  - name: Foo Bar
    id: foobar
    givenName: Foo
    sn: Bar
    uid: 10000
    gid: 1000
    mail: "test@example.org"
```
After running the playbook, use `kadmin.local` on one of the servers and do `cpw foobar` to set a password.

## FAQ

### Enabling memberOf plugin
If you enable the memberOf plugin by setting the `auth_ldap_user_memberof_plugin` variable to true, you need to pay attention to a few points:

* The user objects that should have the memberOf need to have the `objectClass` `inetUser` set. More information about `inetUser` can be found [here](https://msg.wikidoc.info/index.php/InetUser_LDAP_Object_Class#targetText=InetUser%20LDAP%20Object%20Class&targetText=For%20Mail%3A,for%20creating%20a%20mail%20account.&targetText=Group%20entries%20may%20be%20extended%20with%20this%20class.).
* If you're enabling memberOf on an existing database you will have to do an initial sync of the memberOf field, as described in section 5 [here](https://access.redhat.com/solutions/28282).

## License
Apache 2.0, except for the included LDAP schemas:
 * `files/kerberos.ldif` is `CoPyRiGhT=(c) Copyright 2006, Novell, Inc.  All rights reserved` and has been extracted from the freely available openLDAP source.
 * `files/dns.ldif` has been extracted from the [Bind Dyndb LDAP Backend](https://pagure.io/bind-dyndb-ldap/blob/master/f/doc/schema.ldif) (GPLv2) and rewritten to ldapmodify format
