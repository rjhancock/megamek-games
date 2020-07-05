---
# You don't need to edit this file, it's empty on purpose.
# Edit theme's home layout instead if you wanna make some changes
# See: https://jekyllrb.com/docs/themes/#overriding-theme-defaults
layout: page
---
# MegaMek.games

This site is here to to give basic information on:

* Setting up your own private dedicated instance of MegaMek on dedicated hardware.
* Publicly available servers provided by one of the contributers.

## Public Servers

These servers are provided for the public to use. You MUST use the same version
posted to prevent errors from gameplay. List will be updated as servers are
added, removed, and updated.

|Server Name|Port Number|Version|
|:------|:-----:|--------:|
|ghost-bear.megamek.games|2346|0.46.1|
|diamond-shark.megamek.games|2346|0.47.7|

## Instructions

These are basic instructions for getting a dedicated server up and running
using a hosting provider. Does NOT require adjusting your own router or
firewall settings. These are at a basic to intermediate level.

I'll be using DigitalOcean as a basis for this. They have a large collection
of tutorials on setting up a server. They can be used with other providers
as well.

For this tutorial, we'll be using their cheapest droplet to setup an instance
on which is $5/mo USD. This will allow us to dedicate 768M of ram to MegaMek
which should be more than enough for most games and scenarios. If you need
more, upgrading to the $10/mo USD instance is a few clicks and the settings
can be adjusted to give MegaMek 1536M of ram instead. In our testing, that
was overkill.

