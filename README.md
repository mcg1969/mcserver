A $5/month cloud Minecraft server
------

References:

- https://minecraft.net/download
- http://otoris.com/host-your-own-minecraft-and-mumble-server-for-5month/
- https://www.digitalocean.com/community/tutorials/how-to-add-swap-on-centos-7
- http://minecraft.gamepedia.com/Tutorials/Server_startup_script
- https://www.digitalocean.com/community/tutorials/how-to-set-up-a-minecraft-server-on-linux
- http://minecraft.gamepedia.com/Commands
- https://minecraft.net (of course!)

0.  You'll need at least one [paid Minecraft account](https://minecraft.net).

1.  You'll also need a DigitalOcean account. Use 
    [my referral code](https://www.digitalocean.com/?refcode=71af5e43aa08)
    to get $10 credit right off the bat, which means you are not spending
    money to try this out. If you stick with it long enough to rack up $25
    in bills, I will get $25 in credit myself. These guys are well regarded,
    and I use them for business as well as Minecraft.

2. Log into DigitalOcean, click "Create", and configure your server:

	- $5MB/mo (512MB, 1CPU, 20GB SSD)
	- Give it a name. (e.g., "Minecraft")
	- Select your Unix flavor. I chose CentOS 7.0 x64, so if you want 
	    these instructions to align perfectly, pick that.
	- If you have an SSH key, provide it. If not, or you don't know what
		that means, no problem; it will email a root password.

	Once the server is created, it will give you the IP address (e.g.,
	173.172.171.170); and, if necessary, it will email your root password.

3.	Log into the machine, change your root password, instal Java and Screens,
	and add a swap partition. Without a swap, the Minecraft server will die
	when it runs out of memory. (And it will.)

		ssh root@173.172.171.170
		passwd
			<change your root password>
		yum install java-sdk screens
		fallocate -l 4G /swapfile		
		chmod 600 /swapfile
		mkswap /swapfile
		swapon /swapfile

	Add this line to the file "/etc/fstab" with your favorite editor:

		/swapfile swap swap sw 0 0

	A note about Java. At first, I went through some additional hoops to
	download and install the absolute latest greatest verison of Java from
	Oracle. This worked great for vanilla Minecraft, but some of the mods I
	installed are incompatible with Java 8. The instructions above install
	Java 7 instead. Since you're not using this server for *anything* else
	(right?) I don't think you need worry too much about using an older Java.

4.	Create a user account and move over to that.

		useradd minecraft
		passwd minecraft
			<change minecraft's password>
		su minecraft
		mkdir backups
		cd ~

	You'll see what the backups directory is for later.

5.	Download the latest minecraft server software, and rename it.

		https://s3.amazonaws.com/Minecraft.Download/versions/1.8.1/minecraft_server.1.8.1.jar
		mv minecraft_server.1.8.1.jar minecraft_server.jar

	With your favorite text editor, create the file "ops.txt" with a single
	line containing your Minecraft username. The server will read this file
	and designate that person as the first and only "op", or administrator.
	You will be able to add more ops later.

6. 	Time to run the server! The first time you do so, it will create some
	files---and stop, because you need to accept the EULA to go further.

		java -Xmx480M -Xms480M -XX:+UseConcMarkSweepGC \
 			-XX:+CMSIncrementalPacing -XX:ParallelGCThreads=1 \
 			-XX:+AggressiveOpts -jar minecraft_server.jar nogui

 	Edit the file "eula.txt" with your favorite text editor, and change the
 	last line to "eula=TRUE". Then re-run the server.

		java -Xmx480M -Xms480M -XX:+UseConcMarkSweepGC \
 			-XX:+CMSIncrementalPacing -XX:ParallelGCThreads=1 \
 			-XX:+AggressiveOpts -jar minecraft_server.jar nogui

 	Assuming everything is working properly, you'll see the logs show that it
 	is building the world. Once the logs settle down and say it's done
 	building, move on to the next step.

7.	Now you need to try connecting to the server! 

		- Fire up your local Minecraft client.
		- Click "Multiplayer"
		- Click "Add Server". Enter a name for the server, and the IP
			address of the server (e.g., 173.172.171.170). 
		- Click "Done."
		- Now select the server you just added, and click "Join Server."

	If everything is working right, you'll be transported to a brand new
	Minecraft world! 

8.	Shut down the server with "ctrl-c". Now let's open up "server.properties"
	with a text editor and look at some particular lines:

		- white-list=false: you should change this to "white-list=true" to
			prevent univited persons from joining.
		- gamemode=1: change this to "gamemode=0" for creative.
		- level-type=DEFAULT: this is a standard terrain-filled world. If
			you *really* like creative, you may want to change this to 
			level-type=FLAT to get a flat world with no terrain.
		- max-players=20: this is more than your little 512MB server can
			handle. You can reduce this to, say, max-players=5 if you want,
			but if your whitelist is short it won't matter.
		- motd=A Minecraft Server: this is just a Message of The Day. Change
			it to something fun!

	[Here is a reference guide](http://minecraft.gamepedia.com/Server.properties) for this file.
	
	Don't restart the server yet.

9.	Drop back down to root (ctrl-D, log in again, whatever) and download this 
  	init script, which will start Minecraft automatically on boot:

		cd ~
		wget -O minecraft http://j.mp/1w3VTUz

	The abbreviated link is to the script offered 
	[on this page](http://minecraft.gamepedia.com/Tutorials/Server_startup_script).
	Edit this script with your text editor and change these variables. If you
	made any modifications to the above instructions, you may need to do more:

		MCPATH='/home/minecraft'		
		BACKUPPATH='/home/minecraft/backups'
		MINHEAP='480'
		MAXHEAP='480'

	Now move the script into its proper position:

		cp minecraft /etc/init.d/
		chmod +x /etc/init.d/minecraft
		chkconfig --add minecraft

	Fire it up and let's see if it works!

		service minecraft start

	Don't be alarmed if it takes almost 10 seconds before it says 
	"minecraft_server.jar is now running."

10.	Try connecting to the server again with your Minecraft client! Hopefully
	all should be well.

11. If that works, there's one more thing you may want to do. The startup
	script has a backup facility that makes periodic backups of your world.
	which might come in handy if someone really screws up. Type

		crontab -e

	and add the following line to have hourly backups saved:

		15 * * * * /etc/init.d/minecraft backup

	TODO: set up a logrotate script so that you don't fill your entire disk
	with these backups.

12. At this point, you should be able to log out of the server and work with
	the Minecraft client as an operator. Some useful, and hopefully
	self-explanatory, commands:

		/op <username>
		/whitelist <username>
		/banlist <username>

	More commands and explanations can be found 
	[on this page](http://minecraft.gamepedia.com/Commands).

13. If you ever need to log into the server to see what's going on, log in
    *as the minecraft user*, not root, unless you know you need to. Then type

    	screen -r

    to switch over to the running minecraft server (assuming it's still 
    running). There you can type commands to help you manage the server. 
    
