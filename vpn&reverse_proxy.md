# 虚拟局域网与反向代理技术

> 这是信息系统安全的一次实验课,因为觉得还挺有意思,就记录下来.

## 虚拟局域网

虚拟局域网即VPN,通常某些局域网为了安全,不与外界互联网连通,只在局域网内交流信息,比如某家公司的局域网.由于局域网使用的都是私有地址,外部网络无法访问,从而可以抵御大部分的外来攻击.可是,当员工出差的时候,往往需要用到局域网内的一些资料,所以需要使用VPN技术来实现从外网访问内网.
具体思路如下:

```
|----------------|                                   |------------------|
|    公司局域网    |                                   |     酒店局域网     |
|                |                                   |                  |
|  电脑A          |                                   |                  |
|----------------|-----------------------------------|------------------|
|  电脑B          |         |  具有公网IP的服务器  |     |    员工电脑       |
|----------------|-----------------------------------|------------------|
|  ...           |                                   |                  |
```

其中公司局域网内的电脑B和具有公网IP的服务器和员工电脑组成一个局域网,由于这三者并不是使用物理线路连接组成的局域网,故称为虚拟局域网.员工想要访问公司局域网内的资料时,向具有公网IP的服务器发起一个请求,公网IP服务器将请求的内容转发给电脑B,电脑B访问公司局域网获得资料后返回给服务器,服务器再转发给员工电脑,看起来就像员工电脑直接访问公司局域网内部资料.
由于这里有三个局域网,所以虚拟局域网中的电脑(包括服务器)都有两个IP地址,电脑B的一个IP地址是公司局域网分配的,另外一个是虚拟局域网分配的,员工电脑和服务器同理.由于服务器具有公网IP,所以不管员工电脑处在何处局域网,只要能访问到公网IP,就能访问到公司局域网.

### 实现思路

在公网服务器上通过openVPN建立虚拟局域网,通过easy-rsa建立CA,给在虚拟局域网中的节点颁发有期限的证书,虚拟局域网节点(电脑)之间通过配置证书文件建立安全的连接.

### 具体实现

参考https://www.digitalocean.com/community/tutorials/how-to-set-up-an-openvpn-server-on-ubuntu-16-04

下面把一些命令列出来

1. install openvpn

`sudo apt-get update`
`sudo apt-get install openvpn easy-rsa`

2. set up the CA directory

`make-cadir ~/openvpn-ca`
`cd ~/openvpn-ca`

3. config the CA variables

`nano vars`

```
. . .

export KEY_COUNTRY="US"
export KEY_PROVINCE="CA"
export KEY_CITY="SanFrancisco"
export KEY_ORG="Fort-Funston"
export KEY_EMAIL="me@myhost.mydomain"
export KEY_OU="MyOrganizationalUnit"

. . .

export KEY_NAME="server"

. . .
```

注意 KEY_COUNTRY只能填两个字母,不要留空

4. build the Certificate Authority

`cd ~/openvpn-ca `
`source vars`
`./clean-all`
`./build-ca`

一路回车

5. create the server certificate, key, and encryption files

`./build-key-server server`
一路回车
`./build-dh`
generate an HMAC signature to strengthen the server's TLS integrity verification capabilities
`openvpn --genkey --secret keys/ta.key`

6. generate a client certificate and key pair

`cd ~/openvpn-ca`
`source vars`
`./build-key client1`

一路回车

注意 client1只是一个标识符,根据上下文决定要不要`source var`

7. configure the OpenVPN service
Copy the Files to the OpenVPN Directory
`cd ~/openvpn-ca/keys`
`sudo cp ca.crt server.crt server.key ta.key dh2048.pem /etc/openvpn`
`gunzip -c /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz | sudo tee /etc/openvpn/server.conf`
Adjust the OpenVPN Configuration
`sudo nano /etc/openvpn/server.conf`

First, find the HMAC section by looking for the tls-auth directive. Remove the ";" to uncomment the tls-auth line. Below this, add the key-direction parameter set to "0":

```
tls-auth ta.key 0 # This file is secret
key-direction 0
```
Next, find the section on cryptographic ciphers by looking for the commented out cipher lines. The AES-128-CBC cipher offers a good level of encryption and is well supported. Remove the ";" to uncomment the cipher AES-128-CBC line:

`cipher AES-128-CBC`

Below this, add an auth line to select the HMAC message digest algorithm. For this, SHA256 is a good choice:
`auth SHA256`

Finally, find the user and group settings and remove the ";" at the beginning of to uncomment those lines:
`user nobody`
`group nogroup`

8. adjust the server networking configuration
Allow IP Forwarding
`sudo nano /etc/sysctl.conf`

Inside, look for the line that sets net.ipv4.ip_forward. Remove the "#" character from the beginning of the line to uncomment that setting:
`net.ipv4.ip_forward=1`

To read the file and adjust the values for the current session, type:
`sudo sysctl -p`

Adjust the UFW Rules to Masquerade Client Connections
Before we open the firewall configuration file to add masquerading, we need to find the public network interface of our machine. To do this, type:
`ip route | grep default`

Your public interface should follow the word "dev". For example, this result shows the interface named wlp11s0, which is highlighted below:
`default via 203.0.113.1 dev wlp11s0  proto static  metric 600`

When you have the interface associated with your default route, open the /etc/ufw/before.rules file to add the relevant configuration:
`sudo nano /etc/ufw/before.rules`