To setup an account at [DigitalOcean](https://digitalocean.com), you can go
directly there or use this [referral link](https://m.do.co/c/769d663c4411).
Both will get you $100 USD credit for first 60 days, the referral will credit
me $25 USD AFTER you've spent $25.

If unfamilier with initial server setup, please check out
[Initial Server Setup with Ubuntu 20.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-20-04)
for setting up the initial user that we'll be using for MegaMek.

After you've done the inital server setup, follow along using the created
user.

### Server Requirements (recommended)

* Ubuntu 20.04 LTS
* OpenJDK 11 LTS

### System Updates and Java

Before we even begin to get MegaMek setup, we need to make sure the server is
up to date with all of it's patches and OpenJDK is installed. It's best to
log in via the created user vs root for security reasons.

First update all the software with the following command. It's all in one line
to chain 3 commands together.

```bash
# step 1
sudo apt update && sudo apt upgrade -y && sudo apt dist-upgrade -y
```

This will update the software list, upgrade already installed packages, and
upgrade the kernel if needed. A reboot is recommended after this but can wait
until after everything else has been setup.

For OpenJDK, it's another quick command:

```bash
# step 2
sudo apt install openjdk-11-jdk-headless
```

This will install OpenJDK 11 for headless systems. This basically means for
systems without a monitor attached.

If using the UFW firewall as mentioned in the above tutorial, it is important
that you open up the port for MegaMek to use otherwise you wont be able to
connect to the running server later.

```bash
# step 3
sudo ufw allow 2346
```

### Getting MegaMek

Since we don't have a GUI to click to download, we need to get a copy of the
archive directly from the [MegaMek](https://megamek.org/downloads.html)
website. Navigate there in your favorite browser and copy the Linux URL from
the downloads page. You can choose either the MekHQ package OR the MegaMek
one. Both will have the same files for a dedicated server. We recommend
using the MekHQ package to avoid seeing the `client/server hash mismatch`
error in the logs. So long as the VERSION (ie: 0.46.1) matches on both, there
should be no issues.

To get it onto the server, we're going to use `wget` to have it download the
package.

```bash
# step 4
wget <URL of package>
```

From a fresh login, this will download it into your home directory. Once
downloaded, we need to decompress the archive.

> **tip** If unable to determine the filename, you can get the current
> listing with:
>
```bash
ls -lha
```
>
> This will return a listing of files and folders in the current directory.

For MekHQ 0.46.1:

```bash
# step 5
tar -zxvf mekhq-0.46.1.tar.gz
```

To better understand what's going on, we are telling the `tar` command to

1. (z) decompress the compressed file,
2. (x) extract all files from the archive,
3. (v) do it verbosely so you can see the output, and
4. (f) which file to extract from.

### Setting up for auto start on reboot

Now that we have MegaMek download and ready to go, we need to get it setup to
start on system start. What we are about to do is

1. Create a symlink to a stable folder name so we don't have to touch system
files on each update.
2. Create the start up script to load MegaMek
3. Tell the system about the file.
4. Enable it for startup

First the symlink. This is so that when you update to a newer version, you
can stop the running instance, remove the old link, make a new one, and start
it up again without having to run any privileged commands or tell the system
something has changed. It'll `just work`.

Again, using MekHQ 0.46.1:

```bash
# step 6
ln -s mekhq-0.46.1 stable
```

This will create a link from the extracted location to a folder named `stable`
which we will use in our start up script.

For the startup script, we need to create what is known as a systemd service
file. This is how Ubutun will know about MegaMek and to start it at system
startup.

```bash
# step 7
sudo nano /etc/systemd/system/megamek.service
```

Enter your password when prompted. This will open up the about to be created
`megamek.service` file in a place that SystemD will see it using the `nano`
test editor. Copy and paste below into the window, `Control+X` to save and
`Enter` until back to terminal:

```bash
[Unit]
# step 8
# 1
Description=MegaMek Dedicated Server

# 2
After=network.target

[Service]
# 3
Type=simple

# 4
User=megamek

# 5
WorkingDirectory=/home/megamek/stable

# 6
ExecStart=/usr/bin/java -Xms768m -Xmx768m -jar MegaMek.jar -dedicated -port 2346

# 7
Restart=always

[Install]
# 8
WantedBy=multi-user.target
```

What this does is:

1. gives it a name,
2. tells it to start after the network has started,
3. that it's a simple service (it really is),
4. run it as the `megamek` user,
5. inside the `stable` directory we linked above.
6. It'll run the instance using 768M of RAM on port 2346.
7. To always restart when it crashed, and
8. that it's to be setup for a default system state.

Now that we've added a systemD file, we need to let it know it's there.

```bash
# step 9
sudo systemctl daemon-reload
```

This will tell SystemD to reload all of the service files and check for changes.

To confirm it caught the new file...

```bash
# step 10
sudo systemctl status megamek.service
```

and you should see something similar to

```bash
● megamek.service - MegaMek Dedicated Server
     Loaded: loaded (/etc/systemd/system/megamek.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
```

That's a good sign! Now we just need to enable it and start it with...

```bash
# step 11
sudo systemctl enable megamek.service
sudo systemctl start megamek.service
```

Now re-run the status command above and it should look something like...

```bash
● megamek.service - MegaMek Dedicated Server
     Loaded: loaded (/etc/systemd/system/megamek.service; enabled; vendor preset: enabled)
     Active: active (running) since Sun 2020-07-05 17:51:35 UTC; 5s ago
   Main PID: 36908 (java)
      Tasks: 16 (limit: 1137)
     Memory: 149.8M
     CGroup: /system.slice/megamek.service
             └─36908 /usr/bin/java -Xms768m -Xmx768m -jar MegaMek.jar -dedicated -port 2346

Jul 05 17:51:35 ghost-bear systemd[1]: Started MegaMek Dedicated Server.
Jul 05 17:51:35 ghost-bear java[36908]: Redirecting output to megameklog.txt
```

Congratulations! You have your own server setup now using the stock MegaMek
package! You can get to it using the IP Address in the control panel using the
default port of 2346.

Now, if you haven't already, you should restart the server to make sure all the
patches have applied and that the server will load up on startup.

```bash
# step 12
sudo shutdown -r now
```

This will trigger the shutdown command with the reboot option and to do it now
vs scheduling it for later. If using DigitalOcean, in about 1 minute, the
server should be back up and running. Log back in and re-run the status command
to confirm.

Happy Hunting MechWarrior!
