---
layout: post
tags:
- blog
- MacOS
title: Postfix relay on MacOS High Sierra (and later)
date: 2017-12-07
category: MacOS
featured_image: /blog/etcpostfix.png
---

Every once in a while, I wipe and re-install my laptop. When I do that, I invariably have to remember to add smtp relaying via GMail to my local postfix configuration. Since it's something that I do so rarely, I've given up on trying to remember.

So, for posterity, assuming you’re using [Gmail](https://mail.google.com), and on MacOS High Sierra (nb: This still works on Catalina too):

In `/etc/postfix`, edit `main.cf`, make the following edits

Line 657 (or thereabouts) contains a inet_protocols directive.

Change it from:

``` shell
inet_protocols = all
```

to:

``` shell
inet_protocols = ipv4
```

And add the following:

``` shell
# Customizations
# Disable backwards-compatibility mode -- not needed.
compatibility_level = 2

#Gmail SMTP
relayhost=[smtp.gmail.com]:587

#Enable SASL authentication in the Postfix SMTP client.
smtp_sasl_auth_enable=yes
smtp_sasl_password_maps=hash:/etc/postfix/smtp_sasl_passwd
smtp_sasl_security_options=noanonymous
smtp_sasl_mechanism_filter=plain

#Enable Transport Layer Security (TLS), i.e. SSL.
smtp_use_tls=yes
smtp_tls_security_level=encrypt
compatibility_level = 2
```

A couple notes:

- Regarding relayhost, enclosing smtp.gmail.com in square brackets prevents Postfix from doing an MX lookup on the hostname.  Google explicitly tells you to use [smtp.gmail.com in their docs](https://support.google.com/a/answer/2956491?hl=en), so no need to do an MX lookup on it.
- Your Gmail password is saved in a hash map named smtp_sasl_passwd.  We’ll create it next.

Create `smtp_sasl_passwd` in your favorite text editor. The format should follow this pattern:

    [smtp.gmail.com]:587 username@gmail.com:password

Replace the username with yours, and use an [App Password](https://myaccount.google.com/apppasswords).  Google will fail your authentication attempt if you try it with your normal Google password.

Once that's done, I like to set permissions on the file to owner read/write only, and then generate the hashmap.

``` shell
sudo chown root:wheel /etc/postfix/smtp_sasl_passwd
sudo chmod 600 /etc/postfix/smtp_sasl_passwd
sudo postmap /etc/postfix/smtp_sasl_passwd
```

Then, start up Postfix:

``` shell
sudo postfix start
```

You can verify that Postfix has started correctly with nc:

``` shell
    nc localhost 25
    220 hostname.local ESMTP Postfix
```

Try pointing any applications to use the local smtp daemon, send a message, and you should be all set.
