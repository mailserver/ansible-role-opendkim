Ansible Role: OpenDKIM
======================

[OpenDKIM](http://www.opendkim.org/) (Domain Keys Identified Mail) sender authentication.

When a user submits new mail from a domain that has a private DKIM key, the message is signed by this milter. Receiving servers can use this signature to verify it against the public DKIM key returned by a DNS lookup.

This role is part of the [Mailserver](https://github.com/mailserver) project.

Configuration
-------------

### opndkim_config

| Key | Default | Description |
| --- | ------- | ----------- |
| policy_source | mysql | Storage method for OpenDKIM private keys |
| daemon_user | opendkim | User that executes the opendkim milter |
| daemon_group | opendkim | Default group of the daemon user |
| milter.group | milter | Group of the milter socket |
| milter.path | 

MySQL Schema
------------

OpenDKIM expects a list of domain names, their private key and a selector used for DNS lookups.

| domain | dkim_private_key | dkim_selector |
| ------ | ---------------- | ------------- |
| example.com | `----- BEGIN PRIVATE RSA KEY ------` | `my-selector` |

DKIM key-pair generation
------------------------

With `opendkim-tools` installed, the tool [`opendkim-genkey`](http://www.opendkim.org/opendkim-genkey.8.html) can be used to generate a new OpenDKIM key-pair. It writes two files - one contains the private key that needs to be stored in the database, the other holds the DNS TXT record in Bind9 format.

```bash
export DKIM_SELECTOR=my-selector
opendkim-generate -b 4096 -d example.com -s $DKIM_SELECTOR 
cat $DKIM_SELECTOR.txt # Bind9 Configuration
cat $DKIM_SELECTOR.private # Private Key
```

After updating the DNS the result should be manually reviewed.

```bash
dig TXT my-selector._domainkey.example.com
```

Examples
--------

### Postfix MTA with OpenDKIM as submission milter with MySQL backend

```yaml
- role: mailserver.postfix
  ...
  submission_milters:
    - name: opendkim
      socket: "milters/opendkim.sock"
    ...

- role: mailserver.opendkim
  opendkim_mysql:
    host: "127.0.0.1"
    port: 3306
    user: "opendkim"
    password: "correct horse battery staple"
    database: "mail"
    table: "opendkim"
  opendkim_config:
    milter:
      group: "milter"
      socket_path: "/var/spool/postfix/milters/opendkim.sock"
```
