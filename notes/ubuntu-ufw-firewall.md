# UFW Ubuntu Firewall Setup Tutorial

https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-with-ufw-on-ubuntu

# Step 1 — Making Sure IPv6 is Enabled

### Enable IPv6

```bash
sudo nano /etc/default/ufw

# set to yes then save the file `IPV6=yes`
```

# Step 2 — Setting Up Default Policies

### Setup Default Policy

Deny all incoming connections and allow all outgoing connections.
Anyone trying to reach your server would not be able to connect, while any application within the server would be able to reach the outside world.

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

# Step 3 — Allowing SSH Connections

### Allowing the OpenSSH UFW Application Profile

If you’re using a cloud server, you will probably want to allow incoming SSH connections so you can connect to and manage your server.

```bash
# check which profiles are currently registered in UFW
sudo ufw app list


# To enable the OpenSSH application profile, run:
# This will create firewall rules to allow all connections on port 22,
# which is the port that the SSH daemon listens on by default.
sudo ufw allow OpenSSH 


# Another way to configure UFW to allow incoming SSH connections is by
# referencing its service name: ssh
sudo ufw allow ssh

# Alternatively, you can write the equivalent rule by specifying the port
# instead of the application profile or service name. For example,
# this command works the same as the previous examples:
sudo ufw allow 22
```

### SSH in different port

If you configured your SSH daemon to use a different port,
you will have to specify the appropriate port. For example,
if your SSH server is listening on port 2222, you can use
this command to allow connections on that port:

```bash
sudo ufw allow 2222
```

# Step 4 — Enabling UFW

**WARNING:** if you are connected to a remote server through SSH, make sure the SSH rule from Step 3 has been added **before** enabling the firewall, otherwise you will lock yourself out.

Verify which rules were added so far, even when the firewall is still disabled

```bash
sudo ufw show added
```

```bash
# enable firewall
sudo ufw enable
```

The firewall is now active. Run this command to see the rules that are set. 

```bash
sudo ufw status verbose
```

## Step 5 — Allowing Other Connections (intro to ufw `allow` parameter)

At this point, you should allow all of the other connections that your server needs to respond to. The connections that you should allow depend on your specific needs. 

### Allow `HTTP` on port `80`, which is what unencrypted web servers use

```bash
sudo ufw allow http
# or
sudo ufw allow 80
```

### Allow `HTTPS` on port `443`, which is what encrypted web servers use

```bash
sudo ufw allow https
# or
sudo ufw allow 443
```

### Allow Nginx & Apache with both HTTP and HTTPS

(these application profiles only exist after Apache/Nginx is installed, check with `sudo ufw app list`; use normal straight quotes, not curly ‘ ’ quotes)

```bash
sudo ufw allow 'Apache Full'
sudo ufw allow 'Nginx Full'
```

### Specific Port Ranges

For example, to allow X11 connections, which use ports 6000-6007, use these commands:

```bash
sudo ufw allow 6000:6007/tcp
sudo ufw allow 6000:6007/udp
```

### Specific IP Addresses (intro to ufw `from` parameter)

You can also specify IP addresses within your rules. For example,
if you want to allow connections from a specific IP address,
such as a work or home IP address of 203.0.113.4, you need to use
the from parameter, providing then the IP address you want to allow:

```bash
sudo ufw allow from 203.0.113.4
```

You can also specify a port that the IP address is allowed to connect
to by adding `to any port` followed by the port number. For example,
if you want to allow 203.0.113.4 to connect to port 22 (SSH):

```bash
sudo ufw allow from 203.0.113.4 to any port 22
```

### Allow Subnets (intro to ufw `/<bit-mask>`)

If you want to allow a subnet of IP addresses, you can do so using CIDR
notation to specify a netmask. For example, if you want to allow all of
the IP addresses ranging from 203.0.113.1 to 203.0.113.254:

