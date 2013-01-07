## 3.11.1 - ufw, the Uncomplicated Firewall

One of Linux's most commonly used firewall systems is `iptables`. Iptables is an extremely customizable and extensible firewalling solution, however it is very complicated to set up and maintain on its own. Luckily we have `ufw`. ufw operates by establishing certain rules in its own front end, then translating those rules into the many lines that iptables can understand and execute.

Install `ufw`:

`sudo apt-get install ufw` 

We will set our firewall to deny connections that we have not explicitly granted by default. To do this, run:

`sudo ufw default deny`
 
At this point, we can enable our firewall with:

`sudo ufw enable` 

Now it will be up to us to set specific rules (and open ports) based on the apps/servers we are running. This goes for anything operating off of this server or any other client on the internal network that requires open ports.

  
## 3.11.2 - Setting ufw Rules

To allow traffic through our firewall, we will need to allow ports through it. We can do that with:

`sudo ufw allow $xxxx` 

This will allow any system externally to use port "xxxx" on your server. This may be a good option for setting up indiscriminate servers like for email or web hosting, but what if you want to only offer services to your internal network? For example, you might host a Samba server to share and edit files on the network, but you might not necessarily want this open to the internet, even if it is password-protected. Then the following rule is for you:

`sudo ufw allow from 192.168.0.0/24 to any port $xxxx` 

This allows any system on your network (that has an address in the range of 192.168.0.0) to access port "xxxx" on your server.

Notice that UFW can also recognize a certain number of applications and server names instead of just the port number. You could run:

`sudo ufw allow Apache` 

... and UFW would open port 80 on your server to the Internet. Port 80 is the web port that Apache accepts connections on.

To list your rules, run `sudo ufw status numbered` and you will get a numbered list of rules that are currently active. To delete a rule you've already set up, grab its number from that list and run `sudo ufw delete $xx`.

  
This guide explains how to set up several different services that, depending on your usage, you may want to open up to your internal network or the internet. Remember that the ports have to be open before you will be able to use them! Here is a list of the common applications and ports you might want to allow through.

*   **Remote Connect**: port 22 for SSH, port 5900 for VNC
*   **Mail**: port 143 for IMAP, port 110 for POP; port 25 for SMTP, port 587 for SMTP submission (optional)
*   **Web**: port 80 for standard HTTP, port 443 for SSL HTTPS, port 3306 for MySQL
*   **Network Controller**: ports 53 and 953 for DNS, ports 67 & 68 for DHCP, port 5351 for NAT
*   **File Sharing**: port 21 for FTP, ports 137-139 and 445 for Samba, port 69 for TFTP, port 192 for AFP (Apple Filesharing Protocol), port 2049 for NFS (Linux Filesharing)
*   **Windows Services**: port 139 for NetBIOS, port 161 for SNMP
*   **Media Streaming**: port 3689 for DAAP (Apple/iTunes), ports 1900 and 5000 for uPnP

  
## 3.11.3 - SSH Tunnelling: Maintain Secure Access through a Closed Firewall

For applications you don't want to allow through to the Internet (if you think you are going to rarely use them away from home, or if you have significant security concerns), but you still might want to use them someday, it's good practice to use them over an SSH tunnel. You can create SSH tunnels with the following format, replacing the values where necessary:

`ssh -f -L $localport:localhost:$remoteport $remotehost -N`

The local port should be a port that is not already in use on your client computer. `localhost` can be left alone; this creates the tunnel to your client computer. `remoteport` refers to the port for whatever service you want to tunnel through. And of course `remotehost` is the address of your server on the Internet. So, for example, if you set up the following...

`ssh -f -L 9876:localhost:5900 server.mydomain.com -N`
 
... this will create an SSH tunnel for Minecraft on my computer. Simply connect Minecraft to a server located at localhost:9876 and you can use it via a remote connection, as if you were just connected to the local network.

  
### 3.11.4 - Setting up fail2ban

Our firewall is in place, which will go a long way to helping secure our system from most attack attempts. However we will go a step further by using `fail2ban`. Fail2ban monitors the logs of network-capable applications for entry attempts and repeated failures, and promptly bans the associated IP addresses for a determined amount of time. This can help dissuade and eliminate the threat posed by certain bots that like to roam the internet, testing many different attack strategies at once to try and find one that sticks. It also helps stop some DDoS attempts, which are commonly used to bring down websites and other services.

Install fail2ban with:

`sudo apt-get install fail2ban` 

Make a copy of the configuration template to the one we will actually be working from:

`sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
 
Open up `/etc/fail2ban/jail.local` in your text editor and modify the following fields:

*   **ignoreip**: set this to your internal network's subnet (like 192.168.0.0/24). This will keep fail2ban from blocking you if you trip its conditions while testing your services.
*   **bantime**: Like it says on the tin, this is the amount of time the offending IPs are banned for (in seconds).
*   **maxretry**: By default, the amount of failed attempts that should be allowed before an IP is banned.

Further down in the file, you will find a section for "Action Shortcuts." The line for default action begins with "action," and can be set here. There are three options when triggered, explained above: ban only, ban and send an email with the IP information, or ban and send an email with IP AND relevant log information.

The next section, "Jails," deals with the services we want to monitor. The entries you make in this section will depend on what services you have enabled. There are sections installed by default for ssh, Apache, ftp, postfix, etc. For some services, there are multiple jails, each that monitor for different circumstances. For example, "apache" monitors authorization attempts to your website, while "apache-php" monitors repeated failures in access to PHP files, which can often signify someone fishing for a way into your site's configuration. Make sure you select any jail that you feel you will need based on your setup.

To enable a jail, uncomment every line within it after the "[xxxx]" section, then set enabled to equal "true".

Once you have completed the configuration, you can start fail2ban with `sudo service fail2ban start`. It will immediately begin monitoring the selected services and banning repeat offenders accordingly.

With some services, errors can come up innocently yet frequently. If you create a broken link to a PHP page on your website, for example, and you have the apache-php jail enabled, you might be sending people to an error screen that can ban them if they try to refresh it too much. Keep this in mind when enabling jails AND when choosing a ban time. 

  
## 3.11.5 - More Security Tips

*   **SSH**: Make sure you've disabled root logins as well as password logins for SSH, and are only using key-based logins if at all possible. Remember to keep your keys safe on your devices!
*   **Mail**: Remember to set your `smtpd_recipient_restrictions` in Postfix, and to make `ssl = required` in Dovecot.
*   **Apache/ownCloud**: Any websites set up that handle password authentication or the transmission of even *remotely-*sensitive data should have HTTPS enforced in the settings.
*   **File Sharing and Media Streaming**: Set permissions where possible so that only your authorized network users have read/write privileges. Block services like Samba, uPnP and AFP from being used outside your network by blocking their ports at the firewall.
*   **Backups**: Encrypt any backups that are stored on your server for an extra level of protection.
*   **In General**: It's better to use SSH tunnels for applications you only use remotely on an infrequent basis than to leave them open and forget about them!

Instructions for how to enable these tips can be found in their respective sections of this guide.

  
## 3.11.6 - Further Reading

*   [UFW Questions and Answers - Ubuntu Forums][1]
*   [Fail2ban Wiki][2]

 [1]: http://ubuntuforums.org/showthread.php?t=823741
 [2]: http://www.fail2ban.org/wiki/index.php/Main_Page