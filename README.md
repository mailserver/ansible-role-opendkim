Ansible Role: OpenDKIM
======================

[OpenDKIM](http://www.opendkim.org/) (Domain Keys Identified Mail) sender authentication

This role is part of the [Mailserver](https://github.com/mailserver) project.

MySQL Schema
------------

- ToDo

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
    pass: "correct horse battery staple"
    db: "mail"
    table: "opendkim"
  opendkim_config:
    milter:
      group: "milter"
      socket_path: "/var/spool/postfix/milters/opendkim.sock"
```
