# Nginx Proxy Manager #

Our server answers to one IP address so there is only one port 80 and one port 443. If we want to host multiple web sites we need a way to access these ports. The best way to do this is to create a proxy server. Nginx is one of the most popular proxy servers. I've chosen to use [Nginx Proxy Manager](https://www.jc21.com/2018/02/nginx-proxy-manager.html) because it provides a simple interface for containers running on the server.

There are many ways to implement this on your server. I've chosen to create a private network with docker and attach all sites to the private network. This seems the simplest method and I like the isolation. I created a private network for docker when the server was configured with ansible. I've provided a stack with the `docker-compose.yml` file in this directory and a `network.env` file with all of the environment variables making it easy to configure and modify.

By placing all public servers in the private network you can use the stack's `services:` name as a host name. So for example in the wordpress example the wordpress container is running as the `services:` `wordpress` so in the proxy I set the host to `wordpress`. This makes everything host agnostic and works both live and in the vm.

Follow the guide in the stacks [README.md](../README.md) file for how to use [Portainer](https://codeopolis.com/posts/beginners-guide-to-portainer/) to add this stack to your server.

Here's what the stack `docker-compose.yml` looks like, this file can be copied and pasted as is.

```yaml
version: "2.1"
services:
  proxy:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    container_name: proxy-manager
    ports:
      - '80:80' # Public HTTP Port
      - '443:443' # Public HTTPS Port
      - '81:81' # Admin Web Port
      # Add any other Stream port you want to expose
      # - '21:21' # FTP
    environment:
      DB_MYSQL_HOST: "proxydb"
      DB_MYSQL_PORT: 3306
      DB_MYSQL_USER: ${DB_MYSQL_USER}
      DB_MYSQL_PASSWORD: ${DB_MYSQL_PASSWORD}
      DB_MYSQL_NAME: ${DB_MYSQL_NAME}
      DISABLE_IPV6: ${DISABLE_IPV6}
    volumes:
      - ${data}:/data
      - ${letsencrypt}:/etc/letsencrypt
    depends_on:
      - proxydb

  proxydb:
    image: 'jc21/mariadb-aria:latest'
    restart: unless-stopped
    container_name: proxy-manager-db
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
      MYSQL_USER: ${MYSQL_USER}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
    volumes:
      - ${mysql}:/var/lib/mysql

# This is the private network that Nginx Proxy Manager will use
# to connect public sites to the internet. This needs to be on all
# public facing stacks such as our example wordpress stack.
networks:
  default:
    external:
      name: proxy
```

And here is the `network.env` file you'll need to edit:

```ini
# This is the Nginx Proxy Manager Database Username.
# Set this to what you want.
DB_MYSQL_USER="NPMUSER"
DB_MYSQL_PASSWORD="SUPERSTRONGNPMUSERPASSWORD" # Make this long and secure.

# I like short names but you can name it what you want.
# This will be the name of the Nginx Proxy Manager Database
DB_MYSQL_NAME="npm"
# Set this to false if IPv6 is enabled on your host
DISABLE_IPV6='true'
# Volume paths store persistent data on the server.
# I put all docker persistent data in the "portmaster" user's
# home folder.
# The "portmaster" user has no login or shell.
data="/home/pm/.appdata/nginx-proxy-manager/data"
letsencrypt="/home/pm/.appdata/letsencrypt"

# This is the Nginx Proxy Manager Database Password.
# for the mysql root user.
# make this different from DB_MYSQL_PASSWORD above
MYSQL_ROOT_PASSWORD='SUPERSTRONGROOTPASSWORD' # Make this long and secure.
MYSQL_DATABASE='npm' # Same as DB_MYSQL_NAME above.
MYSQL_USER='NPMUSER' # Same as DB_MYSQL_USER above.
MYSQL_PASSWORD='SUPERSTRONGNPMUSERPASSWORD' # Same as DB_MYSQL_PASSWORD above.
# Volume Paths
mysql="/home/pm/.appdata/nginx-proxy-manager-db"```
