(Note that Virtual Private Server (VPS) users can skip this article entirely. Embedded miniserver users like those with the Raspberry Pi can skip down to section 3.2.3.)

## 3.2.1 - List Your Options

What do you want your server to do? What will it be handling for you on a daily basis? These are important questions to answer before shopping for your server hardware.

*   **Will you be running hardware-intensive services?** Servers that run virtual machines or media servers traditionally have much higher hardware requirements than simple email/web servers. The more VMs you want to run, the better CPU you will need; similarly, the more media services you wish to host, the more memory you will need. For any server that is to run a media service, it is recommended to have at least 8GB of memory.
*   **Will this be a "headless" server?** Will this computer need to be used directly, or can you simply put it in a corner and manage it from your laptop via an SSH connection? If you need to access it more than once, it would be a good idea to buy a monitor as well. Keep in mind that you will need to use a monitor during the server setup, but if you have another desktop, you can borrow that monitor just for the installation.
*   **Will this be a firewall or network controller?** Do you plan on using this computer to serve as a firewall instead of using your existing router configuration? Will this computer be serving DHCP clients, or will you leave that to another router connected to the network? If you've answered yes to any of those questions, it would be a good idea to get a server motherboard equipped with two ethernet ports (NICs). One will be "front-facing," that is, connected to your cable/DSL modem; the other will connect to a hub or wireless access point for your internal network.

![][1]
**Network-attached only (no firewall)**

![][2]
**Network-routed and firewalled**

  
## 3.2.2 - Buy Hardware

Now we get to the fun part - doing some shopping! Load up your favourite computer parts vendor and let's get started.

Popular parts vendors in the US and Canada are [Newegg][3] and [TigerDirect][4]. Newegg usually has the better prices and availability, but whichever one you pick is up to you. It's usually best to make lists on a few different sites to see which one actually has the cheapest price for that specific application. In the UK/Europe, check out [Misco][5].

### CPU

The most popular server CPUs these days are Intel, hands down. The Sandy Bridge and Ivy Bridge-class processors are really without comparison when it comes to performance and dependability. You can find decent ones for between $250 and $350 that will provide more than enough power for what we are looking to accomplish with our server.

It is also important to remember your CPU's cooling requirements. Most new Intel CPUs come with cheap but decent cooling fans; though if you are looking to improve your server's noise production, it may be a good idea to buy a nicer fan as well. Just make sure the fan is compatible with your chosen CPU's socket type.

### Memory

Some individuals and companies may consider this heresy, but you really don't need to buy the most expensive RAM out there in order to have a dependable and quick system. If you are spending more than $150 on RAM, you are very likely spending too much. Decent server memory is not too much more than normal memory.

### Motherboard

The motherboard is where the entire system comes together. Choosing one depends on the services you wish to offer with this server.

99% of the time, you will want to choose a server motherboard. These boards support server-class CPUs like the Intel Xeon series. Furthermore, most of them come with two Ethernet ports (NICs). This is indispensable for servers that act as routers for internal networks, or servers that will host email/web services. A common setup is to plug the cable/DSL modem into the first NIC as a "front-facing" interface, then to route the internet connection through to the second NIC, which is connected directly to your network hub or wireless access point.

It is possible to get by with a standard motherboard and CPU if you only want to do media sharing on your internal network, but if you are even *considering* doing more than that, it's best to go for the server motherboard and CPU.

Regardless of the class of motherboard you go with, the most important match you will make is between motherboard and CPU. You MUST remember to pair them by their socket type. For example, socket LGA1155 CPUs might not fit every socket LGA1366 or LGA2011 motherboard, etc.

Also keep RAM (memory) in mind. Motherboards have different types, sockets and speeds for RAM, as well as limits to how much memory they can handle, so make sure you can find one that works with your memory requirements. Your motherboard's manual, usually available in PDF from the manufacturer's website, will have all of this information.

### Case

Cases might not seem like an important consideration, but there are two critical elements to be aware of when choosing one to meet your needs.

*   **Size**: There are many size designations for motherboards: ATX, Mini ATX, Micro ATX, etc etc. Make sure the case is the correct size for the motherboard you are looking to purchase.
*   **Power Supply**: Most cases these days come with their own power supplies, but they are not all created equal. If you are planning on purchasing a computer with an Intel server CPU, you will definitely need a power supply with 24-pins (or "20+4"). The extra 4 pins are required to meet the motherboard and CPUs extra requirements. Keep in mind that, if you have your heart set on a particular case that comes with an incompatible power supply, you can always remove the old one and install one separately purchased.

### Hard Drive(s)

Again, the type of hard drives you will need will vary depending on what you want to accomplish with them. For simple web/email servers, you will not need much space at all. For those looking to do any sort of file hosting, space will likely be very important. You can pick a certain number of drives that can be matched via a RAID array, which can either:

