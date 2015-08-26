---
layout: docs
title: Single Sign-On
menu: docs
lang: en
---

You can connect a 4front instance to an LDAP compliant provider such as Active Directory to provide single sign-on for users, both to the [portal](/docs/portal.html) and the [CLI](/docs/cli.html). Additionally, virtual apps deployed to the platform can take advantage of LDAP authentication via the [ldap-auth plugin](/docs/plugins/ldap-auth.html).

## Configuration

In the global 4front configuration, the following configuration settings are required:

__`LdapUrl`__

The URL to the LDAP server, i.e. `ldap://directory.acme.net`.

__`LdapBaseDN`__

The base distinguished name (DN) of the directory, i.e. `dc=acme,dc=net`.

__`LdapUsersDN`__

The root DN of the tree where user objects are located, i.e. `ou=users,ou=accounts,dc=acme,dc=net`.

__`LdapGroupsDN`__

The root DN of the tree where group objects are located, i.e. `ou=groups,ou=accounts,dc=acme,dc=net`. This is used by ldap-auth plugin to restrict access to members of a specific group. Upon login the plugin will discover all the groups the authenticated user is a member of (including nested groups) and store the list in session state.

__`LdapUsernamePrefix`__

Optional setting for LDAP implementations where the bind operation requires the username be prefixed with a domain name, i.e. `acmecorp/username`. In Active Directory this is known as the NetBIOS name. In this case the config value would be set to `acmecorp/` (including the trailing slash).
