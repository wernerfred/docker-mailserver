---
title: 'Best Practices | DKIM'
---

DKIM is a security measure targeting email spoofing. It is greatly recommended one activates it. See [the Wikipedia page](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail) for more details on DKIM.

## Enabling DKIM Signature

To enable DKIM signature, **you must have created at least one email account**. Once its done, just run the following command to generate the signature:

```sh
./setup.sh config dkim
```

After generating DKIM keys, you should restart the mail server. DNS edits may take a few minutes to hours to propagate. The script assumes you're being in the directory where the `config/` directory is located. The default keysize when generating the signature is 4096 bits for now. If you need to change it (e.g. your DNS provider limits the size), then provide the size as the first parameter of the command:

```sh
./setup.sh config dkim <keysize>
```

For LDAP systems that do not have any directly created user account you can run the following command (since `8.0.0`) to generate the signature by additionally providing the desired domain name (if you have multiple domains use the command multiple times or provide a comma-separated list of domains): 

```sh
./setup.sh config dkim <key-size> <domain.tld>[,<domain2.tld>]
```

Now the keys are generated, you can configure your DNS server with DKIM signature, simply by adding a TXT record. If you have direct access to your DNS zone file, then it's only a matter of pasting the content of `config/opendkim/keys/domain.tld/mail.txt` in your `domain.tld.hosts` zone.

```console
$ dig mail._domainkey.domain.tld TXT
---
;; ANSWER SECTION
mail._domainkey.<DOMAIN> 300 IN TXT    "v=DKIM1; k=rsa; p=AZERTYUIOPQSDFGHJKLMWXCVBN/AZERTYUIOPQSDFGHJKLMWXCVBN/AZERTYUIOPQSDFGHJKLMWXCVBN/AZERTYUIOPQSDFGHJKLMWXCVBN/AZERTYUIOPQSDFGHJKLMWXCVBN/AZERTYUIOPQSDFGHJKLMWXCVBN/AZERTYUIOPQSDFGHJKLMWXCVBN/AZERTYUIOPQSDFGHJKLMWXCVBN"
```

## Configuration using a Web Interface

1. Generate a new record of the type `TXT`.
2. Paste `mail._domainkey` the `Name` txt field.
3. In the `Target` or `Value` field fill in `v=DKIM1; k=rsa; p=AZERTYUGHJKLMWX...`.
4. In `TTL` (time to live): Time span in seconds. How long the DNS server should cache the `TXT` record.
5. Save.

**Note**: Sometimes the key in `config/opendkim/keys/domain.tld/mail.txt` can be on multiple lines. If so then you need to concatenate the values in the TXT record:

```console
$ dig mail._domainkey.domain.tld TXT
---
;; ANSWER SECTION
mail._domainkey.<DOMAIN> 300 IN TXT "v=DKIM1; k=rsa; "
    "p=AZERTYUIOPQSDF..."
    "asdfQWERTYUIOPQSDF..."
```

The target (or value) field must then have all the parts together: `v=DKIM1; k=rsa; p=AZERTYUIOPQSDF...asdfQWERTYUIOPQSDF...`

## Verify-Only

If you want DKIM to only _verify_ incoming emails, the following version of /etc/opendkim.conf may be useful (right now there is no easy mechanism for installing it other than forking the repo):

```txt
# This is a simple config file verifying messages only

#LogWhy                 yes
Syslog                  yes
SyslogSuccess           yes

Socket                  inet:12301@localhost
PidFile                 /var/run/opendkim/opendkim.pid

ReportAddress           postmaster@my-domain.com
SendReports             yes

Mode                    v
```

## Debugging

- [DKIM-verifer](https://addons.mozilla.org/en-US/thunderbird/addon/dkim-verifier): A add-on for the mail client Thunderbird.
- You can debug your TXT records with the `dig` tool.

```console
$ dig TXT mail._domainkey.domain.tld
---
; <<>> DiG 9.10.3-P4-Debian <<>> TXT mail._domainkey.domain.tld
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 39669
;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;mail._domainkey.domain.tld. IN	TXT

;; ANSWER SECTION:
mail._domainkey.domain.tld. 3600 IN TXT "v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCxBSjG6RnWAdU3oOlqsdf2WC0FOUmU8uHVrzxPLW2R3yRBPGLrGO1++yy3tv6kMieWZwEBHVOdefM6uQOQsZ4brahu9lhG8sFLPX4MaKYN/NR6RK4gdjrZu+MYSdfk3THgSbNwIDAQAB"

;; Query time: 50 msec
;; SERVER: 127.0.1.1#53(127.0.1.1)
;; WHEN: Wed Sep 07 18:22:57 CEST 2016
;; MSG SIZE  rcvd: 310
```

## Switch Off DKIM

Simply remove the DKIM key by recreating (not just relaunching) the mailserver container.
