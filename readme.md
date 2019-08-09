# Wordpress Caddy Docker Project

## What is this?
I've made a docker compose config to raise up a wordpress instance using mariadb and served by caddy webserver as a reverse proxy.

### docker-compose.yaml
```yaml
version: "3.7"
services:
  caddy:
    image: abiosoft/caddy:php
    volumes:
      - ./config/Caddyfile:/etc/Caddyfile
      - ./config:/root/.caddy
    ports:
      - "2015:2015"
  mariadb:
    image: 'bitnami/mariadb:10.3'
    volumes:
      - 'mariadb_data:/bitnami'
    environment:
      - MARIADB_USER=bn_wordpress
      - MARIADB_DATABASE=bitnami_wordpress
      - ALLOW_EMPTY_PASSWORD=yes
  wordpress:
    image: 'bitnami/wordpress:5'
    volumes:
      - 'wordpress_data:/bitnami'
    depends_on:
      - mariadb
    environment:
      - MARIADB_HOST=mariadb
      - MARIADB_PORT_NUMBER=3306
      - WORDPRESS_DATABASE_USER=bn_wordpress
      - WORDPRESS_DATABASE_NAME=bitnami_wordpress
      - ALLOW_EMPTY_PASSWORD=yes
volumes:
  mariadb_data:
    driver: local
  wordpress_data:
    driver: local
```

### Caddyfile
```
0.0.0.0:2015 {
    proxy / wordpress:80 {
        header_upstream Host {host}
        header_upstream X-Real-IP {remote}
        header_upstream X-Forwarded-For {remote}
        header_upstream X-Forwarded-Port {server_port}
        header_upstream X-Forwarded-Proto {scheme}
    }
}
```
This is a simple caddy config to open a webserver on `0.0.0.0:2015` which defines a proxy from `/` to `wordpress` container on port `80`.
The proxy is transparent, all upstream headers are transmitted to wordpress' webserver.


### MariaDB
```yaml
      - MARIADB_USER=bn_wordpress
      - MARIADB_DATABASE=bitnami_wordpress
      - ALLOW_EMPTY_PASSWORD=yes
```
Those are default options from bitnami's [MariaDB](https://hub.docker.com/r/bitnami/mariadb). You can put your preferred credentials.

### Wordpress
```yaml
      - MARIADB_HOST=mariadb
      - MARIADB_PORT_NUMBER=3306
      - WORDPRESS_DATABASE_USER=bn_wordpress
      - WORDPRESS_DATABASE_NAME=bitnami_wordpress
      - ALLOW_EMPTY_PASSWORD=yes
```
Those are default options from bitnami's [Wordpress](https://hub.docker.com/r/bitnami/wordpress. You can put your preferred credentials here too.

### Persistence
For your data to persist on MariaDB or Wordpress containers, define a volume for each one like in bellow example.
```yaml
services:
  mariadb:
  ...
    volumes:
      - /path/to/mariadb-persistence:/bitnami
  ...
  wordpress:
  ...
    volumes:
      - /path/to/wordpress-persistence:/bitnami
  ...
```
Also, check this link [Wordpress Docker Hub](https://hub.docker.com/r/bitnami/wordpress#persisting-your-application).

> I've already done this for you.

## Environment variables for wordpress
You can find all variables on [Wordpress Environment Variable](https://hub.docker.com/r/bitnami/wordpress#environment-variables).
I've made a copy here with the essential ones if you still don't want to click it.

```
# User and Site configuration
WORDPRESS_USERNAME: WordPress application username. Default: user
WORDPRESS_PASSWORD: WordPress application password. Default: bitnami
WORDPRESS_EMAIL: WordPress application email. Default: user@example.com
WORDPRESS_FIRST_NAME: WordPress user first name. Default: FirstName
WORDPRESS_LAST_NAME: WordPress user last name. Default: LastName
WORDPRESS_BLOG_NAME: WordPress blog name. Default: User's blog
WORDPRESS_SCHEME: Scheme to generate application URLs. Default: http
WORDPRESS_HTACCESS_OVERRIDE_NONE: Set the Apache AllowOverride variable to None. All the default directives will be loaded from /opt/bitnami/wordpress/wordpress-htaccess.conf. Default: yes.

# Use an existing database
MARIADB_HOST: Hostname for MariaDB server. Default: mariadb
MARIADB_PORT_NUMBER: Port used by MariaDB server. Default: 3306
WORDPRESS_DATABASE_NAME: Database name that WordPress will use to connect with the database. Default: bitnami_wordpress
WORDPRESS_TABLE_PREFIX: Table prefix to use in WordPress. Default: wp_
WORDPRESS_DATABASE_USER: Database user that WordPress will use to connect with the database. Default: bn_wordpress
WORDPRESS_DATABASE_PASSWORD: Database password that WordPress will use to connect with the database. No defaults.
WORDPRESS_SKIP_INSTALL: Force the container to not execute the WordPress installation wizard. This is necessary in case you use a database that already has WordPress data. Default: no
ALLOW_EMPTY_PASSWORD: It can be used to allow blank passwords. Default: no
```

# Final note
> HAVE FUN!