# Create Your Own Streaming Music Service with Plex Media Server

Demo for the Rock River Linux User Group - 11 March 2021

## Musick Has Charms to Sooth a Savage Breast

With the death of Google Play Music and what I considered a subpar user experience with its replacement, YouTube Music, I needed a new way to stream my ripped CDs and [Bandcamp](https://bandcamp.com) downloads on the go. I was not keen on most of the alternatives as I would either have to pay to upload my existing music collection or simply wouldn't be able to. I looked at a number of self-hosted options before finally settling on [Plex Media Server](https://plex.tv). While Plex is most widely known as a video streaming platform, it is also a good music platform. In this demo, we'll set up a headless Plex Media Server together on Debian Linux (or one of its derivatives - I'm using an Ubuntu 20.04 LTS server VM, but you could just as easily use a spare PC or Raspberry Pi) at home. When we're done, you should be able to enjoy your personal music collection anywhere you have an Internet connection, on multiple devices and platforms.

Of course, you can also use a RedHat-based distribution (see the Plex installation page for those instructions) and/or set your instance up in a cloud provider if you so choose.

## Batteries Not Included

For this demo, I assume you're comfortable setting up a basic install of a Debian-based OS, so I won't be covering it. Setting up the required port forwarding on your router, DNS resolution, and IP addressing in your home network is also out of scope. There are many different ways to manage your media storage on the Plex server, too, so I'll leave that up to you.

You will need a free Plex account, even if you're self-hosting the media server, which you can sign up for by clicking the appropriate button on the Plex website.

## Some Assembly Required

Let's get started by logging into the system via SSH.

Once you're connected, add the Plex repository to Apt's sources: `echo deb https://downloads.plex.tv/repo/deb public main | sudo tee /etc/apt/sources.list.d/plexmediaserver.list`

Next, import the repo signing key: `curl https://downloads.plex.tv/plex-keys/PlexSign.key | sudo apt-key add -`

Now, update Apt's package cache and install the Plex Media Server package: `sudo apt-get update && sudo apt-get install -y plexmediaserver` (If you're prompted to overwrite **/etc/apt/sources.list.d/plexmediaserver.list**, choose 'Y' to use the repository's version. This will prevent you from being prompted for this on future updates.)

Let's make sure the Plex service starts at boot time by enabling the SystemD unit: `sudo systemctl enable plexmediaserver.service`

Before we configure the application via the web interface, let's create someplace to house our media. For this demo, we're just going to create a directory in the root filesystem, but for actual use, I'd recommend setting up a separate LVM or mounting an external hard drive and creating the necessary directories there. In any case, let's add that music directory: `sudo mkdir -p /media/music` (If you were also going to use Plex for other media, you'd want to create separate subdirectories for those as well.)

The 'plex' user needs to have full ownership of the media directories, so let's set that now: `sudo chown -R plex:plex /media`

In order to make our lives easier when adding music to our collection, and do some least-privilege security, we're going to create a normal user account that we'll use only for uploading our files over SFTP. For this demo, we'll use 'musiclover' as the account name and 'i<3PJD' as the password, but you should create your own memorable credentials. Invoke `sudo adduser --gecos "Music Lover" musiclover` and supply the password when prompted.

Next, we're going to update the SSH configuration to lock our upload user into their home directory with a chroot. Add the following block to the end of the **/etc/ssh/sshd_config** file:

```
Match User musiclover
        ChrootDirectory %h
        X11Forwarding no
        AllowTcpForwarding no
        PermitTTY no
        ForceCommand internal-sftp
        PasswordAuthentication yes
```

Restart the SSH service to make the change effective: `sudo systemctl restart ssh.service`

We'll also need to modify some settings around the user's home directory to make the chroot work: `sudo chown root:root /home/musiclover && sudo chmod 0755 /home/musiclover`

Next, let's create a SystemD bind mount unit file at **/etc/systemd/system/home-musiclover-mnt.mount** with the following contents:

```
[Unit]
Description=Bind mount for Plex media
Requires=network-online.target
After=plexmediaserver.service

[Mount]
What=/media
Where=/home/musiclover/mnt
Type=none
Options=bind,defaults
[Install]
WantedBy=multi-user.target
```

Make sure SystemD picks up the unit file: `sudo systemctl daemon-reload`

Create the mount target directory inside musiclover's home directory by invoking this command: `sudo mkdir /home/musiclover/mnt`

Now, enable the unit so it's mounted at boot time and start the mount so it's mounted immediately: `sudo systemctl enable home-musiclover-mnt.mount && sudo systemctl start home-musiclover-mnt.mount`

Verify the mount is active: `mount -l | grep music`

Validate the chroot by attempting a normal SSH connection to the server as the musiclover user. You should receive a response like this:

```
PTY allocation request failed on channel 0
This service allows sftp connections only.
Connection to 1.2.3.4 closed.
```

We'll set the media directory to be writable by the 'plex' group and add our upload user to that group: `sudo chmod -R 0770 /media && sudo usermod -a -G plex musiclover`

From here, we'll walk through the web setup interface by connecting to http://plex_hostname_or_ip_address:32400/web and signing in with your Plex account when prompted. If your Plex server is not on the same network as your workstation, you might need to set up an SSH tunnel as mentioned in the installation docs.

Be sure to point the Music library at the **/media/music** directory during the setup wizard.

Once the wizard is complete, use your SFTP client to upload your music to the server, then return to Plex and start listening!

## Et Finis

I hope you found the demo informative! I'd love to hear about your experience with streaming music from Plex or another self-hosted, Linux-based option. Feel free to shoot me an email: victor alpha yankee golf hotel at victor alpha yankee golf hotel dot com

## Works Cited

“Installation.” Plex Support, 9 July 2020, <https://support.plex.tv/articles/200288586-installation/> 

“Enable Repository Updating for Supported Linux Server Distributions.” Plex Support, 24 May 2019, <https://support.plex.tv/articles/235974187-enable-repository-updating-for-supported-linux-server-distributions/>