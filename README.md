## A $5/month cloud Minecraft server

### Introduction

If your child is a full-bore Minecrafter, he or she is probably going to want to
join a server at some point, to play with other people. Finding a public server
that is also safe is no easy task. One alternative is to create your own
"invite-only" server, so that you and/or your child can control who can play
along. 

Minecraft has its own service, (Minecraft Realms)[https://minecraft.net/realms],
that is quite easy to use. But it costs $13/month to run a server, or $10.40/month
if you let them bill your credit card automatically. If you don't mind investing
a bit of effort, and your are roughly comfortable with a Unix command line, you
can get your own server up and running for only $5/month! If that interests you,
read on.

### Prerequisites

These instructions make the following assumptions. If these assumptions are
not satisfied, then you might consider Minecraft Realms.

- You have at least one [paid Minecraft account](https://minecraft.net) to serve
  as the "operator" of your server. When this player connects to the server using
  a Minecraft client, he/she will have access to operator commands that can,
  for instance, add or remove players from the whitelist, switch between creative
  and survival mode, and so forth.
- You are roughly familiar with a Unix command line. You don't have
  to understand what every typed command below actually means.
- You need to know how to use a Unix text editor like `vi` or `nano`. Honestly,
  `nano` is pretty straightforward.
- You need an SSH client on your host computer. On the Mac, this means firing
  up the Terminal application and typing `ssh `*<hostname*>. There are SSH apps
  available for the PC, iPad, and (probably) Android as well.

### References

- https://minecraft.net/download
- http://otoris.com/host-your-own-minecraft-and-mumble-server-for-5month/
- https://www.digitalocean.com/community/tutorials/how-to-add-swap-on-centos-7
- http://minecraft.gamepedia.com/Tutorials/Server_startup_script
- https://www.digitalocean.com/community/tutorials/how-to-set-up-a-minecraft-server-on-linux
- http://minecraft.gamepedia.com/Commands
- https://minecraft.net (of course!)

### Here we go!

1.  Create a DigitalOcean account. DigitalOcean is a well regarded cloud server
    provider that I use for business as well as for Minecraft. Use
    [my referral code](https://www.digitalocean.com/?refcode=71af5e43aa08)
    to get $10 credit right off the bat, which means you are not spending
    money to try this out. If you stick with it long enough to rack up $25
    in bills, I will get $25 in credit myself. And of course, you can 
    start referring your friends, too, once you sign up.

2. Log into DigitalOcean, click "Create", and configure your server:

	- *Droplet hostname*: your choice; e.g. "Minecraft".
	- *Select size*: $5MB/mo (512MB, 1CPU, 20GB SSD)
	- *Select region*: stick with the default unless you happen to live
	  closer to one of the other regions.
    - *Available settings*: no changes needed.
    - *Select image*: from the "Linux distributions" tab, choose CentOS 7.0 x64.
      Feel free to choose another distribution that you are more familiar 
      with, but these instructions were built using CentOS.
	- *Add SSH Keys*: if you have an key, provide it. If not, or you don't know what
		that means, no problem---DigitalOcean will email a root password.

	Once the server is created, it will give you the IP address (e.g.,
	173.172.171.170); and, if necessary, it will email your root password.

3.	Log into the machine as `root`. For instance, on the Mac, open Terminal
    and type this command (changing the address, of course):

		ssh root@173.172.171.170

	Once you are connected to the server, you need to change the root password,
	install Java and Screens, and add a swap file. Without a swap file, the
	Minecraft server will die whenever it runs out of memory. (And it will.)

		passwd
			<change your root password>
		yum install java-sdk screens
		fallocate -l 4G /swapfile		
		chmod 600 /swapfile
		mkswap /swapfile
		swapon /swapfile

	Edit the file `/etc/fstab` and add this line:
	
		/swapfile swap swap sw 0 0

	A note about Java. At first, I went through some additional hoops to
	download and install the absolute latest greatest verison of Java from
	Oracle. This worked great for vanilla Minecraft, but some of the mods I
	installed are incompatible with Java 8. The instructions above install
	Java 7 instead. Since you're not using this server for *anything* else
	(right?) I don't think you need worry too much about using an older Java.

4.	It's not good form to run any sort of server with root-level access. So
    create a user account that will hold the minecraft program, log in as that,
    and do some initial preparations.

		useradd minecraft
		passwd minecraft
			<change minecraft's password>
		su minecraft
		cd ~
		mkdir backups

	You'll see what the backups directory is for later.

5.	Download the latest minecraft server software, and rename it.

		wget https://s3.amazonaws.com/Minecraft.Download/versions/1.8.1/minecraft_server.1.8.1.jar
		mv minecraft_server.1.8.1.jar minecraft_server.jar
		
6.  Create a new file called `ops.txt` with the Unix text editor.
	In this file, place the Minecrat usernames of anyone you wish to give
	"operator" access to this server. You need at least one operator, but
	one is enough for now. Any operator can add more operators, if desired,
	right from the Minecraft client itself during gameplay.

7. 	Time to run the server! Here's the magic command:

		java -Xmx480M -Xms480M -XX:+UseConcMarkSweepGC \
 			-XX:+CMSIncrementalPacing -XX:ParallelGCThreads=1 \
 			-XX:+AggressiveOpts -jar minecraft_server.jar nogui

 	The first time you run this, it will create some files---and then stop!
 	It needs you to accept the EULA before it will go further. Edit the file
 	`eula.txt` that it just created, and change the
 	last line to "eula=TRUE". Then re-run the server.

		java -Xmx480M -Xms480M -XX:+UseConcMarkSweepGC \
 			-XX:+CMSIncrementalPacing -XX:ParallelGCThreads=1 \
 			-XX:+AggressiveOpts -jar minecraft_server.jar nogui

 	Assuming everything is working properly, you'll see the logs show that it
 	is building the world. Once the logs settle down and say it's done
 	building, move on to the next step.

8.	Now you need to try connecting to the server! 

	- Fire up your local Minecraft client.
	- Click "Multiplayer"
	- Click "Add Server". Enter a name for the server, and the IP
		address of the server (e.g., 173.172.171.170). 
	- Click "Done."
	- Now select the server you just added, and click "Join Server."

	If everything is working right, you'll be transported to a brand new
	Minecraft world! Once you have confirmed everything is working, 
	disconnect your Minecraft client from the server, and then halt
	the server itself by typing Ctrl-C.

9.	With the server shut down, edit the file `server.properties` and consider
    changing a few of the parameters therein:

	- `white-list=false`: you should change this to `white-list=true` to
		prevent univited persons from joining. Any operator will be able
		to add people to the whitelist with the `/whitelist` command.
	- `gamemode=1`: change this to "gamemode=0" for creative.
	- `level-type=DEFAULT`: this is a standard terrain-filled world. If
		you *really* like creative, you may want to change this to 
		`level-type=FLAT` to get a flat world with no terrain.
	- `max-players=20`: this is more than your little 512MB server can
		handle. You can reduce this to, say, max-players=5 if you want,
		but if your whitelist is short it won't matter.
	- `motd=A Minecraft Server`: this is just a Message of The Day. Change
		it to something fun!

	[Here is a reference guide](http://minecraft.gamepedia.com/Server.properties) for this file.
	
	If you changed the `level-type` to `FLAT`, you will need to delete the
	world that Minecraft has already created so that it will create a new one.
	To do so, remove the entire `world` directory:
	
		rm -rf world
	
	In any case, don't restart the server yet!

10.	Drop back down to root (ctrl-D, log in again, whatever) and download this 
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

11.	Try connecting to the server again with your Minecraft client! Hopefully
	all should be well.

12. If that works, there's one more thing you may want to do. The startup
	script has a backup facility that makes periodic backups of your world.
	which might come in handy if someone really screws up. Type

		crontab -e

	and add the following line to have hourly backups saved:

		15 * * * * /etc/init.d/minecraft backup

	TODO: set up a logrotate script so that you don't fill your entire disk
	with these backups.

13. At this point, you should be able to log out of the server and work with
	the Minecraft client as an operator. Some useful, and hopefully
	self-explanatory, commands:

		/op <username>
		/whitelist <username>
		/banlist <username>

	More commands and explanations can be found 
	[on this page](http://minecraft.gamepedia.com/Commands).

14. If you ever need to log into the server to see what's going on, log in
    *as the minecraft user*, not root, unless you know you need to. Then type

    	screen -r

    to switch over to the running minecraft server (assuming it's still 
    running). There you can type commands to help you manage the server. 
    
