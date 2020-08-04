---
layout: page
---

# Custom Data

So you want custom data on your server but everytime you update you have to
copy things back in place. Gets annoying especially when you may not be
familier with the OS you're working on. Well, let's make that easy for you
shall we?

## Shared Folder

First we need to make a common shared folder for all of our units, maps, etc:

```bash
cd ~
mkdir -p shared/boards shared/images shared/mechfiles shared/scenarios shared/forcegenerator shared/mapgen shared/names shared/rat shared/sounds
```

First we make sure we're inside our home directory (assuming this is also where
you have megamek installed at). Then we create each of the subfolders inside of
a central shared folder. Place any custom for each inside of these respective
folders.

## Linking Shared Folder to Stable

There are 2 ways we can do this.

1) We can manually link each folder every time we update MegaMek.
2) We can create a script to do the hard work for us.

Let's keep this easy and just write a script to handle this. Lucky for you,
the hard part has been done below.

> Note: Only have to create this once. Can skip this part if it's arleady
> there.

```bash
cd ~
nano update-links.sh
```

Use your favorite text editor here, I personally prefer Nano. Copy the below
lines into the editor, exit and save.

> Again, we are assuming you will be running this from your home folder with
> MegaMek installed and running under it. Replace `stable` with whatever
> folder name you are using.

```bash
#!/bin/sh
# This script will link up each folder under `shared` to it's place under
# the `stable` folder for use with MegaMek as a dedicated server.
ln -s shared/boards stable/data/boards/custom
ln -s shared/images stable/data/images/custom
ln -s shared/mechfiles stable/data/mechfiles/custom
ln -s shared/scenarios stable/data/scenarios/custom
ln -s shared/forcegenerator stable/data/forcegenerator/custom
ln -s shared/mapgen stable/data/mapgen/custom
ln -s shared/names stable/data/names/custom
ln -s shared/rat stable/data/rat/custom
ln -s shared/sounds stable/data/sounds/custom
```

All this is doing is taking whatever you have in the shared folder and
symbolic linking them to a `custom` folder under each respective area.
This way all your shared data stays in one place and all you have to do
is run a single command during updates to re-link them all up.

But this alone with work. On *nix based system, there is a permission for
scripts that needs to be enabled so it can run with minimal fuss. To do so,
run the following command in the same directory that the script is in:

```bash
chmod +x ./update-links.sh
```

This adds the `Execute` bit to the file so that all you have to do is type
`./update-links.sh` for it to run.

Once this is done, make sure to restart MegaMek so that it sees the new
files.

```bash
sudo systemctl start megamek.service
sudo systemctl status megamek.service
```

So long as everything shows up as running, you're custom units are installed.

> Note: You are responsible for making sure your custom files work with
> newer versions of MegaMek. Test them locally on your machine BEFORE
> uploading or linking them.
