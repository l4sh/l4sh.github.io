---
layout: post
current: post
navigation: True
title: Set up a dockerized wordpress instance behind nginx
author: l4sh
date: 2019-01-10
category: development
class: post-template
subclass: post
tags:
  - devops
  - sysadmin
  - wordpress
  - nginx
  - docker
comments: true
---

This is a quick recipe for installing a containerized WordPress
environment behind Nginx.

The configuration files as well as any revision can be found in the
[Github repo](https://github.com/l4sh/dockerized-wordpress).

Nginx will act as a reverse proxy serving the content through HTTPS to
the clients and forwarding the requests to the Docker container
running WordPress.

A graphical representation of this configuration would be

```
             HTTPS                HTTP
           Port 443            Port 8000
[Browser] ----------> [Nginx] -----------> [Wordpress Container]
```

The project structure is the following

```
.
├── docker-compose.yml
├── nginx.conf
└── wp-content
```

If you did not download the files from the
[repo](https://github.com/l4sh/dockerized-wordpress) create them, as
well as the `wp-content` folder.

```
touch nginx.conf docker-compose.yml
mkdir wp-content
```

## Nginx configuration

For the nginx part lets start with the [secure configuration
maintained by plentz](https://gist.github.com/plentz/6737338).

I already have a wildcard certificate in place for my domain. If you
need one and are managing your DNS records with Cloudflare [this
guide](https://bjornjohansen.no/wildcard-certificate-letsencrypt-cloudflare)
might be of interest.

The site information to be used for this configuration is the following.

| Site domain          | wordpress.example.com                           |
| WP Container Port    | 8000                                            |
| Upstream name        | wordpress                                       |
| SSL Certificate path | /etc/letsencrypt/live/example.com/fullchain.pem |
| SSL Key path         | /etc/letsencrypt/live/example.com/privkey.pem   |
| SSL DHParam          | /etc/letsencrypt/live/example.com/dhparam.pem   |

Resulting in this nginx configuration file. I've added a `<-- CHANGEME`
indicator to highlight the lines that should be changed.

```nginx
# nginx.conf

# Configuration adapted from https://gist.github.com/plentz/6737338

# don't send the nginx version number in error pages and Server header
server_tokens off;

# config to don't allow the browser to render the page inside an frame or iframe
# and avoid clickjacking http://en.wikipedia.org/wiki/Clickjacking
# if you need to allow [i]frames, you can use SAMEORIGIN or even set an uri
# with ALLOW-FROM uri
# https://developer.mozilla.org/en-US/docs/HTTP/X-Frame-Options
add_header X-Frame-Options SAMEORIGIN;

# when serving user-supplied content, include a X-Content-Type-Options: nosniff
# header along with the Content-Type: header,
# to disable content-type sniffing on some browsers.
# https://www.owasp.org/index.php/List_of_useful_HTTP_headers
# currently suppoorted in IE > 8
# http://blogs.msdn.com/b/ie/archive/2008/09/02/ie8-security-part-vi-beta-2-update.aspx
# http://msdn.microsoft.com/en-us/library/ie/gg622941(v=vs.85).aspx
# 'soon' on Firefox https://bugzilla.mozilla.org/show_bug.cgi?id=471020
add_header X-Content-Type-Options nosniff;

# This header enables the Cross-site scripting (XSS) filter built into most
# recent web browsers.
# It's usually enabled by default anyway, so the role of this header is to
# re-enable the filter for
# this particular website if it was disabled by the user.
# https://www.owasp.org/index.php/List_of_useful_HTTP_headers
add_header X-XSS-Protection "1; mode=block";

# with Content Security Policy (CSP) enabled(and a browser that supports it
# (http://caniuse.com/#feat=contentsecuritypolicy),
# you can tell the browser that it can only download content from the domains
# you explicitly allow
# http://www.html5rocks.com/en/tutorials/security/content-security-policy/
# https://www.owasp.org/index.php/Content_Security_Policy
# I need to change our application code so we can increase security by disabling
# 'unsafe-inline' 'unsafe-eval'
# directives for css and js (if you have inline css or js, you will need to keep
# it too).
# more: http://www.html5rocks.com/en/tutorials/security/content-security-policy/#inline-code-considered-harmful
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' 'unsafe-eval' https://ssl.google-analytics.com https://assets.zendesk.com https://connect.facebook.net; img-src 'self' https://ssl.google-analytics.com https://s-static.ak.facebook.com https://assets.zendesk.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com https://assets.zendesk.com; font-src 'self' https://themes.googleusercontent.com; frame-src https://assets.zendesk.com https://www.facebook.com https://s-static.ak.facebook.com https://tautt.zendesk.com; object-src 'none'";

upstream wordpress {
    server 127.0.0.1:8000 fail_timeout=0;  # <-- CHANGEME
}

# Redirect traffic to SSL
server {
    listen 80;
    listen [::]:80;
    server_name wordpress.example.com;     # <-- CHANGEME
    return 301 https://$host$request_uri;
}

server {
  listen 443 ssl http2;
  listen [::]:443 ssl http2;
  server_name wordpress.example.com;                                  # <-- CHANGEME

  ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;    # <-- CHANGEME
  ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;  # <-- CHANGEME

  # enable session resumption to improve https performance
  # http://vincent.bernat.im/en/blog/2011-ssl-session-reuse-rfc5077.html
  ssl_session_cache shared:SSL:50m;
  ssl_session_timeout 1d;
  ssl_session_tickets off;

  # Diffie-Hellman parameter for DHE ciphersuites, recommended 2048 bits
  ssl_dhparam /etc/letsencrypt/live/example.com/dhparam.pem;          # <-- CHANGEME

  # enables server-side protection from BEAST attacks
  # http://blog.ivanristic.com/2013/09/is-beast-still-a-threat.html
  ssl_prefer_server_ciphers on;
  # disable SSLv3(enabled by default since nginx 0.8.19) since it's less secure then TLS http://en.wikipedia.org/wiki/Secure_Sockets_Layer#SSL_3.0
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  # ciphers chosen for forward secrecy and compatibility
  # http://blog.ivanristic.com/2013/08/configuring-apache-nginx-and-openssl-for-forward-secrecy.html
  ssl_ciphers ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS;

  # enable ocsp stapling (mechanism by which a site can convey certificate
  # revocation information to visitors in a privacy-preserving, scalable manner)
  # http://blog.mozilla.org/security/2013/07/29/ocsp-stapling-in-firefox/
  resolver 8.8.8.8 8.8.4.4;
  ssl_stapling on;
  ssl_stapling_verify on;
  # ssl_trusted_certificate /etc/letsencrypt/live/example.com/fullchain.pem;

  # config to enable HSTS(HTTP Strict Transport Security)
  # https://developer.mozilla.org/en-US/docs/Security/HTTP_Strict_Transport_Security
  # to avoid ssl stripping https://en.wikipedia.org/wiki/SSL_stripping#SSL_stripping
  # also https://hstspreload.org/
  add_header Strict-Transport-Security "max-age=31536000; includeSubdomains; preload";

  location / {
      proxy_pass http://wordpress;
      proxy_set_header Host $host;
      proxy_redirect off;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header X-Forwarded-Proto https;
      proxy_set_header X-Real-IP $remote_addr;
  }
}
```

After finishing any modifications create a link to this file from the
respective nginx folder. E.g.

```shell
sudo ln -s /srv/www/dockerized-wordpress/nginx.conf /etc/nginx/sites-enabled/wordpress
```

Check that the file works and reload the nginx configuration.

```
sudo nginx -t
sudo systemctl reload nginx
```

## Docker configuration

For the docker configuration I'll be running the [official
WordPress image](https://hub.docker.com/_/wordpress/) and a compose
file very similar to [the one found in the Docker documentation.
](https://docs.docker.com/compose/wordpress/)

`wp-content` will be mounted separately and points to a folder inside
within the project. This will allow us to apply version control and
back up any modification with more ease.

In my case the environment is for development and experimentation, so
I'll be setting up a multisite with some developer friendly
options. These settings are commented out for now. In the last section
you can see how to create a multisite. If you're looking on how to
create a multisite read that section before running the containers.

*NOTE: It's important to know that any change in
`WORDPRESS_CONFIG_EXTRA` won't be added to `wp-config.php` after the
container volume is created.  [Check this issue
[docker-library/wordpress/issues/333]](https://github.com/docker-library/wordpress/issues/333).
A way to work around this is to remove the `wp-config.php` file after
any update to `WORDPRESS_CONFIG_EXTRA` or remove the volume and allow
it to recreate it.*

```yaml
# docker-compose.yml

version: '3.3'

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: wproot
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8000:80"
    volumes:
      - wp_data:/var/www/html
      - ./wp-content:/var/www/html/wp-content
    restart: unless-stopped
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_CONFIG_EXTRA: |
        /* Site URL */
        define('WP_HOME', 'https://wordpress.example.com');     # <-- CHANGEME
        define('WP_SITEURL', 'https://wordpress.example.com');  # <-- CHANGEME
        /* Developer friendly settings */
        # define('SCRIPT_DEBUG', true);
        # define('CONCATENATE_SCRIPTS', false);
        # define('WP_DEBUG', true);
        # define('WP_DEBUG_LOG', true);
        # define('SAVEQUERIES', true);
        /* Multisite */
        # define('WP_ALLOW_MULTISITE', true );
        # define('MULTISITE', true);
        # define('SUBDOMAIN_INSTALL', false);
        # define('DOMAIN_CURRENT_SITE', 'wordpress.example.com');  # <-- CHANGEME
        # define('PATH_CURRENT_SITE', '/');
        # define('SITE_ID_CURRENT_SITE', 1);
        # define('BLOG_ID_CURRENT_SITE', 1);

volumes:
  db_data: {}
  wp_data: {}

```

Once the configuration is done run

```
docker-compose up
```

If everything went ok you should be able to go to your site URL (in
this case https://wordpress.example.com) and see the classic WordPress
5 minute install wizard (which now is more like 1 minute since some of it
has been preconfigured).

Only one more thing to do. Add your user to the `www-data` group and
change the owner of the `wp-content` folder to
`www-data:www-data`. Otherwise it will ask for information on where to
store any plugin or theme when attempting to install.

```
chown -Rf www-data:www-data wp-content
```

The site should be ready to start development now.

## Wordpress Multisite

The process for creating a multisite is a little different, and a
little annoying to be honest. If you know a quicker and better way
please let me know.

These instructions are for subdirectories multisites. I've not yet
tried out a subdomains configuration this way.

We need to follow the [Create a Network
guide](https://codex.wordpress.org/Create_A_Network) to some extent,
but since we're to lazy to just go into the container to manually
delete the `wp-config.php` file we'll just remove and recreate the
volume after updating the configuration. The database and any custom
addition should be left intact (that is if they were done in the
`wp-content` folder).


The docker way would be to first run the WordPress installation with
the `WP_ALLOW_MULTISITE` option enabled. That section of the
`docker-compose.yml` file should look like this.

```
      ...
      WORDPRESS_CONFIG_EXTRA: |
        ...
        /* Multisite */
        define('WP_ALLOW_MULTISITE', true );
        # define('MULTISITE', true);
        # define('SUBDOMAIN_INSTALL', false);
        # define('DOMAIN_CURRENT_SITE', 'wordpress.example.com');  # <-- CHANGEME
        # define('PATH_CURRENT_SITE', '/');
        # define('SITE_ID_CURRENT_SITE', 1);
        # define('BLOG_ID_CURRENT_SITE', 1);
...
```

Run the containers

```
docker-compose up
```

Go to the site URL and perform the installation.

After installing go to [Administration > Tools > Network
Setup](https://codex.wordpress.org/Tools_Network_Screen) and enter the
required information. Make sure to select Subdirectories.

After the process is finished and WordPress now displays which
settings to add to the `wp-config.php` file update the
`docker-compose.yml` file to uncomment the rest of the multisite
settings.

```
      ...
      WORDPRESS_CONFIG_EXTRA: |
        ...
        /* Multisite */
        define('WP_ALLOW_MULTISITE', true );
        define('MULTISITE', true);
        define('SUBDOMAIN_INSTALL', false);
        define('DOMAIN_CURRENT_SITE', 'wordpress.example.com');  # <-- CHANGEME
        define('PATH_CURRENT_SITE', '/');
        define('SITE_ID_CURRENT_SITE', 1);
        define('BLOG_ID_CURRENT_SITE', 1);
...
```

Since it won't add the updated configuration we'll just take down the
service, delete the `wp_data` volume and bring it back up.

The name of the volume can be different, make sure to check first. But
essentially the process should be.

```
docker-compose down
docker volume rm wp_data
docker-compose up
```

Now you should have a WordPress multisite
