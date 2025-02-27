+++
title = 'Configuring Apache for OIDC with an Authentik backend'
date = 2025-02-26T20:06:16-05:00
draft = false
+++

## Summary

For the past year and half I've been running my own mailserver using [Postfix](https://www.postfix.org/) and [Dovecot](https://www.dovecot.org/).

This setup works quite well, and I've also installed [Roundcube](https://roundcube.net/) to have a convenient web frontend. It's nice to have, though most of the time I interact with the mailserver via apps like Thunderbird or FairEmail.

While Roundcube is quite nice, I don't necessarily trust the authentication mechanism to be exposed directly to the internet.

Since I set this up, I put Apache basic authentication in front of it as a deterrent. However, the issue with basic auth is that it requires you to log in again *every single time* you restart your browser.

So, I decided on a more robust solution. I've already been running Authentik for some of the other services in my lab that support OIDC, so I decided to tie back into that system, which will give me nice single sign on features.

## Configuring Apache

Here's how you can set this up for yourself.

First, install the Apache module that adds OIDC support:

```bash
sudo apt install mod_auth_openidc
```

Enable the module:

```bash
sudo a2enmod auth_openidc
```

Here's an example Apache virtual host with working OIDC support:

```
<VirtualHost *:443>
ServerAdmin webmaster@localhost
ServerName mail.example.net

OIDCProviderMetadataURL "https://auth.example.net/application/o/roundcube-webmail/.well-known/openid-configuration"
OIDCClientID redacted
OIDCClientSecret redacted
OIDCRedirectURI "https://mail.example.net/roundcube"
OIDCCacheType file
OIDCCacheDir /var/cache/mod_auth_openidc
OIDCCryptoPassphrase redacted

<Location /roundcube>
AuthType openid-connect
Require valid-user
</Location>

<Location />
AuthType openid-connect
Require valid-user
</Location>

# Handle the case if someone hits the root location and redirect them to /roundcube 
RewriteEngine On
RewriteCond %{REQUEST_URI} ^/$
RewriteCond %{HTTP_REFERER} !auth.example.net [NC]
RewriteRule ^/$ /roundcube/ [R=302,L]

ErrorLog ${APACHE_LOG_DIR}/error.log
CustomLog ${APACHE_LOG_DIR}/access.log combined

SSLEngine on
SSLCertificateFile      /etc/letsencrypt/live/mail.example.net/cert.pem
SSLCertificateKeyFile /etc/letsencrypt/live/mail.example.net/privkey.pem
SSLCertificateChainFile /etc/letsencrypt/live/mail.example.net/fullchain.pem
</VirtualHost>
```

Once you have this set up, restart apache:

```bash
sudo systemctl restart apache2
```

On systems other than Ubuntu the service may be called httpd rather than apache2.

## OIDC configuration options

`OIDCClientId`, `OIDCProviderMetadataURL` and `OIDCClientSecret` can both be found in your Authentik provider configuration.

`OIDCCryptoPassphrase` is only used internally. You can generate any random string for this, I'd recommend at least 64 characters.

Don't forget that you also need to set the redirect URI in the provider settings!
