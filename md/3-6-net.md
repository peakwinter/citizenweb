For those who will be using their servers to manage their network (including as a firewall), we will now be setting up various services allowing our internal network to use the Internet and various other services hosted by our server.

## 3.6.1 - Serve Network Clients via DHCP

First, install the DHCP server from the Ubuntu package repositories.

`sudo apt-get install isc-dhcp-server` 

Now, to configure it, we will create several customized entries in `/etc/dhcp/dhcpd.conf` to handle our setup.

	default-lease-time 432000;
	max-lease-time 604800;
	option routers 192.168.0.1;
	option domain-name-servers 192.168.0.1;
	option broadcast-address 192.168.0.255;
	option subnet-mask 255.255.255.0;
	option domain-name "$home.local";

	subnet 192.168.0.0 netmask 255.255.255.0 {
		range 192.168.0.10 192.168.0.50;
		host $myhost {
			hardware ethernet xx:xx:xx:xx:xx:xx;
			fixed-address 192.168.0.x;
			option host-name "$Myhost";
		}
	}

Now let's walk through these lines and figure out what each of them does.

*   **default-lease-time** and **max-lease-time** govern how often your computers will check back with the server to have their IP address assignment renewed. The figure is in seconds. In the majority of cases, you can set this to be a somewhat long time and there will be no issues. If you set the leases to be too short, it may impact your network performance. 432,000 seconds equals 5 days.
*   **option routers** and **option-domain-name-servers** needs to point to your server's static IP address, that you gave it in the Server Installation chapter.
*   **option broadcast-address** is for the internal network broadcast address. The last octet (set of numbers) should always be 255. If your network is in the 192.168.1.x range, then change the 1. Otherwise it should be left alone.
*   **option subnet-mask** should be left at its default, 255.255.255.0. If you need a different one, it's likely because you have a huge network with hundreds of computers; if that's the case, then you shouldn't be following this guide anyway :)
*   **option domain-name** should match what you chose as your internal domain name. In most cases, "home.local" will suffice.
*   **subnet 192.168.0.0 netmask 255.255.255.0 {** begins the section that outlines the internal network we are now setting up. The first IP address (192.168.0.0) combined with the second number (255.255.255.0) means that all of our clients will have IP addresses that begin with 192.168.0, AND that we can add any number at the end of that from 0-254 for network clients.
*   **range 192.168.0.10 192.168.0.50** is important, because it tells the DHCP client how many addresses in the 192.168.0.0 block it can claim as its own and assign to clients. Its usually a good idea to have a bit more than you need here; as you are not likely to have over 200 machines on this network, than you won't be needing to worry about space.
*   The next nested section (**host $myhost**) is optional. If you want one of your computers to always receive the same IP address via DHCP, which is convenient for diagnostic purposes and is recommended for any other servers running on your network. Replace the hostnames listed here with what they should be for that computer. Set the MAC address to the network adapter that the computer will connect from. (On Linux-based systems you can usually find the MAC address by running `ip addr`.)
*   And finally, don't forget to close out all the open sections you opened with "{" with a corresponding "}"!

Once your configuration is in order, start the server with `sudo service isc-dhcp-server restart`. Your devices will now be able to communicate with each other on your network. But don't get too excited yet! They still won't be able to get internet access. For this, we will need to set up a gateway and NAT forwarding with iptables, then we will set our server to handle DNS requests.

  
## 3.6.2 - Give Clients Internet Access with iptables

The next step is to enable your server as an Internet gateway, so that it will share its connection to devices connected to the internal network. To do this, we will be using the iptables firewall system.

	sudo iptables -A FORWARD -o eth0 -i eth1 -s 192.168.0.0/24 -m conntrack --ctstate NEW -j ACCEPT
	sudo iptables -A FORWARD -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
	sudo iptables -t nat -F POSTROUTING
	sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
	sudo iptables-save | sudo tee /etc/iptables.sav
	sudo sh -c "echo 1 > /proc/sys/net/ipv4/ip_forward"

"-o eth0" should match your outside interface (connected to your modem), and "-i eth1" should match your inside interface, connected to your hub or access point.

Set your iptables configuration to load at boot by editing `/etc/rc.local` and adding the following line:

`iptables-restore < /etc/iptables.sav`

Finally, edit `/etc/sysctl.conf` and uncomment the line that reads `net.ipv4.ip_forward=1`.

And with that, our iptables configuration should be working. We will work more with iptables in the chapter on firewalling and security, 3.10. At this point your devices should now be able to ping IP addresses that are on the internet, and view internet sites via IP addresses. But the final piece of the puzzle comes in handling DNS requests.

  
## 3.6.3 - Set Up a Local DNS Server

In brief, DNS is the method that the Internet uses to translate IP addresses to the domain names we are all used to typing in our browsers. We know that every internet server has at least one IP address, and this is how it can be "found" online. And DNS is what is used to give these addresses a human-readable name.

Our server will be set up for DNS for two purposes:

*   **Caching**: For every page request made to the Internet from one of your computers, the server will keep a cache of its location data. You may notice that the first time you view a site, it is often slower to load than the subsequent times you visit it. This is subsequently due to your computer "seeking" the address of the server the first time; every time after that, it will remember where it went before. Setting your server to act as a DNS cache locally should improve internal network performance overall.
*   **Internal Authority**: This DNS server will keep track of the device names on our network, and allow other devices to be able to find them by those names. So if you want to SSH to your computer in the other room, you can do so by running `ssh ComputerName` instead of having to keep track of its IP address at any given time and running `ssh 192.168.0.?`.

The DNS server we will use is called BIND. Install it by running `sudo apt-get install bind9`.

To configure BIND as a caching nameserver, edit `/var/lib/bind/named.conf.options` and change the following lines:

	forwarders {
		x.x.x.x;
		x.x.x.x;
	};

The x.x.x.x lines should match the Primary and Secondary DNS addresses given to you from your Internet Service Provider. If you do not have any or do not know what they are, you can use 8.8.8.8, which forwards to Google's public DNS servers.
  
Now we will set up our DNS server to act as our internal network's authority. This comes via setting up two zonefiles. Create a file named `/var/lib/bind/db.home.local`. (Change the trailing "home.local" to whatever you decided your internal domain would be earlier.)

Enter the following in this file, replacing the $ values where appropriate EXCEPT leave the $TTL and $ORIGIN as they are:

	$ORIGIN .
	$TTL 86400
	$home.local	IN SOA	$myserver.home.local. $username.home.local. (
					2012112301 	; serial
					28800		; refresh (8 hours)
					14400		; retry (4 hours)
					2419200		; expire (4 weeks)
					86400		; minimum (1 day)
					)
				NS	$myserver.home.local.
				MX	10 $myserver.home.local.
	$ORIGIN home.local.
	myserver		A	192.168.0.1
	laptop			A	192.168.0.2
	workstation		A	192.168.0.3
	phone			A	192.168.0.4
	xbox			A	192.168.0.5

The third line (starting with "home.local") should feature your internal domain. The next bit (myserver.home.local.) should reflect your server's hostname with the internal domain and a "." appended to the end. The last bit on this line (username.home.local.) is actually an administrative email address - change this to match the email you want to use for this field, making sure there is a "." in the place of the "@", and a "." at the end of it all.

The NS and MX lines should point to your server's hostname and internal domain. This is used to designate the server as the internal domain's nameserver and main mail server.

The repeated entries below the second $ORIGIN tag are individual records for devices on the network. These are called "host entries." Remember when, in our DHCP configuration, we had the opportunity to reserve specific addresses based on the MAC addresses of our devices? These same entries should be repeated here, with the accompanying "A" tag in the middle. Now we don't need to add entries for every possible device we will have on our network here: in the next section we will have DHCP do this for us. But it is a good idea to include your server in this list, as well as anything you've given static or reserved IP addresses.

Whenever you change a zonefile, you **must** increase its serial number. Many people use the date in YYYYMMDD format, then a couple digits marking the number of the change you've made.

There are many other kinds of host entries you can make here; for information on them see the BIND links in the Further Reading section.
 
Now for every DNS zonefile we establish, we must have a corresponding "reverse DNS zonefile." This is fairly simple to do; create a file called `/var/lib/bind/db.192` and insert the following, replacing the $ values where appropriate EXCEPT leave the $TTL and $ORIGIN as they are:

	$ORIGIN .
	$TTL 86400
	0.168.192.in-addr.arpa	IN SOA	$myserver.home.local. $username.home.local. (
					2012112301 	; serial
					28800		; refresh (8 hours)
					14400		; retry (4 hours)
					2419200		; expire (4 weeks)
					86400		; minimum (1 day)
					)
				NS	$myserver.home.local.
	$ORIGIN 0.168.192.in-addr.arpa.
	1			PTR	myserver.home.local.
	2			PTR	laptop.home.local.
	3			PTR	workstation.home.local.
	4			PTR	phone.home.local.
	5			PTR	xbox.home.local.

The "0" in "0.168.192.in-addr.arpa" refers to the third octet in your network's IP subnet. It assumes your network operates on the 192.168.0.0 range. If it is otherwise, update this number accordingly.

Now a lot of these options are customized in the same way they are in the first zonefile we made, but we can see a pretty important difference when we get down to the host records. They are in reverse order. The last octet of the IP address for each device (e.g. the "1" in "192.168.0.1") is placed first, followed by the "PTR" (pointer) flag, then the fully-qualified hostname with internal domain appended at the end. Remember that you only need to create records here if you created them in your first zonefile, and you don't need to create records for *every* device on your network.

To activate these zonefiles for use in BIND, edit `/etc/bind/named.conf.local` and add the following lines:

	zone "home.local" IN {
		type master;
		file "/var/lib/bind/db.home.local";
	};
	zone "0.168.192.in-addr.arpa" {
		type master;
		file "/var/lib/bind/db.192";
	};

*Whew,* are you still with me? DNS setups can be a real headache, but if you've made it this far with your sanity intact, then you are almost ready to reap the rewards!

Start up bind with `sudo service bind9 restart`. At this point, your clients should be able to connect to the Internet using regular ol' domain names like usual. Hooray!

  
## 3.6.4 - Allow DHCP to Update DNS Entries

Now we can not only use the Internet on our internal network, we can also communicate with our static servers/hosts using their proper names. But what if you want to reach other devices by their hostnames? Say you have a friend come over that's bringing his laptop, and you want to set up a fileshare on it and to reach that share via his laptop's hostname. For that, we can allow our DHCP server to fetch these names and update our network's DNS records accordingly. This is done by providing a secure socket for the DNS and DHCP servers to communicate on.

First, change the owner of your zonefiles to let BIND be able to edit them at will:

`sudo chown bind:bind /var/lib/bind/*`

Now we will generate a key that will allow the two programs to communicate securely between each other.

`sudo cat Kdhcp_updater.*.private | grep Key`

Copy the output or write it down; we will need it soon. Open up `/etc/bind/named.conf.local` again and add the following lines:

	key DHCP_UPDATER {
	    algorithm HMAC-MD5.SIG-ALG.REG.INT;

	    Important: Replace this key with your generated key.
	    Also note that the key should be surrounded by quotes.
	    secret "asdasddsaasd/dsa==";
	};

While in `named.conf.local`, add the following line inside the brackets for each zone you have declared there:

`allow-update { key DHCP_UPDATER; };` 

So we are set up on the DNS end, now let's give DHCP the other end. Edit `/etc/dhcp/dhcpd.conf` and add the following to the very top of the file:

	ddns-domainname "$home.local.";
	ddns-rev-domainname "in-addr.arpa.";
	ddns-update-style interim;
	ignore client-updates; 
 
Next, add the following before the "subnet" section:

	key DHCP_UPDATER {
	    algorithm HMAC-MD5.SIG-ALG.REG.INT;

	    Important: Replace this key with your generated key.
	    Also note that the key should be surrounded by quotes.
	    secret "asdasddsaasd/dsa==";
	};

	zone home.local. {
	  primary 127.0.0.1;
	  key DHCP_UPDATER;
	}

	zone 0.168.192.in-addr.arpa. {
	  primary 127.0.0.1;
	  key DHCP_UPDATER;
	}

Accordingly, we will allow the DHCP server to write to its files:

`sudo chown dhcpd:dhcpd /etc/dhcp/dhcpd.conf`
 
Restart the servers with `sudo service bind9 restart` and `sudo service isc-dhcp-server restart`, and it's done!

Don't forget to remove the key file that we created, `Kdhcp_updater.*`.

From now on, if you want to make manual changes to your BIND DNS zonefiles, you will need to "freeze" them first. Freeze it with `sudo rndc freeze home.local.` and then you are free to make your edits. Once completed, "thaw" the zonefile again by running `sudo rndc unfreeze home.local.` And of course, don't forget the "." at the end!
  
## 3.6.5 - Further Reading

*   [DHCP (Ubuntu Documentation)][1]
*   [BIND Configuration (Ubuntu Documentation)][2]
*   [Internet Connection Sharing (Ubuntu Documentation)][3]
*   [DNS Record Updates via DHCP - Lani's Weblog][4]

 [1]: https://help.ubuntu.com/12.04/serverguide/dhcp.html
 [2]: https://help.ubuntu.com/12.04/serverguide/dns-configuration.html
 [3]: https://help.ubuntu.com/community/Internet/ConnectionSharing
 [4]: https://lani78.wordpress.com/2012/07/23/make-your-dhcp-server-dynamically-update-your-dns-records-on-ubuntu-12-04-precise-pangolin/