## 2.4.1 - Secure Your Web Browsing

### Encrypt Your Connections with SSL/TLS

The first step to take in assuring your web browser's security is to make sure every connection possible is made over SSL. SSL should be familiar to you by now -- every time you log into your bank account, for example, you should see a little "https" in your address bar with a little green check-mark or a lock symbol. This means that your personal connection data is being encrypted between you and the server you are communicating with. Your username, your password and other form data on the bank's website cannot be "snooped" on by anyone else on your network.

Most sites that require logons will have SSL capability. The problem is that SSL is often not equipped by default on sites that don't handle financial information. This means that sites like Facebook might still be handling your connections over regular unencrypted HTTP by default.

To change that, there are browser plugins that you can use to enforce SSL by default for any site that has it enabled. The Electronic Frontier Foundation has developed a tool named "HTTPS Everywhere" which enables HTTPS by default on many popular sites and services that have HTTPS available. It can be paired with another plugin called HTTPS Finder or KB SSL Enforcer, which searches other sites that appear to have HTTPS installed, and passes them to HTTPS Everywhere.

HTTPS Everywhere can be downloaded for Firefox and Chrome (Chromium) by [visiting the website].[1] In Firefox, click the Install link, then click Allow, then click Install Now. You will need to restart Firefox before the plugin will be enabled. For Chrome, click the Install link, click "Add to Chrome," then click "Add."

To install HTTPS Finder in Firefox, open your browser than click Tools > Add-ons. Click "Get Add-ons," and search for HTTPS Finder. 

To install KB SSL Enforcer in Chromium: go to the [Chrome Web Store][2] and search for "KB SSL Enforcer" under "Extensions."

And in the future - for EVERY site that requires a login, make sure that the address shows as HTTPS on the login page! If not, go into your settings and there will often be an option to use HTTPS by default tucked away somewhere.


### Block Monitoring Scripts

Once your connection data is secured, there is now the matter of tracking scripts. Everywhere on the web, on nearly every commonly-used website nowadays, there are "trackers." Trackers come in many forms but the most frequent way is via tracking cookies that are transparently downloaded to your computer, or active scripts that run on pages that report home with specific data. The idea of web tracking is a very broad concept but I will give two of the most common examples below.

1. Google Analytics -- This is a piece of software run by Google that gives webmasters a huge amount of data on each visitor. It can pinpoint everything from their approximate geographic location, to details about their computer and its operating environment, to how long they spent on the site, what pages they looked at, and other details. It can even tell what site you came from to get to a certain location, and what site you visit when you leave. Some of this information can be discerned simply by looking at one's server logs, but Analytics renders this much easier. Not everyone has a problem with this data being in the hands of webmasters, but the fact that it is also kept by Google can be more than worrying.

2. Facebook -- Facebook's "Beacon" scripts are everywhere on the web. Anywhere you see a dynamically-loaded "Like" button, on a blog article or on a company's website, this information is sent to Facebook. What's worse is that if you are logged into Facebook (which most people are, even if it is not actively open in their browser), Facebook will be able to match your profile with your Web browsing habits.

If you'd rather not surf the web with strange corporations watching your every move, you certainly aren't alone. Thankfully there are browser plugins that can help! There is one in particular called Ghostery that is very effective at blocking trackers you don't want to see, while still giving you the power to enable the ones you may find useful from time to time. If you like the ability to click the "like" button from time to time, for example, but don't want Google Analytics to track you, you can manually allow the Facebook tracker in Ghostery's easy-to-search database.

To install Ghostery in Firefox or Chrome, go to the browser's Add-ons section and search for Ghostery. Once it is installed, it will ask you what sites to block. My advice is to choose "Select All" to block trackers by default. Then, later on, if you find one you need to use, you can go back into your Add-on settings and uncheck the box next to that tracker's name.

![][3]

With Ghostery you can also pause all tracking easily. If you find a website doesn't quite work properly without its trackers, click the Ghostery button in your browser window, than click the "Pause" button. Then refresh the page and try the functionality again. Just don't forget to press "play" again when you are done!


### Encrypt Your Browsing with Tor

There is another option, perhaps the most advanced one yet when it comes to completely anonymous Internet surfing. That option is Tor. Originally developed by the US Government, Tor is a type of "onion router" that routes your internet traffic through a complicated layered system. There is much to say about Tor and a lot of explaining behind how it works. If you are interested in it, you can visit the Tor Project [on its website.][4]

If you would like to use Tor for anonymous browsing, it's easy to do so. However we will not be installing Tor using the Ubuntu package repository, like has been done in the past. Since Tor updates are considered very important for stability and security reasons, we want to make sure that we are getting them on time. For this, we will patch Tor's custom update server into our Ubuntu installation. That way, whenever we run `sudo apt-get update` and `sudo apt-get upgrade`, Tor will update itself whenever a new version is available.