```
#
# rules.before
#
# Rules that should be run before the ufw command line added rules. Custom
# rules should be added to one of these chains:
#   ufw-before-input
#   ufw-before-output
#   ufw-before-forward
#

# START OPENVPN RULES
# NAT table rules
*nat
:POSTROUTING ACCEPT [0:0] 
# Allow traffic from OpenVPN client to wlp11s0 (change to the interface you discovered!)
-A POSTROUTING -s 10.8.0.0/8 -o wlp11s0 -j MASQUERADE
COMMIT
# END OPENVPN RULES

# Don't delete these required lines, otherwise there will be errors
*filter
. . .
```

We need to tell UFW to allow forwarded packets by default as well. To do this, we will open the /etc/default/ufw file:
`sudo nano /etc/default/ufw`

Inside, find the DEFAULT_FORWARD_POLICY directive. We will change the value from DROP to ACCEPT:
`DEFAULT_FORWARD_POLICY="ACCEPT"`

Open the OpenVPN Port and Enable the Changes
Next, we'll adjust the firewall itself to allow traffic to OpenVPN.

If you did not change the port and protocol in the /etc/openvpn/server.conf file, you will need to open up UDP traffic to port 1194. If you modified the port and/or protocol, substitute the values you selected here.

We'll also add the SSH port in case you forgot to add it when following the prerequisite tutorial:
`sudo ufw allow 1194/udp`
`sudo ufw allow OpenSSH`

Now, we can disable and re-enable UFW to load the changes from all of the files we've modified:
`sudo ufw disable`
`sudo ufw enable`
Our server is now configured to correctly handle OpenVPN traffic.

9. start and enable the OpenVPN service
`sudo systemctl start openvpn@server`
`sudo systemctl enable openvpn@server`

10. create client configuration infrastructure
Creating the Client Config Directory Structure
`mkdir -p ~/client-configs/files`
`chmod 700 ~/client-configs/files`
`cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf ~/client-configs/base.conf`
`nano ~/client-configs/base.conf`

First, locate the remote directive. This points the client to our OpenVPN server address. This should be the public IP address of your OpenVPN server. If you changed the port that the OpenVPN server is listening on, change 1194 to the port you selected:
```
. . .
# The hostname/IP and port of the server.
# You can have multiple remote entries
# to load balance between the servers.
remote server_IP_address 1194
. . .
```

Be sure that the protocol matches the value you are using in the server configuration:
`proto udp`

Next, uncomment the user and group directives by removing the ";":
```
# Downgrade privileges after initialization (non-Windows only)
user nobody
group nogroup
```

Find the directives that set the ca, cert, and key. Comment out these directives since we will be adding the certs and keys within the file itself:
```
# SSL/TLS parms.
# See the server config file for more
# description.  It's best to use
# a separate .crt/.key file pair
# for each client.  A single ca
# file can be used for all clients.
#ca ca.crt
#cert client.crt
#key client.key
```

Mirror the cipher and auth settings that we set in the /etc/openvpn/server.conf file:
```
cipher AES-128-CBC
auth SHA256
```

Next, add the key-direction directive somewhere in the file. This must be set to "1" to work with the server:
`key-direction 1`

Finally, add a few commented out lines. We want to include these with every config, but should only enable them for Linux clients that ship with a /etc/openvpn/update-resolv-conf file. This script uses the resolvconf utility to update DNS information for Linux clients.
```
# script-security 2
# up /etc/openvpn/update-resolv-conf
# down /etc/openvpn/update-resolv-conf
```

If your client is running Linux and has an /etc/openvpn/update-resolv-conf file, you should uncomment these lines from the generated OpenVPN client configuration file.
Save the file when you are finished.

Creating a Configuration Generation Script
Next, we will create a simple script to compile our base configuration with the relevant certificate, key, and encryption files. This will place the generated configuration in the ~/client-configs/files directory.

Create and open a file called make_config.sh within the ~/client-configs directory:

`nano ~/client-configs/make_config.sh`

```
#!/bin/bash

# First argument: Client identifier

KEY_DIR=~/openvpn-ca/keys
OUTPUT_DIR=~/client-configs/files
BASE_CONFIG=~/client-configs/base.conf

cat ${BASE_CONFIG} \
    <(echo -e '<ca>') \
    ${KEY_DIR}/ca.crt \
    <(echo -e '</ca>\n<cert>') \
    ${KEY_DIR}/${1}.crt \
    <(echo -e '</cert>\n<key>') \
    ${KEY_DIR}/${1}.key \
    <(echo -e '</key>\n<tls-auth>') \
    ${KEY_DIR}/ta.key \
    <(echo -e '</tls-auth>') \
    > ${OUTPUT_DIR}/${1}.ovpn
```

`chmod 700 ~/client-configs/make_config.sh`

11. Generate Client Configurations

`cd ~/client-configs`
`./make_config.sh client1`
If everything went well, we should have a client1.ovpn file in our ~/client-configs/files directory:

Transferring Configuration to Client Devices
`local$ sftp sammy@openvpn_server_ip:client-configs/files/client1.ovpn ~/`

12. install the client configuration

13. test the client
`sudo openvpn --config client1.ovpn` 

14. revoking client certificates
`cd ~/openvpn-ca`
`source vars`
`./revoke-full client3`
`sudo cp ~/openvpn-ca/keys/crl.pem /etc/openvpn`
`sudo nano /etc/openvpn/server.conf`
At the bottom of the file, add the crl-verify option, so that the OpenVPN server checks the certificate revocation list that we've created each time a connection attempt is made:
`crl-verify crl.pem`
Save and close the file.

Finally, restart OpenVPN to implement the certificate revocation:
`sudo systemctl restart openvpn@server`