*   ...stripe them together (i.e. effectively making 4x 2TB drives into one giant 8TB drive);
*   ... OR mirror them, for an instant backup in case one drive in the formation fails. (Making 4x 2TB drives into two sets of 4TB drives, with one acting as a live backup in case the other set goes down).

Drives should also be purchased according to their type and the compatibility with the motherboard. Nearly every motherboard these days supports SATA, the new standard for drive connectivity; however there are multiple types of SATA: 1.5GB/s, 3.0GB/s and the newer 6.0GB/s. If your motherboard supports 6.0GB/s, and you plan on hosting/moving very large files with your server, it would be worth it to consider 6.0GB/s SATA drive(s).

Finally, brand name and warranty does still mean something, especially since hard drives are such important components in your server. After all, all your personal data rests on them; replacing the drive is much easier than replacing the data. Go with a brand that is known to be good. Western Digital Black series drives have a good record of dependability; many of them also come with record 5-year warranties, making them an excellent option.

### Other Stuff

Other things you will need to consider:

*   Keyboard/Mouse
*   CD/DVD drive
*   Power strips and plugs
*   Monitor: *Remember that this is optional if you are going to run a headless server, but you will at least need access to one temporarily when you install your distribution.*

  
## 3.2.3 - ISP and Domain Name Options

If you are not planning to use your server to host any external (Internet) service, OR you have opted to use a Virtual Private Server (VPS), you can skip this section.

Dealing with your internet service provider, no matter how much you might dread it, will be a necessary component of this setup if you plan on hosting a website or your email on this server. Your server needs the ability to be linked to a domain name, which means it also needs a static IP. This is something your internet service provider can give you. If you want to host multiple servers and services on VMs (say a fileserver VM and an email/web host VM) it would be a good idea to also get a static subnet.

Usually when you connect to the Internet, your service provider gives you a dynamically-set IP address to use. However when your web/email services go live, the Internet will need a steady and static address with which to look you up. This is why at least one static IP address is required. A static subnet is an extension of the above idea, but it obtains multiple static IP addresses that belong to a specific "subnet," or a subset of IP numbers. For example, if you were to obtain what is called a "/29 subnet," that gives you six static IP addresses to use.

Some residential internet providers no longer allow clients to request static IP addresses or subnets; if this is the case, you may need to consider springing for a Business class plan, as these always have the ability to obtain static IP addresses. In many cases they are not more than $10 or $15 more than your original residential plan would be.

Once you've dealt with your ISP, you must purchase a domain name. This will likely be much easier (and probably cheaper) than the prior step. There are many decent domain name registrars out there, but I have to recommend NameCheap.com. As far as price, ease-of-use and customer service are concerned, they are consistently cited as one of the very best. For a domain, you can choose anything with any ending; though something simple is advisable if you are to be using an email address as well. Nothing like typing a 15-character domain when you want to send someone an email.

When buying a domain name, keep in mind that the domain you purchase will be subject to the laws and regulations of the country that you register it in. Wikipedia ran into trouble in the United States when its ".org" address was rescinded by US authorities because it published material that the government wasn't too happy to see. The common ".com," ".net" and ".org" are overseen by the US Government. Other countries, such as Iceland, have a more favourable policy towards the publishing of controversial or leaked information that would be in the public interest. It's advisable for those who look to post potentially sensitive information to consider an Icelandic domain. For more information regarding Iceland's national freedom of expression policy known as the "Icelandic Modern Media Initiative," [visit its website][6]. -- 

With the static IP in hand and the domain name registered, it's time to get them linked together. On your domain registrar's account page, there will be a place marked something like "Host Records" or "Domain Settings." (On NameCheap it is found at My Account > Manage Domains > click the domain name > All Host Records.) You will be presented with a list of fields, usually arranged into at least four columns: Host Name, IP Address, Record Type, and TTL.

*   In the Host Name field "@", put your static IP address in the correct field, and set the record type as "A". This will allow people to reach your website by visiting http://mydomain.com.
*   If there is a field for "www" hostname, or if you can create one yourself, do the same for an A record with your same IP address. This will allow people to reach the same site when going to http://*www*.mydomain.com as well.
*   Finally, we will set our domain up for mail. There should be a section for "MX Records" or "Mail Settings." The hostname should be "mail", the IP address matching your static IP, and the "MX Pref" should be "10". When an email server wants to forward you an email, they will check this record and see your IP, allowing them to actually make the connection between servers and deliver the message.

With the correct settings enabled, and the Internet ready to welcome our server, let's start assembling the computer itself.

 [1]: ../img/3-2-1.jpg
 [2]: ../img/3-2-2.jpg
 [3]: http://www.newegg.com
 [4]: http://www.tigerdirect.com
 [5]: http://www.misco.co.uk
 [6]: https://immi.is/Icelandic_Modern_Media_Initiative