First, run `cat /etc/debian_version` to check your Ubuntu's version codename. If you are using 12.04, the codename is "precise." Next, open up `/etc/apt/sources.list` and add the following line, with your version codename in the appropriate place:

`deb     http://deb.torproject.org/torproject.org $codename main`

Next, add the Tor project's GPG key, used to sign its packages and verify their authenticity:

	gpg --keyserver keys.gnupg.net --recv 886DDD89
	gpg --export A3C4F0F979CAA22CDBA8F512EE8CBC9E886DDD89 | sudo apt-key add -

Then the final few commands:

	sudo apt-get update
	sudo apt-get install deb.torproject.org-keyring
	sudo apt-get install tor

From this point on, Tor is installed and running on your system. But before you can use it, you must configure your browser to use it. You can do this manually of course, but we will use the most convenient and automatic method -- via a browser plugin. Download the [Tor Browser Bundle found here][5]. Make sure you download the Linux version, and the architecture that corresponds to your computer. If you don't know your architecture, run `uname -m`. If you get "x86_84" as a response, you have a 64-bit system; if you get "i386" or "i686" as a response, you are using a 32-bit system.

After downloading the package, run the following to extract it and install:

	tar -xvzf tor-browser-gnu-linux-*.tar.gz
	cd tor-browser_*
	./start-tor-browser

This will start a specially-patched version of Firefox that has Tor enabled. You can create a shortcut to the `start-tor-browser` script on your desktop or in the sidebar, and you will be able to launch your Tor browser whenever you want. You will need to reinstall your Add-ons in this Tor browser, and you will not be able to use your old browser (Chrome or Firefox) if you want to have the protection of Tor. However the Tor browser is based on Firefox, so any plugins that work for Firefox should also work for the Tor browser.

Before you start using Tor, there are some things you should be aware of before you start surfing! Make sure you [check out the list][6] and are aware of what they might mean for you.


## 2.4.2 - Secure Your Email

### Encrypt Your Connections With SSL/TLS

Just as it is important to use websites that enable SSL, you will want to do the same with your email connection. If you always use your email in a browser, like Yahoo Mail or Gmail, you don't need to worry about this. But if you use a third-party client like Thunderbird, there are settings you should make sure are set.

In Thunderbird, click Edit > Account Settings. On the left side of the window, you will see an expanded list of email accounts. Choose the one you want to set for SSL and click "Server Settings." Connection Security should be set to STARTTLS. Then click "OK." If you choose this and your email does not send or receive, select SSL/TLS in this field instead and try again

If you experience complications enabling SSL in your email client, your email provider will give you instructions on how to do so in its Help section. It may have a different server name or port configuration for you to enter here.


### Encrypt Your Messages with PGP

PGP is the standard for email encryption nowadays. It allows you to seamlessly encrypt mail messages to people and have them just as easily decrypt them upon receipt. You might send a full message to someone, and if anyone that might come across your message happens to open it without your key, this is all they will see:

	-----BEGIN PGP MESSAGE-----
	Charset: ISO-8859-1
	Version: GnuPG v2.0.19 (GNU/Linux)

	hQEMAyL1sE8aLy0uAQf9G12ng+ijfKmMEyInN6iauYaP6ITIrzOtTK+ZiEHDc1oA
	eKZwh4zgt1O6ll1AUU+nYC1WCTMKP1cIU0yOqp1INFl9ZNtn7qNneUcmYmfyaDBA
	Tpz15qXiM5mVMCrK82e1XtGLRKko76In4oh8WEVxISZhw4ATD+Vx0jXqQR6HUkeK
	1sr4a+OTjSZlT+iTYy0Q2PQjSLMp5xKyjoYt9ArxOQDBbznwRcwfRIMzUnCf2Q87
	uayssbp5HmnpZj8Izgm7/EehrkQfnklhAvgGPrNk/d8oD+PK2D4h3plAqpSres6O
	7k6OAehppAJt/TKUYoNZeM6qCeBOrRQohuSmGg3kNNLpAUJOONXIYFavuc2Iyb+p
	hyBRSxrcZJ/e2PN/Xx7i6Ki/P3347foZ0/GaVpUrwP9MQJLjawR/cVEEBY2lar4t...

An indecipherable mess is all that awaits them.

For PGP to function properly, you must generate a "keypair" for yourself, and you must have the public key for your chosen recipient. Let's go through the steps.

We will use Thunderbird and it's Enigmail plugin to handle our email encryption and decryption. In Thunderbird, click Tools > Add-ons, click Get Add-ons, then search for and install Enigmail. Once the plugin is installed and Thunderbird has restarted, click the OpenPGP menu, then choose the Setup Wizard.

![][7]

Click Next, then choose the email accounts you want to use encryption with. (Remember that you will have the choice whether or not to encrypt each message, so you don't have to worry about making everyone you know get PGP keys if you don't want to encrypt your emails to them!)

