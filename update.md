---
layout: page
---
# Updating

Updating the version on your server is considerably easier than the initial
installation. Few steps, less things to break. In fact, the process should
feel similar to how it was for the initial installation.

## System Updates and Java

Before we update MegaMek, lets make sure our server has the latest patches
installed.

First update all the software with the following command. It's all in one line
to chain 3 commands together.

```bash
# step 1
sudo apt update && sudo apt full-upgrade -y
```

## Getting MegaMek

Same as with the installation, we need to get a copy of the latest version onto
the server. So navigate to [MegaMek](https://megamek.org/downloads.html)
and copy the URL for the Linux version (you're choice on either the MegaMek
or MekHQ package).

```bash
# step 2
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

For MekHQ 0.49.17:

```bash
# step 3
tar -zxvf mekhq-0.49.17.tar.gz
```

To better understand what's going on, we are telling the `tar` command to

1. (z) decompress the compressed file,
2. (x) extract all files from the archive,
3. (v) do it verbosely so you can see the output, and
4. (f) which file to extract from.

## Stop Previous Version

Now that we have MegaMek download and ready to go, we need to get swap it out
with the currently running version. First we need to stop it so we don't have
any data corruption.

```bash
# step 4
sudo systemctl stop megamek.service
```

## Remove and Relink current version

Now we need to get rid of the old link (assuming you used `stable` for
installation) then link the new version up with

```bash
# step 5
rm stable
ln -s mekhq-0.49.17 stable
```

## Start the new version

Now that it's linked, time to start up the new version and verify it's
running with:

```bash
# step 6
sudo systemctl start megamek.service
sudo systemctl status megamek.service
```

And you should see something akin to below.

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

Congratulations! You have updated your server to the latest version of
MegaMek!

## Cleaning Old Files

To remove the old files, execute the following similar commands. Be mindful to replace the file name
with whatever is the previous version(s).

```bash
ls -lha
rm -rf mekhq-0.49.15*
```

## Restart for any system updates

It's probably a good idea to restart the server to ensure any security updates
to the system get fully applied.

```bash
# step 7
sudo shutdown -r now
```

This will trigger the shutdown command with the reboot option and to do it now
vs scheduling it for later. If using DigitalOcean, in about 1 minute, the
server should be back up and running. Log back in and re-run the status command
to confirm.

Happy Hunting MekWarrior!
