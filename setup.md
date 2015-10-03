# Preparation

Installing needed dependencies:

`apt-get install -y unionfs-fuse encfs fuse python3-pip htop iftop git aptitude
software-properties-common`

## Installing [ACD_CLI](https://github.com/yadayada/acd_cli "ACD_CLI"):

`pip3 install --upgrade git+https://github.com/yadayada/acd_cli.git@dev`

## Updating, updating:

`apt-get update`
`apt-get upgrade`

## User creation: 

### Creating **plex** user: 

`adduser plex`
`addgroup plex fuse`

### Creating *admin* user: 

`adduser ctejada`

#### Create ssh-keys

## Directory structure:

 * ** `AmazonCloudDrive` mount point**: `/home/plex/.acd`
 * ** Decrypted downloaded files**: `/home/plex/.downloaded`
 * ** Encrypted downloaded files**: `/home/plex/.local_media`
 * ** Merged media library (encrypted)**: `/home/plex/.media`
 * ** Merged media library**: `/home/plex/media`
 
# File System Setup

## Creating pertinent directories:

  * `su - plex`: Using `plex` user.
  * `mkdir .acd media`: Creating `.acd` and `media` directories on `plex` home
  directory.

## Initializing `acd_cli`:

  * Issue an `acd_cli sync`.
  * Go to the [authentication page](https://tensile-runway-92512.appspot.com/).
  * Log in and get the `oauth_data` json file.
  * Mount Amazon Cloud Drive in `/home/plex/.acd`: `acd_cli mount .acd`

## Decrypt media files:
> Note that the mountpoint shouldnt exist at the moment of decryption.

  * Get `encfs.xml`.
  * Decrypt media from Amazon Cloud Drive: `ENCFS6_CONFIG='setup/encfs.xml' encfs
    /home/plex/.acd/Media/ /home/plex/.media`

## "Merge" Amazon Cloud Drive media with local media:

  `unionfs-fuse -o cow /home/plex/.local_media/=RW:/home/plex/.media/=RO /home/plex/media/`

# uTorrent:

## Downloading: uTorrent

  ```
wget http://download-new.utorrent.com/os/linux-x64-ubuntu-13-04/track/beta/endpoint/utserver/
mv index.html utorrent.tar.gz
tar -xvf utorrent.tar.gz
mv utorrent-server* utorrent
rm -rf utorrent.tar.gz
mkdir utorrent/downloaded/
  ```

## To Run uTorrent:

`cd utorrent
./utserver -daemon`

In a browser go to http://IP:8080/gui. Login is admin with no password.
Change this password then ensure you can relogin Settings -> Web UI -> Password
and Save settings. Once a torrent has downloaded, make it move to `/home/plex/utorrent/downloaded/`.
Set up your SOCKS proxy

## Starting uTorrent on reboot:

Adding the entry to `crontab`:
  `crontab -e`

  `@reboot /home/plex/utorrent/utserver -daemon`

## Proxy Setup:

Type:  Socks5
Proxy: proxy.btguard.com
Port:  1025

# PleX installation:

> Go to https://plex.tv/downloads to get the Ubuntu 64 download


```
wget https://downloads.plex.tv/plex-media-server/0.9.12.4.1192-9a47d21/plexmediaserver_0.9.12.4.1192-9a47d21_amd64.deb
dpkg -i plex*.deb
rm -rf plex*.deb
```
Open an ssh tunnel: ssh user@host -L 32400:localhost:32400

In a browser go to http://localhost:32400/web. Login. Preferences -> Server - ensure logged in.
-> Network, disable local discovery. Add your libraries to your server on the left. Point to the decrypted libraries.



> This is optional.
# FileBot installation

> Get the latest .deb from http://sourceforge.net/projects/filebot/files/filebot/

```
wget http://downloads.sourceforge.net/project/filebot/filebot/FileBot_4.6/filebot_4.6_i386.deb?r=http%3A%2F%2Fsourceforge.net%2Fprojects%2Ffilebot%2Ffiles%2Ffilebot%2FFileBot_4.6%2F&ts=1439257518&use_mirror=iweb
dpkg -i filebot.deb
rm -rf filebot.deb
```

## Install Java for Filebot

Java 8 is not on default repositories, so you will need to add a PPA.

```
add-apt-repository ppa:webupd8team/java
apt-get update
apt-get install oracle-java8-installer
```

## Configure FileBot execution:

```
#!/bin/bash
HOME="/home/plex" filebot -script /home/plex/scripts/amc.groovy
--output "/home/plex/sorted" --log-file /home/plex/filebot.log -non-strict
--def plex=127.0.0.1 --def excludeList=/home/plex/filebot.cache "/home/plex/utorrent/downloaded"
```
