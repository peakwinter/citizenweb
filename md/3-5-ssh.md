Now that we have our server assembled and our OS installed, we must make sure we can get inside!

SSH is a protocol for secure communication between systems. It can be used for a wide variety of things, from executing commands on remote systems, to getting a remote terminal prompt on your local computer, to even running visual programs on a remote computer, but redirecting them to show up on your local computer's screen (X forwarding).

For the purposes of this guide, we will want to set up SSH and get comfortable with using the terminal remotely. If you are running a headless server, this is going to be your best friend.

  
## 3.5.1 - Install OpenSSH

Chances are that our Ubuntu Server came with OpenSSH already installed (that's how important it is!), but in the off-chance it hasn't, fire up your trusty-dusty package manager and install it:

`sudo apt-get install openssh-server` 

Most of the configuration for OpenSSH is stored in `/etc/ssh/sshd_config`. This is your first stop for any additional configuration options, such as denying root login or allowing public-key authentication.

The great thing about SSH is that (in most cases) it works right out of the box. First, make sure the server is running:

`sudo service ssh restart` 

Next, on your local computer, make sure you have a valid SSH client. (This is the package `openssh-client` on Ubuntu.) To test your setup, use the following command with the correct information:

`ssh $username@$ip-address` 

After this you will get a prompt asking for your password. Once you enter it, you should get a command prompt as if you were using the terminal on your server locally. Voila! Type "exit" when you want to get back to your local computer's command prompt.

  
## 3.5.2 – Securing SSH

### No Root Logins!

In its current state, your SSH is actually quite risky. Unless you laugh in the face of danger, you will want to take some steps to secure it. 

First, we will prevent root SSH logins to our server. This is a popular line of attack – people (scripts) hoping to find just that *one* server that got lax and lazy with its configuration. We won't fall for that, obviously.

Edit your `/etc/ssh/sshd_config` file and change the following line:

`PermitRootLogin	No`

... then restart your SSH server. If you need to SSH into your server and change files/configurations that require root access, then you can SSH in as your normal user and use su/sudo, just like you would if you were working directly.

### SSH Key Setup

What follows is completely optional but highly recommended. There is a way to set up a cryptographic key called an “SSH key” that will allow our computer to handle SSH connections without needing you to enter your password. There are two main reasons why people opt to use SSH keys:

1.  **Security** - Even if you have what you might consider to be a “good” password, if somehow that password is guessed or compromised than there is a lot of potential risk. With an SSH key, you can actually turn off password logins, meaning that people remotely won't even get a chance to *try* to crack your password. If they don't have your SSH key, then they're out in the cold.
2.  **Laziness** – Like I said, SSH keys allow you to SSH to your remote machine without having to use your password. So if you are someone who needs to SSH to your server frequently, it can be a pain having to enter your password every so often. Much easier to let your SSH key do the talking for you – if your computer can produce the right key, the server will never ask you for a login password.

When you create an SSH key, you are creating two files: a **private** key and a **public** key. The private key is the actual file that is used to authenticate you. The public key contains a string that the server can use to compare with the private key and verify if it's really you trying to login. The private key is the one you do not want to lose.

To create an SSH key, run the following command on your **client** machine:

`ssh-keygen -t rsa` 
This will ask you a few questions. First, go ahead and save it in the default location. Second, it's a good idea to enter a passphrase with which to unlock your SSH key. This is intended to provide a good last line of defence: should your SSH key somehow to fall into the wrong hands, they still won't be able to get into your server. (Don't worry, if you set a passphrase here, you can still set it to automatically unlock itself on your own computer via `ssh-agent`.)

After you've created your key and given it a passphrase, run the following command with the correct information in place to upload it to your server:

`ssh-copy-id $username@$servername` 
This copies your public key to an “authorized keys” list, telling your server that whichever computer SSHes in with your private key in hand can be trusted. The neat thing about this is that you can put your SSH private key on any computer you own (even your Android smartphone) and be able to gain password-less access to your server.

When you test your SSH connection, your client will automatically use your SSH key. It should only ask you for your passphrase the first time; if not, run the command `ssh-add` and it should be permanently added to your `ssh-agent`.

It should go without saying that it's very important this key be kept secure. I would recommend storing a backup on a USB key that you can hide somewhere in your home with your personal files. And if you store it anywhere else on your computer/server, like in a backups folder, make sure you store it in an encrypted archive (see the chapter on Backups for how to do that).

### Use Your SSH Key On Other Devices

If you wish to use your SSH key on...

*   ...Other Linux machines **OR** Mac OS X: Copy ~/.ssh/id\_rsa and ~/.ssh/id\_rsa.pub to the same folder on your other Linux computer. Run `ssh-add,` then voila.
*   ...A Windows computer (hisssssss): Download [PuTTY][1]. Enter your hostname/IP in the first section, then choose “SSH.” In the menu on the left, choose SSH > Auth. Browse to the location of your private key, click OK and start the session.
*   ...an Android phone: Copy your id_rsa file to your phone (in a preferably secure location) via your file transfer method of choice. Download [ConnectBot][2] from the Play Store and install it. Open the app, press Menu and choose “Manage Public Keys.” Press Menu and choose “Import,” then browse to the location of the file and choose it. Note that when you create a new connection, you can hold down the line in the list and choose Edit Server, then explicitly set that you wish to use the key for that connection. This provides the best results.

  
## 3.5.3 - Install VNC

VNC is another way to remotely gain access to your computer. Where SSH gets you into the terminal, VNC is a more direct approach. It resembles the "Remote Desktop" application on Windows systems.

This protocol is only worthwhile for servers with graphical interfaces, like the full version of Ubuntu. If you are using the Ubuntu Server we have been talking about, you will be better off sticking to SSH.

Ubuntu comes with a built-in VNC server called `vino`. It is enabled by default.

On your local machine you will need a VNC viewer. Ubuntu has one built-in named `vinagre` that will work nicely for our purposes. From the command line, enter the following with your server's IP address:

`vinagre 192.168.0.1`

When it comes to securing your VNC connection, the best way to do that is to run VNC over an SSH tunnel and block the VNC port (5900) on your firewall. We will discuss port blocking and SSH tunnelling in chapter 3.10.

  
## 3.5.4 - Further Reading

*   [OpenSSH Server - Ubuntu Server (12.10) Official Documentation][3] 
*   [ssh_config man page][4] 
*   [VNC - Community Ubuntu Documentation][5]

 [1]: http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html
 [2]: https://play.google.com/store/apps/details?id=org.connectbot&hl=en
 [3]: https://help.ubuntu.com/12.10/serverguide/openssh-server.html
 [4]: http://linux.die.net/man/5/sshd_config
 [5]: https://help.ubuntu.com/community/VNC