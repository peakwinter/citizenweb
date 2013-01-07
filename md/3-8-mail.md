There are two components to the mail system we are going to build. The first component is **postfix**. Postfix is what we call a "Mail Transfer Agent" (MTA). An MTA is responsible for transporting email between different destinations. When you open your email application and send an email, that document gets transferred first to your email provider's MTA. The MTA then parses the message for a destination address, looks up its server's location on the Internet, then facilitates the transfer of the message to that server. An MTA also handles incoming email in the same way: your MTA gets contacted with a message from somebody else, then your MTA delivers the message to the Mail Delivery Agent.

The Mail Delivery Agent (MDA) is the second part of the mail system. Our MDA is called **dovecot**. The MDA handles the storage and organization of your mail once it is received. It may help to think of it as such: your MTA is your postman, going from house to house and delivering the mail; the MDA is your mailbox itself.

  
## 3.8.1 - First Steps: Install Postfix

We will begin with installing our MTA, Postfix.

`sudo apt-get install postfix` 

Postfix comes with a handy semi-graphical configuration tool, which we will use to start. Run the following:

`sudo dpkg-reconfigure postfix` 

Fill in the following details, which will match our configuration.

1.  **Mail server configuration type:** "Internet Site".
2.  **System mail name:** *mydomain.com*
3.  **Root and postmaster mail recipient:** *leave blank*
4.  **Other destinations to accept mail for:** Add *mydomain.com* to the beginning of this comma-separated list.
5.  **Force synchronous updates?:** No
6.  **Local networks:** Enter your IP subnet that we picked in the Networking section.
7.  **Use procmail?:** No
8.  **Mailbox size limit:** "0"
9.  **Local address extension character:** Leave as default.
10. **Internet protocols to use:** all

Now we need a place to put all that mail that's sure to arrive. In this example we will use the Maildir format, so run the following with your username in the place of $username:

	sudo postconf -e 'home_mailbox = Maildir/'
	export MAIL=/home/$username/Maildir
	sudo postfix restart

And with that, we have a simple mail transport system running! Take a moment to pat yourself on the back.

Now we will test what we have just set up. Ensure that postfix is running with `sudo postfix status`. If it's not, run `sudo postfix start`. Then open up `telnet` and open a session to your local SMTP port:

`telnet localhost 25` 

You'll receive the following output and a prompt if you have successfully connected:

	Trying 127.0.0.1...
	Connected to mail.mydomain.com.
	Escape character is '^]'.
	220 localhost.localdomain ESMTP Postfix (Ubuntu)

This prompt is a little different from the standard command, as it only understands SMTP commands. But not to worry - enter the following commands line-by-line to send yourself a test message:

	ehlo localhost
	mail from: root@localhost
	rcpt to: $username@localhost
	data
	Subject: My Postfix Test

	Test Message 123
	This is the body
	Goodbye
	.
	quit

Make sure to put your username in the right spot. Also, that line right above "quit" is indeed just a period. That tells postfix that our test message is complete and ready to be sent.

Now let's see if it worked. Run the `mail` command and you should see the subject line of your message. Press 1 and Enter to read it. Postfix is aliiiiiiiiiiiiiive!

In most cases, mail clients will send their outgoing mail on port 25. This is the port that mail servers communicate between each other with to transfer mail. However, many mail clients are set up by default to use port 587 to send mail over Secure (TLS) SMTP. If you'd like, you can also enable this port in Dovecot by editing `/etc/postfix/master.cf` and uncommenting the line that starts with "submission." To require logins over TLS for this port, uncomment the "-o smtpd\_recipient\_restrictions" section underneath "submission" and add reject\_sender\_login_mismatch to the list.
  
## 3.8.2 - Setting Up Mail Storage with Dovecot
This guide assumes that you want to run a mailserver for personal use only. It will therefore base your mail account off of your login account. If you are planning on running a server for others as well, it would be a good idea to set up virtual users, instead of setting up multiple users on your computer itself and potentially compromising it. To enable virtual users, follow the steps outlined in this guide, then add the steps found [here][1].

On Ubuntu Server, there are two flavours of dovecot: `dovecot-imapd` and `dovecot-pop3d` You can install either or both if you'd like. Though the one you choose will depend on which email protocol you would like to use for remote connections. POP is the older protocol, which operates by downloading all email on a remote server to local folders and organizing them by their type. POP then deletes the original messages from your server, leaving you with the copies and folder organization on your local computer only.

IMAP, on the other hand, is a more robust system and is recommended for those who prefer to have their mail synced to multiple locations (for example, on your laptop and on your mobile phone). IMAP syncs your mailbox's folders between the client and the server. Whenever you move an email between boxes, for example, IMAP will sync those changes to your email server in real time. You should be able to see how this is beneficial to people who use their email on multiple devices: no matter what you read or where you read it, the email's status and location can be synced across all of your devices.

So choose the version(s) of dovecot you would like to install:

`sudo apt-get install dovecot-imapd` 

Your main dovecot configuration file is stored at `/etc/dovecot/dovecot.conf`. In some versions of the software, including newer Ubuntu versions, this file "includes" other configuration files stored elsewhere, which can be found in `/etc/dovecot/conf.d`. We will be editing a variety of files to get our mail storage system set up.

