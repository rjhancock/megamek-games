---
layout: page
---
# Installation

These are basic instructions for getting a dedicated server up and running
using a hosting provider. Does NOT require adjusting your own router or
firewall settings. These are at a basic to intermediate level. **Familiarity
with SSH and the shell are imperative! Do *NOT* use a web based console for
this tutorial.**

I'll be using DigitalOcean as a basis for this. They have a large collection
of tutorials on setting up a server. They can be used with other providers
as well.

For this tutorial, we'll be using their cheapest droplet to setup an instance
on which is $5/mo USD as of this writing. This will allow us to dedicate 768M
of ram to MegaMek which should be more than enough for most games and
scenarios. If you need more, upgrading to the $10/mo USD instance is a few
clicks and the settings can be adjusted to give MegaMek 1536M of ram instead.
In our testing, that was overkill.

> If increasing the ram, adjust the start up script and replace the two
> instances of 768 with the amount of memory you want to allocate.
> Remember to leave at least 256M available for the system to function.

To setup an account at [DigitalOcean](https://digitalocean.com), you can go
directly there or use this [referral link](https://m.do.co/c/769d663c4411).
Both will get you $200 USD credit for first 60 days (as of 4/20/23), the
referral will credit me (TapEnvy.us, LLC) $25 USD AFTER you've spent $25.

> If using DigitalOcean, the initial survey can be a bit daunting. Most is for
> their internal CRM to help them figure out how people are using their
> services. We've suggested people fill in `self hosted game server` in for
> `What is the project?` (or however it's phrased).

If unfamiliar with initial server setup, please check out
[Initial Server Setup with Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-22-04)
for setting up the initial user that we'll be using for MegaMek.

After you've done the initial server setup, follow along using the created
user.

> We suggest calling the user `megamek` to keep it all the same as below.
> If you choose a different username, you'll need to make adjustments to
> the startup script replacing the user with what you chose as well as
> the `megamek` portion of the working directory.

## Server Requirements (recommended for most)

* Ubuntu 22.04 LTS
* OpenJDK 11 LTS
* 1 vCPU
* 1GB RAM
* 25G HDD

### Before you begin

Make sure you have a non-privileged user created with sudo privileges and you
can login as said user. You do NOT want to run MegaMek as the `root` user for
security reasons. If not user how to do this, follow this tutorial for Ubuntu 20.04 -
[Initial Server Setup with Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-22-04).

### System Updates and Docker

Before we even begin to get MegaMek setup, we need to make sure the server is
up to date with all of it's patches and Docker is installed. It's best to
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

For Docker, it's several steps that are being copied from the official documentation.

```bash
# step 2
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

These are a lot of steps, I know. In order, the first command removes any existing install on the current system. On a
fresh setup they should not exist but this is precautionary to prevent any conflicts. The next several lines ensures
the package cache is current, local certificates (used for ensuring secure communication), and curl (command line tool
for getting files and webpages) are installed. The next 3 lines sets ups a place for storying signing keys used to verify
packages (don't want unauthorized packages compromising our systems now), downloads Docker's public key, and makes it
readable by all.

The next line adds a package source, with the signing key, for our server architecture (Intel/Arm) to the package manager
so we can easily install and update docker packages. The last 2 lines updates the cache of packages and installs the various
Docker utilities.

Once all are installed, run `sudo systemctl enable docker` to ensure the Docker Engine boots up upon server startup.

### Permissions

Once Docker is installed, you need to make sure you can access it without being root. This is simple as

```bash
sudo usermod -aG docker <username>
```

then log out the back in for it to take effect. At this point, run `docker ps` to see that there are no containers running.
This is just a final test to ensure Docker is up and running.

## Getting MegaMek

This will be where we actually get the version of MegaMek from the Docker Hub. An account is not needed as we made
these images public. You can pick a version, `latest` (development), or `milestone` to get the version you want.

```bash
# step 4
docker pull tapenvyus/megamek:milestone
```

This will pull the latest milestone version. Replace the text after the `:` with either the version number, latest,
or milestone.

## Creating and Running the Image

Once the version is pulled, to run the image,

```bash
docker run -d --restart unless-stopped -p 2346:2346 --name megamek tapenvyus/megamek:milestone
```

This will take the newly downloaded image, start it up, set it to restart automatically upon system reboot, ensure
the ports are open to the public, and give it an easy to use name for updates.

## Updating Images

When using the `milestone` or `latest` tags for the images, this will make updating very easy. In
5 commands you can update, stop, and restart the container and be ready to go in seconds.

These will pull the latest tagged version of the image, stop the existing and running container,
remove the old container to be sure we have a clean slate, re-create and run the latest image,
then clean up old data from the system.

```bash
docker pull tapenvyus/megamek:milestone
docker stop megamek
docker container rm megamek
docker run -d --restart unless-stopped -p 2346:2346 --name megamek tapenvyus/megamek:milestone
docker system prune --volumes -f
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

Happy Hunting MekWarrior!