```bash
# the first 24 bits of the IP address (203.0.113) are fixed,
# remember IPv4 is composed of 4 uint8/bytes, meaning using /24
# is masking the ip with 255.255.255.0
# (this actually covers 203.0.113.0 - 203.0.113.255)
sudo ufw allow from 203.0.113.0/24
```

Likewise, you may also specify the destination port that the
subnet 203.0.113.0/24 is allowed to connect to. Again,
we’ll use port 22 (SSH) as an example:


```bash
sudo ufw allow from 203.0.113.0/24 to any port 22
```

### Allow Subnets with specific protocol

UFW rules default to allowing both TCP and UDP unless a protocol is
explicitly defined. To restrict the rule to only TCP:

```bash
sudo ufw allow from 203.0.113.0/24 to any port 22 proto tcp
```

### Connections to a Specific Network Interface

If you want to create a firewall rule that only applies to a specific
network interface, you can do so by specifying “allow in on” followed
by the name of the network interface.

You may want to look up your network interfaces before continuing

```bash
ip addr

Output Example
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state
. . .
3: eth1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default
. . .
```

Let's assume `eth0` is a public network interface and `eth1` is a private network interface.

You could allow HTTP traffic (port 80) on the `eth0` public interface:

```bash
# allow your server to receive HTTP requests from the public internet
sudo ufw allow in on eth0 to any port 80
```

Or, if you want your MySQL database server (port 3306) to
listen for connections on the `private` network interface `eth1`:

```bash
sudo ufw allow in on eth1 to any port 3306
```

# Step 6 — Denying Connections

UFW is configured to deny all incoming connections. Generally, this simplifies
the process of creating a secure firewall policy by requiring you to create rules
that explicitly allow specific ports and IP addresses through.

### Deny specific connections based on the source `IP address` or `subnet`

However, sometimes you will want to deny specific connections based on the source IP
address or subnet, perhaps because you know that your server is being attacked from there. 

```bash
# For example, to deny HTTP connections
sudo ufw deny http

# Or if you want to deny all connections from 203.0.113.4
sudo ufw deny from 203.0.113.4
```

**NOTE:** UFW checks the rules **in order** and the first matching rule wins, so a `deny` rule added after an `allow` rule that matches the same traffic has no effect. To make the deny rule take priority, insert it at the top: `sudo ufw insert 1 deny from 203.0.113.4`.

### Block `outgoing` connections `from` the `server`

In some cases, you may also want to block outgoing connections from the server.
To deny all users from using a port on the server, such as port 25 for SMTP traffic,
you can use deny out followed by the port number:

```bash
sudo ufw deny out 25
```

# Step 7 — Deleting Rules

### Deleting a UFW Rule By Number

To delete a UFW rule by its number, first you’ll want to obtain
a numbered list of all your firewall rules:

```bash
sudo ufw status numbered
```

Example to delete rule number 2 that allows port 80 (HTTP) connections

```bash
sudo ufw delete 2
```

### Deleting a UFW Rule By Name

```bash
sudo ufw delete allow "Apache Full"
```

### Deleting a UFW Rule By Service Name or Port

The delete command works the same way for rules that were created referencing a service by its name or port.

```bash
sudo ufw delete allow http

# since service names are interchangeable with port numbers when specifying rules,
# you could also refer to the same rule as allow 80, instead of allow http:

sudo ufw delete allow 80
```

# Step 8 — Checking UFW Status and Rules


### At any time, you can check the status of UFW with this command:

```bash
sudo ufw status verbose
```

# Step 9 — Disable or Reset Firewall

If you decide you don’t want to use the UFW firewall, you can disable it,
any rules that you created with UFW will no longer be active. You can always
run `sudo ufw enable` if you need to activate it later.

```bash
sudo ufw disable
```

If you already have UFW rules configured but you decide that you
want to start over, you can use the reset command (it disables UFW and deletes all rules):

```bash
sudo ufw reset
```