Click Next again, and follow the rest of the wizard. It explains well the steps and options you need to choose, and it also helps you automatically generate a PGP key.

Now, once this is complete, you have the option of submitting your public key to a keyserver. A keyserver is like a search engine for people's public keys -- if you have someone you wish to communicate with, you can import their key from a public keyserver without them needing to give you their key directly. This does not reduce the security of your keys, as the message can only be decrypted by the specific recipient anyway. You are not required to upload your public key to a keyserver; if you choose not to, you will need to keep your messages signed with your PGP signature (which Enigmail usually enables by default), or you will need to export a copy of your public key to an .asc file and give that to your conversation partner.

To upload your key to a public keyserver with Thunderbird, click OpenPGP > Key Management, then right-click your key and choose Upload Public Keys to Keyserver. It doesn't matter which server you choose at this stage, as they all will share their data with each other.

To find someone else's public key on a keyserver, open up Key Management then click Keyserver > Search for Keys. Type in the email address of the person you want to email, then check the box next to their name. If their name doesn't come up in the list, you can import a public key that they give you in .asc format by choosing File > Import Keys From File. It is usually a best practice to use a key that is given to you from someone rather than using a public keyserver, if you trust them.

Remember that if you have uploaded your public key to a keyserver, you are pretty much locked into using that key. If you ever lose your keyfiles or want to change keys for some reason, you will need to generate and upload a revocation certificate. This is done to ensure trust, and the knowledge that the keyholder really is who he purports to be via their name and email address.

After this, you can write encrypted emails to whoever you want, provided you have imported their public key! If you chose to automatically encrypt your messages in the Setup Wizard, you don't have to set anything; if not, you can click OpenPGP > Encrypt Message in the New Message window to write an encrypted message. Once you click "send" and enter your password, the message will automatically be encrypted. When you receive an encrypted message from a conversation partner, Thunderbird will automatically ask you for your password, and will decrypt the message for your viewing. The message will remain encrypted so you will need to enter your password each time you wish to read it.


## 2.4.3 - Secure Your Chat Applications

### Encrypt Pidgin Chats with OTR

If mail is a bit too slow for your taste and you prefer Instant Messaging (IM), there is a solution for you. The chat application Pidgin, a mainstay of Linux communication suites, has a plugin named "OTR" (Off The Record) that can be used to encrypt your chat conversations. It operates in a similar way to PGP, in that you must first exchange public keys with your conversation partner. If you don't already use Pidgin, it is available for install in the Ubuntu repositories.

To install the OTR plugin, head to [the Cypherpunks site][8] and download the tarball for the OTR Library and Toolkit, as well as the one for "OTR Plugin for Pidgin." Then run the following:

	tar xzf libotr-*.tar.gz
	cd libotr-*
	./configure --prefix=/usr
	make
	sudo make install
	tar xzf pidgin-otr-*.tar.gz
	cd pidgin-otr-*
	./configure --prefix=/usr
	make
	sudo make install

This will install both the required libraries for OTR as well as the plugin specific to Pidgin.

To configure the plugin, open Pidgin and click Tools > Plugins. Check the box next to "Off The Record Messaging." Then, click the entry for "Off The Record Messaging" and choose Configure Plugin.

![][9]

Here you can choose a set of options based on how you want the plugin to behave. Also, you can choose to generate a key for a specific account. Once you begin a conversation with a friend who also has OTR enabled, you will see a notification display that you can begin a conversation with that person. Click "Not Private" and choose "Start Private Conversation" to enable encryption with the active conversation partner. And you're off! OTR is notoriously easy to set up and use.


## 2.4.4 - Further Reading

* [How To: Protect Your Privacy with Ghostery - Chip.eu][10]
* [Tor documentation for Linux][11]
* [Enigmail PGP Quick Start Guide][12]
* [How to Use OTR to Initiate a Secure Messaging Session in Pidgin - Tactical Technology Collective][13]

 [1]: https://www.eff.org/https-everywhere
 [2]: https://chrome.google.com/webstore/category/extensions
 [3]: ../img/2-4-1.png
 [4]: https://www.torproject.org
 [5]: https://www.torproject.org/projects/torbrowser.html.en
 [6]: https://www.torproject.org/download/download-easy.html.en#warning
 [7]: ../img/2-4-2.png
 [8]: http://www.cypherpunks.ca/otr/index.php#downloads
 [9]: ../img/2-4-3.png
 [10]: http://download.chip.eu/en/how_to/Protect-Your-Privacy-with-Ghostery_144165304.html
 [11]: https://www.torproject.org/docs/tor-doc-unix.html.en
 [12]: http://www.enigmail.net/documentation/quickstart.php
 [13]: https://securityinabox.org/en/pidgin_securechat