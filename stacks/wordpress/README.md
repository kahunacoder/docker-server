# Wordpress Stack #

This stack is where I configure containers for server administration and monitoring containers. This stack is not part of the proxy network and exposes no ports to the public internet thanks to the tailscale vpn configured on this server and our `Control Node`. In this example I've configured [Wordpress](https://kinsta.com/knowledgebase/what-is-wordpress/) Using the official docker.com wordpress sample. I modified this to work on arm or x86 by replacing the database with mariaDB.

I've also included the popular database gui [Adminer](https://www.adminer.org) for manipulating the wordpress database. This tool also exposes the Nginx Proxy Manager database and any other databases on the server. Adminer is not exposed to the public internet unless you add it using Nginx Proxy Manager. It can only be accessed from your tailscale vpn.

What is WordPress? At its core, WordPress is the simplest, most popular way to create your own website or blog. In fact, WordPress powers over 40.0% of all the websites on the Internet. Yes – more than one in four websites that you visit are likely powered by WordPress.

Follow the guide in the stacks [README.md](../README.md) file for how to use [Portainer](https://codeopolis.com/posts/beginners-guide-to-portainer/) to add this stack to your server.

Here's what the stack `docker-compose.yml` looks like, this file can be copied and pasted as is.

```yaml
version: "2.1"
services:
  # this is the name of a service and since we have it running on the
  # 'proxy' network we can use this when we configure
  # 'Nginx Proxy Manager'
  wordpress: # wordpress:80 in 'Nginx Proxy Manager'
    depends_on: # These are the names of other services
      - wordpressdb # Since we named our database service wordpressdb
      - adminer # Since we named our database gui service adminer
    image: wordpress
    container_name: ${wordpress_container_name}
    restart: unless-stopped
    environment:
      WORDPRESS_DB_HOST: wordpressdb:3306 # Since we named our database service wordpressdb
      WORDPRESS_DB_USER: ${WORDPRESS_DB_USER}
      WORDPRESS_DB_PASSWORD: ${WORDPRESS_DB_PASSWORD}
      WORDPRESS_DB_NAME: ${WORDPRESS_DB_NAME}
    volumes:
      - ${wordpress_data}:/var/www/html

  # Here we name our database service wordpressdb
  wordpressdb:
    image: lscr.io/linuxserver/mariadb
    container_name: ${wordpressDB_container_name}
    restart: unless-stopped
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
      PUID: ${PUID}
      PGID: ${PGID}
      TZ: ${TZ}
    volumes:
      - ${db_data}:/config

  # Here we name our database gui service adminer
  adminer:
    image: adminer
    container_name: ${adminer_container_name}
    restart: unless-stopped
    links:
      - wordpressdb # Since we named our database service wordpressdb
    ports:
      - "${ADMINER_PORT}:8080" # The port the adminer web interface will be available on.
    environment:
      ADMINER_PLUGINS: ${ADMINER_PLUGINS}
      ADMINER_DESIGN: ${ADMINER_DESIGN}
      ADMINER_DEFAULT_SERVER: ${ADMINER_DEFAULT_SERVER}

# This is the private network that Nginx Proxy Manager will use
# to connect public sites to the internet. This needs to be on all
# public facing stacks such as here in our example wordpress stack.
networks:
  default:
    external:
      name: proxy
```

Adminer has some plugins and themes you can configure, see [here](https://github.com/docker-library/docs/blob/master/adminer/content.md) for more information.

And here is the `admin.env` file you'll need to edit:

```ini
# Good container names make it easier to identify what it's doing.
#  As seen in the examples below, the container names have a dash in them.
# The dash indicates what it's for.
wordpress_container_name=Wordpress-Site # Wordpress Site
# This is the Wordpress Database Username.
# Set this to what you want.
WORDPRESS_DB_USER="WPADMINUSER"
WORDPRESS_DB_PASSWORD="SUPERSTRONGWPADMINUSERPASSWORD" # Make this long and secure.
# This will be the name of the Wordpress Database
# Keep in mind that this database is only used by this container.
# There will only be this one database.
# I like short names but you can name it what you want.
WORDPRESS_DB_NAME="wp"
# Volume paths store persistent data on the server.
# I put all docker persistent data in the "portmaster" user's
# home folder.
# The "portmaster" user has no login or shell.
wordpress_data="/home/pm/.appdata/wordpress/data"

wordpressDB_container_name=Wordpress-DB # Wordpress Database
# This is the Wordpress Database Password.
# for the mysql root user.
# make this different from WORDPRESS_DB_PASSWORD above
MYSQL_ROOT_PASSWORD='SUPERSTRONGROOTPASSWORD'
MYSQL_DATABASE='wp' # Same as WORDPRESS_DB_NAME above.
MYSQL_USER='WPADMINUSER' # Same as WORDPRESS_DB_USER above.
MYSQL_PASSWORD='SUPERSTRONGWPADMINUSERPASSWORD' # Same as WORDPRESS_DB_PASSWORD above.

# User and group ids generated when a user is added.
# Since this is an ubuntu install the ubuntu user is set at
# uid=1001 and guid=1001. Each user id is incremented by one
# so this user will get uid=1002 and guid=1002.
# Here we set it with the defaults set in the
# group_vars/all/server.yml file used by ansible.
PUID=1002 # portmaster user id
PGID=1002 # portmaster group id
TZ=America/New_York # the timezone of your server

# Volume paths store persistent data on the server.
# I put all docker persistent data in the
# "portmaster" user's home folder.
# The "portmaster" user has no login or shell.
db_data="/home/pm/.appdata/wordpress-db"

adminer_container_name=WordpressDB-GUI # Wordpress Database GUI
# If a plugin requires parameters to work correctly
# instead of adding the plugin to ADMINER_PLUGINS,
# you need to add a custom file to the container.
ADMINER_PLUGINS="tables-filter tinymce" # plugins to load.
ADMINER_DESIGN='pappu687' # the design to use.
# Here we take advantage of dockers hostname networking.
# Since we put this here for wordpress, we will use 'wordpressdb'
# as the hostname. We can do this because in the docker-compose.yml
# file we have the wordpressdb service: named 'wordpressdb'
ADMINER_DEFAULT_SERVER='wordpressdb' # the default server to use.```
