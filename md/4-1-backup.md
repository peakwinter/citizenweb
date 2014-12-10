## 4.1.1 – Backup

Backing up with Linux is easy. The quickest and simplest way to do it is to simply move your data into a TAR archive. This can be accomplished with the following command:

`tar cvzf archivename.tar.gz FILES`

The "cvzf" is actually a list of options, which means: (c)reate an archive, print out the list of files we want to compress (v)erbosely, (z)ip up the archive with gzip, and specify our own (f)ilename.

If we want to back up a directory of files, we can easily do that with:

`tar cvzf archivename.tar.gz --directory=/path/to/folder/ .`

This will copy the chosen folder's contents to a new archive in the current folder. You can easily use this to backup individual folders into archives, then move them to a different drive or an offsite location.

For paranoid cases, you can choose to backup your entire system with:

`tar cvzf bak-system.tar.gz --directory=/ .`


Restoring from a backup is simple:

`tar xvzf archivename.tar.gz -C /path/to/extract/dir`

Make sure that you set the proper location for whatever data you want to restore.


## 4.1.2 – Encrypt Your Backups

It is advisable that you encrypt your backups before they leave your computer. This is especially true if you wish to use a public backup storage service like tarbackup.

You will need to decide upon a method for encryption. You can opt to enter a password each time you wish to backup or restore an archive, or you can choose to keep a password stored in a file on your harddrive. This isn't the advisable option, as it is less secure: anyone who can get their hands on your computer can potentially find and decrypt your backups if they are stored elsewhere. However, it can be helpful for automating your backups via a bash script (more on that in a future guide).

To encrypt a backup, first create the TAR archive as described above. Then use openssl to create your backup. If you are using a password:

`openssl enc -aes-256-cbc -salt -in archivename.tar.gz -out archivename.tar.gz.enc -pass pass:PASSWORD`

If you are using a password stored in a keyfile:

	echo PASSWORD > enc.key
	openssl enc -aes-256-cbc -salt -in archivename.tar.gz -out archivename.tar.gz.enc -pass file:enc.key

Again, if you are to keep this file around, take precautions to secure it!

After the command completes, your archive file will be encrypted. Remove the .tar.gz file and store the .tar.gz.enc file as you need to.


To decrypt the archive, run:

`openssl enc -d -aes-256-cbc -in archivename.tar.gz.enc -out archivename.tar.gz -pass pass:PASSWORD`

If you are decrypting with a file rather than with an entered password, substitute "file:FILENAME" for "pass:PASSWORD".


## 4.1.3 – Options for Storing Backups

Backing up to TAR files is convenient because you have many different storage options. After the TAR archive is created and/or encrypted, you can simply move it to whatever storage media you want. This can be an external hard drive, a DVD, another server or NAS drive, etc.

There are also a few different options for storing tar-based backups online these days. Tarbackup.com and Tarsnap.com are two such services. You can pay a low fee to store your encrypted backups on their high-capacity servers, ready for download at a moment's notice.

