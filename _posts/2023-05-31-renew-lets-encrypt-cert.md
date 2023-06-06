---
title: "how to renew a lets encrypt certificate in Ubuntu 18.04 or higher"
toc: true
toc_label: "Content"
tags:
  - ubuntu
  - lets encrypt
  - ssl
  - certificate
  - nginx
date: May 31, 2023
header:
  teaser: /assets/images/thumbnails/generic-thumb-400.jpg
excerpt: "Sometimes it is necessary to manually renew the certificate of our website when the nginx cron job fails."
---

# How to renew a lets encrypt certificate in ubuntu 18.04 or higher

To renew a Let's Encrypt certificate in Ubuntu 18.04, you can use the Certbot tool, which is the recommended way to obtain and manage Let's Encrypt certificates. In my case I did the process using AWS EC2 server, and I got an error which is quite common when trying to renew the certificate.

## Scenario

- My domain is: franvergara66.com
- The operating system my web server runs on is: Ubuntu 18.4
- My hosting provider, if applicable, is: Github pages 
- Web Server: Nginx
- I can login to a root shell on my machine: yes
- I'm using the terminal to login into my server
- The version of my client is: 0.40.0

Here's a step-by-step guide:

## Step 1: Install Certbot:

The first step is try to install certbot using the `apt install` command. 

```
$ sudo apt update
sudo apt install certbot
```

This command will install the agent called Certbot, which is free open source software that allows you to easily create Let's Encrypt SSLs on your unmanaged Linux server.

## Step 2: Stop the web server:

- If you're using Apache, run:

```
$ sudo systemctl stop apache2
```

If you're using Nginx, run:

```
$ sudo systemctl stop nginx
```

## Step 3: Run Certbot to renew the certificate:

```
sudo certbot renew
```

Here's an example of the output you might see when running the sudo certbot renew command in Nginx after successfully renewing a certificat: 

```
Certbot will automatically check for any certificates that are due for renewal and attempt to renew them.

Saving debug log to /var/log/letsencrypt/letsencrypt.log

-------------------------------------------------------------------------------
Processing /etc/letsencrypt/renewal/franvergara66.com.conf
-------------------------------------------------------------------------------
Cert is due for renewal, auto-renewing...
Plugins selected: Authenticator nginx, Installer nginx
Renewing an existing certificate for franvergara66.com

-------------------------------------------------------------------------------
new certificate deployed without reload, fullchain is
/etc/letsencrypt/live/franvergara66.com/fullchain.pem
-------------------------------------------------------------------------------

Congratulations, the certificate was successfully renewed!
```


Note: You can also use `sudo certbot renew --dry-run` to simulate the renewal process without actually making any changes. This can be useful for testing.

## Step 4: Common errors and  Troubleshooting

Sometimes when trying to renew the certificate you may encounter a message indicating that the certificate cannot be renewed, so it is necessary to do some validations to find out what is wrong.

I ran this command:

```
$ sudo certbot renew --dry-run
```

If all goes well and your certificate is ready for renewal you should see this message:

```
** DRY RUN: simulating 'certbot renew' close to cert expiry
** (The test certificates below have not been saved.)

Congratulations, all renewals succeeded. The following certs have been renewed:
/etc/letsencrypt/live/prod-backend.alectify.com/fullchain.pem (success)
** DRY RUN: simulating 'certbot renew' close to cert expiry
** (The test certificates above have not been saved.)
```

In my case if got this error (which we will try to solve):

```
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/mydomain.com.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert is due for renewal, auto-renewing...
Plugins selected: Authenticator nginx, Installer nginx
Renewing an existing certificate
Performing the following challenges:
http-01 challenge for mydomain.com
Using default addresses 80 and [::]:80 ipv6only=on for authentication.
Waiting for verification...
Challenge failed for domain mydomain.com
http-01 challenge for mydomain.com
Cleaning up challenges
Attempting to renew cert (mydomain.com) from /etc/letsencrypt/renewal/prod-mydomain.conf produced an unexpected error: Some challenges have failed.. Skipping.
All renewal attempts failed. The following certs could not be renewed:
  /etc/letsencrypt/live/mydomain.com/fullchain.pem (failure)

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

All renewal attempts failed. The following certs could not be renewed:
  /etc/letsencrypt/live/mydomain.com/fullchain.pem (failure)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1 renew failure(s), 0 parse failure(s)

IMPORTANT NOTES:
 - The following errors were reported by the server:

   Domain: mydmain.com
   Type:   unauthorized
   Detail: 2.211.168.8: Invalid response from
   http://mydoamin.com/.well-known/acme-challenge/ov6EBHInETwkZZ-oqLNI908jFXvN7PFK86ZCJYcdrtA:
   404

   To fix these errors, please make sure that your domain name was
   entered correctly and the DNS A/AAAA record(s) for that domain
   contain(s) the right IP address.
   ```

