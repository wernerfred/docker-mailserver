---
title: 'User Management | Accounts'
hide:
  - toc # Hide Table of Contents for this page
---

## Adding a new account

Users (email accounts) are managed in `/tmp/docker-mailserver/postfix-accounts.cf`. **_The best way to manage accounts is to use the reliable [setup.sh](https://github.com/docker-mailserver/docker-mailserver/wiki/Setup-docker-mailserver-using-the-script-setup.sh) script_**. Or you may directly add the _full_ email address and its encrypted password, separated by a pipe:

``` INI
user1@domain.tld|{SHA512-CRYPT}$6$2YpW1nYtPBs2yLYS$z.5PGH1OEzsHHNhl3gJrc3D.YMZkvKw/vp.r5WIiwya6z7P/CQ9GDEJDr2G2V0cAfjDFeAQPUoopsuWPXLk3u1
user2@otherdomain.tld|{SHA512-CRYPT}$6$2YpW1nYtPBs2yLYS$z.5PGH1OEzsHHNhl3gJrc3D.YMZkvKw/vp.r5WIiwya6z7P/CQ9GDEJDr2G2V0cAfjDFeAQPUoopsuWPXLk3u1
````

In the example above, we've added 2 mail accounts for 2 different domains. Consequently, the mail server will automatically be configured for multi-domains. Therefore, to generate a new mail account data, directly from your docker host, you could for example run the following:

``` BASH
docker run --rm \
  -e MAIL_USER=user1@domain.tld \
  -e MAIL_PASS=mypassword \
  -ti mailserver/docker-mailserver:latest \
  /bin/sh -c 'echo "$MAIL_USER|$(doveadm pw -s SHA512-CRYPT -u $MAIL_USER -p $MAIL_PASS)"' >> config/postfix-accounts.cf
```

You will then be asked for a password, and be given back the data for a new account entry, as text. To actually _add_ this new account, just copy all the output text in `config/postfix-accounts.cf` file of your running container. Please note the `doveadm pw` command lets you choose between several encryption schemes for the password. Use doveadm pw -l to get a list of the currently supported encryption schemes.

**Note**: Changes to the accounts list require a restart of the container, using `supervisord`. See [#552](https://github.com/docker-mailserver/docker-mailserver/issues/552).

---

### Notes

- `imap-quota` is enabled and allow clients to query their mailbox usage.
- When the mailbox is deleted, the quota directive is deleted as well.
- Dovecot quotas support LDAP, **but it's not implemented** (_PR are welcome!_).