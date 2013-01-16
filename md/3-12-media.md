## 3.12.1 - Setup File Shares via Samba and NFS

The first step to setting up a file server, whether its for your local network or for remote access, is to decide upon a method for sharing that works with your desired configuration. This guide will explain three different systems, each easy to set up but used for different purposes. You can set up all three, or any combination thereof.


### Samba

Samba is a file sharing server that allows your Linux server to interact with Windows clients on your network. It also easily works with Mac OS X clients. If your home network includes any devices that do not run Linux, and you want those devices to be able to interact with your files stored on the server, it is usually a good idea to set up Samba.

All you need to set up Samba is `sudo apt-get install samba`. After this, you will be ready to add a new share.

Open up `/etc/samba/smb.conf` in your text editor and scroll to the bottom of the file. You will want to add a section that looks like this, changing the fields where appropriate:

	[share]
    		comment = My File Server Title
    		path = /path/to/my/shared/folder
    		browsable = yes
    		guest ok = yes
    		read only = no
    		create mask = 0755

The `guest ok` field above will change if people can use your file server without logging in with a password. The `read only` field will change if someone logged in to your server is able to change the files at all, or just to read from them.

Once this is complete, you merely need to restart your Samba server:

`sudo service smbd restart`

Now, to connect, open up your file browser on a computer connected to your network.

* Windows: In the Address Bar of your file browser, enter `\\$servername\share`. If you want to mount this share permanently like a hard drive, right-click Computer and choose "Map Network Drive." Put the above address in as the folder, and choose a drive letter.
* Mac OS X: In Finder, click Go > Connect To Server. Insert the address "smb://$servername/share".

It is not a good idea to open your Samba server to the world. For sharing with others, use FTP or a separately-installed service like ownCloud. Use a firewall like ufw to block Samba's ports externally, or to only allow it on your local network. The ports used for Samba are 139 and 445.

The easiest way to improve the security of this setup is to require users to log into your server via user accounts. You can easily do this via PAM, which is the software that runs your Linux server's user accounts and logins. To do this, run `sudo apt-get install libpam-smbpass`. Then go back into your Samba configuration file and set `guest ok` to equal "no". Restart your Samba server with `sudo service smbd restart`.

With this, you can restrict your file access to only users that have accounts on your server.


### NFS

NFS is a network filesharing system designed for Linux systems. It is a faster and easier option than Samba if you are only planning to use your fileserver with Linux-based computers.

To install NFS, run `sudo apt-get install nfs-kernel-server`. Then, to add a new share, edit the `/etc/exports` file, and add lines based on the following configuration.

	/path/to/shared/folder  *(ro,sync,no_root_squash)

The first 'ro' indicates if this share should be read-only or writable to clients that connect to it. To make it writable, replace 'ro' with 'rw'. If you want to restrict this share to be available only to specific computers on your network, replace the '*' with those computer hostnames, IP addresses or IP range/subnet.

After adding your shares, start your server with `sudo service nfs-kernel-server restart`.

To connect to these systems from your Linux computer, go to the Terminal and run `sudo mount $servername:/path/to/shared/folder /path/to/local/mount`. You will need to set up a folder on the computer to act as the local mount point. After this, you can go to that folder path and use it, just as if it was a local folder.

Much like Samba, you shouldn't open your NFS server to the world. For sharing with others, use FTP or a separately-installed service like ownCloud. Use a firewall like ufw to block the NFS ports externally, or to only allow it on your local network. NFS uses port 2049 for its connections.


## 3.12.2 - Stream Music/Photos/Video via uPnP

Once we have our file servers set up, that's all well and good, but it does not let us seamlessly stream our content. That is one of the great benefits of having a server act as a NAS (network attached storage) device: being able to stream your media from various devices around your home. uPnP is one of the mechanisms that can be used to achieve this. With it, you can seamlessly stream your music, photos or video with different platforms.

This guide will use a simple uPnP server called minidlna. It can stream to uPnP or DLNA compatible clients.

Install minidlna with `sudo apt-get install minidlna`. Configuration can be adjusted by editing the file `/etc/minidlna.conf`. The important lines to change based on your configuration are as follows:

1. **network_interface** - If you have multiple network interfaces on your device, make sure they are listed here. Or, only list the network interfaces you want to serve. If you have one dedicated to the internal network and one facing your modem, you can easily prevent external access this way.
2. **media_dir** - This will point your minidlna server to the folders containing the media you want to serve. An example: `media_dir=A,/home/user/music` will set minidlna to share the 'a'udio listed in these folders. V is used for video and P is used for photos. You can simply put `media_dir=/home/user/folder` if you have one folder with multiple types of media to stream.
3. **friendly_name** - The name of your server that will be broadcast to clients.
4. **album_art_names** - If you have a specific naming convention for the album art in your music library, like "Cover.jpg", put it here. minidlna will set these files apart and use them for album covers in the library view.

With this, start your minidlna instance with:

	sudo service minidlna restart

... then connect with your client of choice, and enjoy the streaming experience! 

Remember to block the uPnP port to the outside world via your firewall if you don't want anyone and everyone to have access to your media collection! uPnP uses ports 1900 and 2869.


## 3.12.3 - Stream to your Apple Devices with DAAP

If you have an abundance of Apple devices in your home, or are just attached to your iTunes library more than anything else, you can use DAAP streaming instead of (or in addition to) uPnP. The DAAP setup is very similar to that of uPnP, but it will instead allow you to stream directly to iTunes using its "Home Sharing" functionality. There are also DAAP clients for Windows, Linux and Android.

We will install a DAAP server called forked-daapd. Run `sudo apt-get install forked-daapd`. To configure, we will edit the file `/etc/forked-daapd.conf`. You will need to set the 'directories' line to match the path to your music folder. You also may want to change the 'name' line to match what you want to show to your clients.

After this, restart the server by running `sudo service forked-daapd restart`. Then fire up iTunes, enable Home Sharing, and your server will show up in the sidebar!

Remember to block the DAAP port to the outside world via your firewall if you don't want anyone and everyone to have access to your media collection! DAAP uses port 3689 for its connections.


## 3.12.4 - Further Reading

* [Securing a Samba Print and File Server - Ubuntu Documentation][1]
* [NFS HOWTO - Ubuntu Community Documentation][2]
* [minidlna - Ubuntu Community Documentation][3]
* [How to set up forked-daapd - Ubuntu Forums][4]

 [1]: https://help.ubuntu.com/10.04/serverguide/samba-fileprint-security.html
 [2]: https://help.ubuntu.com/community/NFSv4Howto
 [3]: https://help.ubuntu.com/community/MiniDLNA
 [4]: http://ubuntuforums.org/showthread.php?t=1354196