In this case we have to validate some things on our server:

- We need to have a look at the renewal conf and the certificates:

```
$ cat /etc/letsencrypt/renewal/franvergara66*
```

In this case we need to validate that the command shows us 5 elements for our domain:

- archive_dir
- cert
- privkey
- chain
- full chain

This should be the correct output:

```
# renew_before_expiry = 30 days
version = 0.40.0
archive_dir = /etc/letsencrypt/archive/franvergara66.com
cert = /etc/letsencrypt/live/franvergara66.com/cert.pem
privkey = /etc/letsencrypt/live/franvergara66.com/privkey.pem
chain = /etc/letsencrypt/live/franvergara66.com/chain.pem
fullchain = /etc/letsencrypt/lfranvergara66.com/fullchain.pem

if something is missing you should generate a new certificate.

# Options used in the renewal process
[renewalparams]
account = ef20bbb8bca7d645438e5368b9048dee
authenticator = nginx
installer = nginx
server = https://acme-v02.api.letsencrypt.org/directory
```

Now let`s check the certificates we have in the server with this command:

```
$ certbot certificates
```

In the output we need to check if the certificate for our website is listed, something like this:

```
Found the following certs:
  Certificate Name: prod-backend.alectify.com
    Domains: prod-backend.alectify.com
    Expiry Date: 2022-07-03 23:52:05+00:00 (INVALID: EXPIRED)
    Certificate Path: /etc/letsencrypt/live/ranvergara66.com/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/ranvergara66.com/privkey.pe
