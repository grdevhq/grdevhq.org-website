---
title: LXD/LXC Starter
---

**Warning**: Experimental work and post. 


First, I should mention that this post is about the most abrupt you'll find. The latest versions of Ubuntu present us with some opportunities in terms of Linux containerization. LXD is available with most of the Xenial+ distributions. So, I decided to play around and see what happens.

#### Simple LXD Setup

**(1)** Add the Ubuntu user to the lxd group


```shell
sudo usermod --append --groups lxd ubuntu
```

**(2)** Update and Install ZFS

```shell
sudo apt-get update
sudo apt-get install zfsutils-linux
```

**(3)** Initialize LXD

```shell
sudo lxd init
```

**(4)** Walk the prompts and enter what suits your scenario. Here is what I chose.

```shell
Do you want to configure a new storage pool (yes/no) [default=yes]? yes
```

```shell
Name of the storage backend to use (dir or zfs) [default=zfs]: zfs
```

```shell
Create a new ZFS pool (yes/no) [default=yes]? yes
```

```shell
Name of the new ZFS pool or dataset [default=lxd]: lxd
```

```shell
Would you like to use an existing block device (yes/no) [default=no]? no
```

```shell
Size in GB of the new loop device (1GB minimum) [default=15]: 15
```

```shell
Would you like LXD to be available over the network (yes/no) [default=no]? no
```

```shell
Do you want to configure the LXD bridge (yes/no) [default=yes]? yes
```

**(5)** Network Bridge Setup and NAT

- Accept the defaults for the bridge. Unless you want to name your IPv4 subnet. 
- Choose **Yes** to enable IPv4 NAT. 
- Choose **No** for IPv6.

Screenshots below:

![Bridge Setup](/assets/images/lxdlxc/lxdbr01.png)  
![Bridge Setup](/assets/images/lxdlxc/lxdbr02.png)  
![Bridge Setup](/assets/images/lxdlxc/lxdbr03.png)  
![Bridge Setup](/assets/images/lxdlxc/lxdbr04.png)  
![Bridge Setup](/assets/images/lxdlxc/lxdbr05.png)  
![Bridge Setup](/assets/images/lxdlxc/lxdbr06.png)  
![Bridge Setup](/assets/images/lxdlxc/lxdbr07.png)  
![Bridge Setup](/assets/images/lxdlxc/lxdbr08.png)  
![Bridge Setup](/assets/images/lxdlxc/lxdbr09.png)  
![Bridge Setup](/assets/images/lxdlxc/lxdbr10.png)  
![Bridge Setup](/assets/images/lxdlxc/lxdbr11.png)  

#### Nginx On LXD

**(1)** List containers

```shell
lxc list
```

Example output:
```shell

Generating a client certificate. This may take a minute...
If this is your first time using LXD, you should also run: sudo lxd init
To start your first container, try: lxc launch ubuntu:16.04

+------+-------+------+------+------+-----------+
| NAME | STATE | IPV4 | IPV6 | TYPE | SNAPSHOTS |
+------+-------+------+------+------+-----------+

```

**(2)** Create a Ubuntu 16.04 (Xenial) based container, named webserver

```shell
lxc launch ubuntu:16.04 webserver
```

**(3)** Hop into the container and install Nginx

Log into webserver as user ubuntu
```shell
lxc exec webserver -- sudo --login --user ubuntu
```


**(4)** Update and Install Nginx
```shell
sudo apt-get update
sudo apt-get install nginx
```


#### Set Up Port Forwarding using UFW *(Uncomplicated Firewall)*

We actually want to do useful things with our shiny new container. So let's forward some ports. Our ideal setup would look like this:

> HOST:8088 => LXD[Webserver]:80  
> HOST:8843 => LXD[Webserver]:443

**(1)** Add forwarding rules to the top of **/etc/ufw/before.rules**

Open the file for edit: (note: ain't no shame in nano) 
```shell
sudo vi /etc/ufw/before.rules
```

Example:
```shell

*nat
:PREROUTING ACCEPT [0:0]
# forward 192.30.253.113  port 8088 to 10.10.1.100:80   
# forward 192.30.253.113  port 8843 to 10.10.1.100:443
-A PREROUTING -i eth0 -d 192.30.253.113   -p tcp --dport 8088 -j  DNAT --to-destination 10.10.1.100:80
-A PREROUTING -i eth0 -d 192.30.253.113   -p tcp --dport 8843 -j  DNAT --to-destination 10.10.1.100:443
# setup routing
-A POSTROUTING -s 10.10.1.0/24 ! -d 10.10.1.0/24 -j MASQUERADE
COMMIT

```

**(2)** Stating the obvious: 

- **192.30.253.113** is a GitHub IP C Block address, **replace it with yours**. 
- **10.10.1.100** is an example local LXD Container IP, **replace it with yours**.

**(3)** Make sure your system can actually forward IPv4, edit **/etc/sysctl.conf**. 

```shell
sudo vi /etc/sysctl.conf
```

Find and uncomment:

```shell

# Uncomment the next line to enable packet forwarding for IPv4
net.ipv4.ip_forward=1

```

Reload with your change:

```shell
sudo sysctl -p
```

**(4)** Enable UFW SSH, 8088, 8843

```shell
sudo systemctl start ufw
sudo systemctl enable ufw
sudo systemctl status ufw

sudo ufw allow ssh
sudo ufw allow 8088
sudo ufw allow 8843
```

**(5)** Enable UFW 

Enable UFW, confirm with **y** when prompted. 

```shell
ufw enable
```

**(6)** Verify Rules

```shell
sudo ufw status
sudo iptables -t nat -L -nv
```


**Note:** You can choose to forward SSH and you should be able to figure out how from the above. *Remember security.*


#### The End
**Questions?** Post in Slack.    
**Changes?** Fork, git clone/branch/checkout run Jekyll, make updates then PR.