---
layout: post
title: OpenVPN through CGN
---

As I wrote in January 2018 I got a new ISP. The are routing me through a Carrier Grade NAT (CGN). So I
got no public IPv4 address. And I got a 2 € vServer with a stable 100 MBit connection.

So my idea is, to establish a connection from my pfSense to my vServer and from my mobile devices to
my vServer. So I can reach my homenet and got a tunneled connection for spooky access points :)

Recently I configured autossh as systemd user service to map a ssh-port through the CGN on the
vServer.

But back to the OpenVPN topic. In the lack of own documentation (this time I write this post FTW!),
I have to redo some steps that where already made.

I'll stick to the [Video](https://www.youtube.com/watch?v=XcsQdtsCS1U) made by Darren Kitchen and Shaonnon Morse by Hak5. Thanks to you :)

This time I use Ubuntu Server. To get the software from the mirrors just type as superuser.

```zsh
apt install openvpn easy-rsa
```

After that we use the example configuration to build on.
```zsh
gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz > /etc/openvpn/server.conf
```

Next I activated the redirection of the traffic through the VPN and tunnel the DNS through the VPN.
Then I specified some DNS-Servers.

For now I use iptables as a Firewall. Found a Gist over [Here](https://gist.github.com/Tristor/ed0f6867d2b0fa4c1f80300af6e0e12e) and took that as inspiration.

Just changed the Certificate information for the PKI to my settings (Country, City, etc) (file: vars
file in easy-rsa folder).

Generated dh248.pem with
```zsh
openssl dhparam -out /etc/openvpn/dh2048.pem 2048
```

And in the easy-rsa dir (copied the whole stuff to /etc/openvpn/easy-rsa; keep track after updates o.O)
```zsh
source vars
./clean-all
./build-ca
./build-key-server <SERVERNAME>
```

Now copy the certificate for the server and the public CA part to OpenVPN.
```zsh
cd /etc/openvpn/easy-rsa/keys/
cp server.crt server.key ca.crt /etc/openvpn
```

Now start the service with:
```zsh
service openvpn start
or
systemctl start openvpn
```

Generate client certificates with:
```zsh
cd /etc/openvpn/easy-rsa/
./build-key <CLIENTNAME>
```

Copied the client example configuration from to my homedir and edited it with the server hostname.
```zsh
cp /usr/share/doc/openvpn/examples/sample-config-files/client ~/vpn_clients/<CLIENTNAME>.ovpn
```

Now concat all the files to one unified ovpn file.
```zsh
echo "<ca>" >> <CLIENTNAME>.ovpn; cat ca.crt >> <CLIENTNAME>.ovpn; echo "</ca>" >> <CLIENTNAME>.ovpn
echo "<cert>" >> <CLIENTNAME>.ovpn; cat <CLIENTNAME>.crt >> <CLIENTNAME>.ovpn; echo "</cert>" >> <CLIENTNAME>.ovpn
echo "<key>" >> <CLIENTNAME>.ovpn; cat <CLIENTNAME>.key >> <CLIENTNAME>.ovpn; echo "</key>" >> <CLIENTNAME>.ovpn
```

Now just transport that file secure to the device and open a VPN session with:
```zsh
openvpn <CLIENTNAME>.ovpn
```

Note: Under Arch Linux the group `nogroup` does not exist by default. Rename it to `nobody`.

pit