```

If all the above is correct we should check the configuration of our server, in my case Nginx and verify that we have added the port 80 HTTP in the server block, so we will execute the following command:

```
$ nginx -T
```

And we check the output:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
# configuration file /etc/nginx/nginx.conf:
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 768;
	# multi_accept on;
}

http {

	##
	# Basic Settings
	##

	sendfile on;
	tcp_nopush on;
	tcp_nodelay on;
	keepalive_timeout 65;
	types_hash_max_size 2048;
	# server_tokens off;

	# server_names_hash_bucket_size 64;
	# server_name_in_redirect off;

	include /etc/nginx/mime.types;
	default_type application/octet-stream;

	##
	# SSL Settings
	##

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
	ssl_prefer_server_ciphers on;

	##
	# Logging Settings
	##

	access_log /var/log/nginx/access.log;
	error_log /var/log/nginx/error.log;

	##
	# Gzip Settings
	##

	gzip on;

	# gzip_vary on;
	# gzip_proxied any;
	# gzip_comp_level 6;
	# gzip_buffers 16 8k;
	# gzip_http_version 1.1;
	# gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

	##
	# Virtual Host Configs
	##

	include /etc/nginx/conf.d/*.conf;
	include /etc/nginx/sites-enabled/*;
}


#mail {
#	# See sample authentication script at:
#	# http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
# 
#	# auth_http localhost/auth.php;
#	# pop3_capabilities "TOP" "USER";
#	# imap_capabilities "IMAP4rev1" "UIDPLUS";
# 
#	server {
#		listen     localhost:110;
#		protocol   pop3;
#		proxy      on;
#	}
# 
#	server {
#		listen     localhost:143;
#		protocol   imap;
#		proxy      on;
#	}
#}

# configuration file /etc/nginx/modules-enabled/50-mod-http-geoip.conf:
load_module modules/ngx_http_geoip_module.so;

# configuration file /etc/nginx/modules-enabled/50-mod-http-image-filter.conf:
load_module modules/ngx_http_image_filter_module.so;

# configuration file /etc/nginx/modules-enabled/50-mod-http-xslt-filter.conf:
load_module modules/ngx_http_xslt_filter_module.so;

# configuration file /etc/nginx/modules-enabled/50-mod-mail.conf:
load_module modules/ngx_mail_module.so;

# configuration file /etc/nginx/modules-enabled/50-mod-stream.conf:
load_module modules/ngx_stream_module.so;

# configuration file /etc/nginx/mime.types:

types {
    text/html                             html htm shtml;
    text/css                              css;
    text/xml                              xml;
    image/gif                             gif;
    image/jpeg                            jpeg jpg;
    application/javascript                js;
    application/atom+xml                  atom;
    application/rss+xml                   rss;

    text/mathml                           mml;
    text/plain                            txt;
    text/vnd.sun.j2me.app-descriptor      jad;
    text/vnd.wap.wml                      wml;
    text/x-component                      htc;

    image/png                             png;
    image/tiff                            tif tiff;
    image/vnd.wap.wbmp                    wbmp;
    image/x-icon                          ico;
    image/x-jng                           jng;
    image/x-ms-bmp                        bmp;
    image/svg+xml                         svg svgz;
    image/webp                            webp;

    application/font-woff                 woff;
    application/java-archive              jar war ear;
    application/json                      json;
    application/mac-binhex40              hqx;
    application/msword                    doc;
    application/pdf                       pdf;
    application/postscript                ps eps ai;
    application/rtf                       rtf;
    application/vnd.apple.mpegurl         m3u8;
    application/vnd.ms-excel              xls;
    application/vnd.ms-fontobject         eot;
    application/vnd.ms-powerpoint         ppt;
    application/vnd.wap.wmlc              wmlc;
    application/vnd.google-earth.kml+xml  kml;
    application/vnd.google-earth.kmz      kmz;
    application/x-7z-compressed           7z;
    application/x-cocoa                   cco;
    application/x-java-archive-diff       jardiff;
    application/x-java-jnlp-file          jnlp;
    application/x-makeself                run;
    application/x-perl                    pl pm;
    application/x-pilot                   prc pdb;
    application/x-rar-compressed          rar;
    application/x-redhat-package-manager  rpm;
    application/x-sea                     sea;
    application/x-shockwave-flash         swf;
    application/x-stuffit                 sit;
    application/x-tcl                     tcl tk;
    application/x-x509-ca-cert            der pem crt;
    application/x-xpinstall               xpi;
    application/xhtml+xml                 xhtml;
    application/xspf+xml                  xspf;
    application/zip                       zip;

    application/octet-stream              bin exe dll;
    application/octet-stream              deb;
    application/octet-stream              dmg;
    application/octet-stream              iso img;
    application/octet-stream              msi msp msm;

    application/vnd.openxmlformats-officedocument.wordprocessingml.document    docx;
    application/vnd.openxmlformats-officedocument.spreadsheetml.sheet          xlsx;
    application/vnd.openxmlformats-officedocument.presentationml.presentation  pptx;

    audio/midi                            mid midi kar;
    audio/mpeg                            mp3;
    audio/ogg                             ogg;
    audio/x-m4a                           m4a;
    audio/x-realaudio                     ra;

    video/3gpp                            3gpp 3gp;
    video/mp2t                            ts;
    video/mp4                             mp4;
    video/mpeg                            mpeg mpg;
    video/quicktime                       mov;
    video/webm                            webm;
    video/x-flv                           flv;
    video/x-m4v                           m4v;
    video/x-mng                           mng;
    video/x-ms-asf                        asx asf;
    video/x-ms-wmv                        wmv;
    video/x-msvideo                       avi;
}

# configuration file /etc/nginx/sites-enabled/govecs-gateway:
upstream gateway {
  server 127.0.0.1:81 max_fails=0;
}
server {

    listen       80;
    server_name  franvergara66.com;
    
    rewrite ^(.*) https://franvergara66.com$1 permanent;
}

server {

    listen       443 ssl http2;
    server_name  franvergara66.com;
 
    ssl_certificate_key /etc/letsencrypt/live/franvergara66.com/privkey.pem;
    ssl_certificate     /etc/letsencrypt/live/franvergara66.com/fullchain.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;

location ~* /api/connection/web-sockets.+ {
    proxy_pass http://gateway;
    proxy_http_version 1.1;
    break;
}

    location / {
#                 satisfy any;
#
#            allow 193.178.214.0/24;
#            allow 172.16.0.0/12;
#            allow 127.0.0.0/8;
#            deny  all;
#      auth_basic           "restricted Area";
#      auth_basic_user_file /etc/nginx/htpasswd;
      proxy_set_header   X-Real-IP          $remote_addr;
      proxy_set_header   X-Forwarded-For    $proxy_add_x_forwarded_for;
      proxy_pass                  http://gateway;
      break;
    } 
}
```

We must check that the output says syntax is ok and test is successful, as well as that port 80 is present in the server block.

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

If everything seems to be correctly configured, maybe there is a problem with Certbot that can't validate the certificate, so we install or reinstall Certbot and its Ngnix plugin with apt.:

```
$ sudo apt install certbot python3-certbot-nginx
```

After the installation we run again the command `sudo certbot renew --dry-run` and if the error that we had before disappears we can run `sudo certbot renew`.

Finally if the certificate is installed, we just need to restart the Nginx server to take the changes and that's it, we complete the process =)