Let's start with setting up our Maildir. This is the spot where mail is temporarily stored as dovecot routes it to its proper destination. Change the `mail_directory` line in `/etc/dovecot/conf.d/10-mail.conf` (or `/etc/dovecot/dovecot.conf` to match:

`mail_location = maildir:/home/%u/Maildir` 

Now we will set up the mail storage hierarchy and enable it for use with the following commands, again changing $username for the appropriate value:

	sudo maildirmake.dovecot /etc/skel/Maildir
	sudo maildirmake.dovecot /etc/skel/Maildir/.Drafts
	sudo maildirmake.dovecot /etc/skel/Maildir/.Sent
	sudo maildirmake.dovecot /etc/skel/Maildir/.Trash
	sudo maildirmake.dovecot /etc/skel/Maildir/.Templates
	sudo cp -r /etc/skel/Maildir /home/$username
	sudo chown -R $username /home/$username/Maildir
	sudo chmod -R 700 /home/$username/Maildir

Once this is complete, we are ready to start and test Dovecot. Start it with `sudo service dovecot start`. Then open up a telnet with `telnet localhost imap`. If you see something like this:

	Trying localhost...
	Connected to localhost.
	Escape character is '^]'.
	+OK dovecot ready.

... then we are ready to go to the next step!

  
## 3.8.3 - Securing Your Mail System

The importance of running a safe and secure mail system cannot be overstated. For one, you certainly don't want your system to be used to forward spam off across the internet. If your system allows spam to be relayed then it can even find its way onto a blacklist, meaning some providers can refuse mail from your email accounts! And we certainly don't want that. So it is very important that we secure our mail system. To do this, we will enact the following policies:

First comes our Postfix SASL configuration. This is the mechanism that Postfix uses to securely authenticate users and servers. You will need to install the `libsasl2-2, sasl2-bin` and `libsasl2-modules` packages. Now, open up `/etc/default/saslauthd` and change the following lines:

*   `START=yes` should be uncommented
*   Add or change the following lines:
	*   `PWDIR="/var/spool/postfix/var/run/saslauthd"`
	*   `PARAMS="-m ${PWDIR}"`
	*   `PIDFILE="${PWDIR}/saslauthd.pid"`
	*   `OPTIONS="-c -m /var/spool/postfix/var/run/saslauthd"`

Run the following commands to enable SASL in your postfix configuration:

	sudo postconf -e 'smtpd_sasl_local_domain ='
	sudo postconf -e 'smtpd_sasl_auth_enable = yes'
	sudo postconf -e 'smtpd_sasl_security_options = noanonymous'
	sudo postconf -e 'broken_sasl_auth_clients = yes'

Next, we will set the access restrictions for sending mail on our server:

	sudo postconf -e 'smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination' 
	sudo postconf -e 'inet_interfaces = all'

This line tells Postfix that our server will automatically accept mail from authenticated users (like your mail client), OR on any device connected to our own network, because we know they can be trusted. Furthermore, our server will outright reject any mail sent to it that is not addressed to our domain OR that is sent from our domain.

Finally, we will start up our SASL authenticator by running:

`dpkg-statoverride --force --update --add root sasl 755 /var/spool/postfix/var/run/saslauthd` 
`sudo service saslauthd start` 

At this point, were you to run `telnet localhost 25` and pass `ehlo localhost`, you should receive `250-STARTTLS` as one of the responses. That means secure logins are now available for our outgoing mail server.
  
Next we will set up our mail storage system (Dovecot) to allow clients to connect to it in a secure way. This will enable us to use encrypted connections when we are away from home, so no snoops will be able to pick out our passwords when we check our mail.

Edit the `/etc/dovecot/conf.d/10-ssl.conf` file, and change the following lines:

	ssl = required
	ssl_cert_file = /etc/ssl/certs/ssl-cert-snakeoil.pem
	ssl_key_file = /etc/ssl/private/ssl-cert-snakeoil.key

If you are planning on running an email system for multiple people, it may be a good idea to use a purchased SSL certificate instead of a self-signed certificate. If not, all of your clients will get "Untrusted" messages before using their email, which may be unsettling. If you purchase these certificates, change the above pointers to match the location of the appropriate cerificate and keyfile on your system. For more information on SSL certificates, check out the [LDP page on the subject][2].

With this, Dovecot will require its clients to authenticate themselves securely. You can now test your system by opening up your mail client of choice and adding your mail account. Enter the username and password of your user account on the server that you want to use. Set mail.*mydomain.com* as both your incoming (IMAP) and outgoing (SMTP) mail server. Make sure IMAP is using port 143, and SMTP is using port 25 or 587, whichever you chose in the Postfix configuration.

  
## 3.8.4 - Further Reading

*   [Postfix Documentation][3]
*   [Dovecot Wiki][4]

 [1]: http://www.debuntu.org/how-to-virtual-emails-accounts-with-postfix-and-dovecot
 [2]: http://www.tldp.org/HOWTO/SSL-Certificates-HOWTO/
 [3]: http://www.postfix.org/documentation.html
 [4]: http://wiki2.dovecot.org/