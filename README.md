# Secure Reverse Proxy with HAProxy and stunnel

This guide will help you set up a secure reverse proxy using HAProxy and stunnel. The reverse proxy will allow you to securely access a service running on your local PC through a remote server without the need for port forwarding.

## Prerequisites

- A local PC with a service running that you want to expose.
- A remote server with a public IP address or domain.
- stunnel installed on both the local PC and remote server.
- HAProxy installed on the remote server.

## Step-by-Step Guide

### 1. Set up stunnel

#### Local PC NATED BEHIND FIREWALL

a. Download and install stunnel for Windows on your local PC from the official website: https://www.stunnel.org/downloads.html

b. Create a new text file named `stunnel.conf` with the following content:
```
[local_service]
client = yes
accept = 127.0.0.1:local_tunnel_port
connect = remote_server_ip:remote_tunnel_port
```

vbnet
Copy code

Replace `local_tunnel_port` with an unused local port, `remote_server_ip` with the remote server's IP address, and `remote_tunnel_port` with an unused port on the remote server.

#### Remote Server

a. Install stunnel on the remote server using the package manager. For example, on a Debian-based system, use the following command:
```
sudo apt-get install stunnel
```

css
Copy code

b. Create or edit the stunnel configuration file at `/etc/stunnel/stunnel.conf` and add the following content:
```
[local_service]
client = no
accept = remote_tunnel_port
connect = 127.0.0.1:remote_port
cert = /path/to/cert.pem
key = /path/to/key.pem
```
markdown
Copy code

Replace `remote_tunnel_port` with the same port used in the local PC's stunnel configuration, `remote_port` with the desired port on the remote server for HAProxy, and `/path/to/cert.pem` and `/path/to/key.pem` with the paths to your SSL certificate and private key files.

### 2. Start stunnel

#### Local PC

a. Run the stunnel executable and specify the configuration file:
```
stunnel.exe stunnel.conf
```

shell
Copy code

#### Remote Server

a. Start the stunnel service:
```
sudo systemctl enable stunnel
sudo systemctl start stunnel
```

bash
Copy code

### 3. Configure HAProxy on the remote server

a. Create or edit the HAProxy configuration file at `/etc/haproxy/haproxy.cfg`.

b. Add the following configuration:
```
global
log /dev/log local0
log /dev/log local1 notice
chroot /var/lib/haproxy
user haproxy
group haproxy
daemon

defaults
log global
mode tcp
option tcplog
option dontlognull
timeout connect 5000
timeout client 50000
timeout server 50000

frontend main
bind *:public_port
mode tcp
default_backend local_service

backend local_service
mode tcp
server local_pc 127.0.0.1:remote_tunnel_port
```

vbnet
Copy code

Replace `public_port` with the desired port on the remote server that you want to expose to the public, and `remote_tunnel_port` with the same port used in the remote server's stunnel configuration (the `accept` parameter).

### 4. Enable and start the HAProxy service
```
sudo systemctl enable haproxy
sudo systemctl start haproxy
```

css
Copy code

Now, you should be able to access the service
