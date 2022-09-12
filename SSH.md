# SSH ! 

Yes the super useful tool for remote shell and a lot a lot more!

## SSH port forwarding

Technically this is to pass other traffic through the connection that ssh have established. 

Both type of port forwarding have the syntax: 

### Forward port forwarding

When you want to access something as if you are accessing it as the remote machine, then this is helpful.

For example: if the web-server's webpage is only accessible through localhost (many software's management console only allow localhost access for security)

If any process try to connect to port 8080 of local machine, this connection will be forwarded to the remote host's 80 port. So accessing `localhost:8080` is the same as accessing `remote-web-server:80`. And to the remote website, this connection looks like it's from a port inside itself.
```
ssh -L 8080:localhost:80 remote-web-server
```


Or if this remote host is in a private lan, which have another machine that's not normally accessible. We can ssh to "private-lan-gateway" machine, this machine also connect to private network of "192.168.1.0/24". We want to access "192.168.1.10:2345" port. 
```
ssh -L 8080:192.168.1.10:2345 private-lan-gateway"
```

```
-L [bind_address:]port:host:hostport
-L [bind_address:]port:remote_socket
-L local_socket:host:hostport
-L local_socket:remote_socket
```
`bind_address` if not specified, will act according to `GatewayPorts` setting (default to `localhost` only). Empty or "*" bind_address means port is available for all interface.

### Reverse port forwarding

This is in literal term, a reverse action of above. However, this is generally not used and if used wrong, could become a security risk. This is way some system admin will turn of port forwarding to prevent using reverse tunnel. 

Basically, using reverse tunnel mean accessing a port of remote machine is the same as accessing a port on local machine.

```
[bind_address:]port:host:hostport
[bind_address:]port:local_socket
remote_socket:host:hostport
remote_socket:local_socket
[bind_address:]port
```
if `bind_address` is not specified, it's loopback device (localhost). 

Example: When accessing `remote-gateway:80`, the traffic actually will access `localhost:1234`

```
ssh -R 80:localhost:1234 remote-gateway
```

This is specially useful for machines behind firewall or NAT to expose port onto a publicly reachable server, to make it access-able.

## Dynamic forwarding 

In this case, the forwarded port is acting as a SOCKS server port.

WIP

===

## autossh 

This is the tool that establish an ssh connection and auto maintain the connection. This is often used for setting up a long term remote port forwarding that auto re-connect if connection is broken. 

Similar feature could be done using systemd-unit which auto re-start if the process ends or fail. However, autossh is more specifically built and have some other helpers.

```
autossh -M 56565 -N -R localhost:12345:localhost:22 my-public-server
```

* -N is for ssh, "Do not execute a remote command" this is for port-forwarding.
* -M 56565: setups a pair of monitor port, autossh will establish extra connect using port 56565 and 56566 to monitor if the connection is broken. 

The above example setup a tunnel from `my-public-server:12345` to `localhost:22` which is the ssh port on local machine. Note only a localhost connection from the remote server will be able to access port 12345. This is strongly advised for security purpose. Since public server are under constant attach, by setting up this way, attacker will needs to brute force ssh security twice before connect to your local machine.

===

## ssh in systemd unit

A common trick to make a machine in private network publicly available is to do reverse tunnel. To make it auto launch the tunnel on start, a systemd unit is commonly asked for. 

However, there are some catches when running ssh in systemd file. 

### User difference

When testing the ssh connection manually, everything will run under current user, say `machine-user`. However, a systemd unit will launch as root (if not specified user option). This case the following problems:

#### Identity file:

Usually we first setup a connection to the server, then do ssh-copy-id, so we no longer need to type password. However the copied identity file is per user, e.g. `/home/machine-user/.ssh/id_rsa`. SSH authentication will certainly fail when running as root, since root's identity file is not pushed to remote server. And most likely the root user doesn't have an identity file.

The solution to this is to specify the id file in the ssh command `-i /home/machine-user/.ssh/id_rsa`.

#### Authenticate remote host

It's a common thing we will be prompted to verify remote-host's pub-key on first connect. Then this pub-key is stored in `~/.ssh/known_hosts`. This file is also per-user, and not sharable. So when running as root, you often get `read_passphrase: can't open /dev/tty: No such device or address` if you can't connect and turned on debug (`-v` for ssh debug)

One solution for this is simply not check the remote's key. However this is dangerous and should be avoid.

--- 

The general solution to both of the above problem is to either login to root user, test out the connection. Or run the systemd unit as a existing user.
