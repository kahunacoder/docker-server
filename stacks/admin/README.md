# Administration Stack #

This stack is where I configure containers for server administration and monitoring tasks. This stack is not part of the proxy network and exposes no ports to the public internet thanks to the tailscale vpn configured on this server and our `Control Node`. In this example I've configured [Netdata Agent](https://www.netdata.cloud/agent/)

Netdata Agent lets you interact with all your system and app metrics from an intuitive dashboard filled with detailed, real-time visualizations of your data optimized for visual anomaly detection.

Follow the guide in the stacks [README.md](../README.md) file for how to use [Portainer](https://codeopolis.com/posts/beginners-guide-to-portainer/) to add this stack to your server.

Here's what the stack `docker-compose.yml` looks like, this file can be copied and pasted as is.

```yaml
version: '2'
services:

  NetData:
    image: netdata/netdata
    container_name: NetData
    network_mode: bridge
    hostname: "${HOSTNAME}"
    restart: unless-stopped
    ports:
      - "${NETDATA}:19999"
    cap_add:
      - SYS_PTRACE
    security_opt:
      - apparmor:unconfined
    volumes:
      - /etc/passwd:/host/etc/passwd:ro
      - /etc/group:/host/etc/group:ro
      - /proc:/host/proc:ro
      - /sys:/host/sys:ro
      - /etc/os-release:/host/etc/os-release:ro

```

And here is the `admin.env` file you'll need to edit:

```ini
HOSTNAME=docker-vm # This is the host name that will appear in the netdata dashboard. You can set that to whatever you want but I'd use the hostname you set in the server.yml file.
NETDATA=19999 # You can set this to whatever you want, this is the default.

```
