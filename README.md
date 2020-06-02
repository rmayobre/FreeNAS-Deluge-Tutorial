# Install Deluge on FreeNAS Jail
How to install Deluge and setup WebUI and Desktop Interface (pick the interface you want, or use both).

## Prepare Jail

### Prepare a dataset and pool for your jail

If you have a designated Dataset or Pool you wish to use, then use; otherwise create your dataset and pool because you will need a location to store data.

### Create the Jail:

1. Create jail using FreeNAS Web Interface.
2. Name it "deluge" (or something to remember).
3. Configure IP manually (do not use DHCP, it works best to have a static IP).

### Create user for Jail:
1. Create user from FreeNAS Web Interface
2. Set similar values for user (do **NOT** change last two values)
```
username : deluge
comment : deluge bit torrent client
homedirectory : /nonexistant
disable login password : checked
```
3. Make note of the ID created for user (you can look up user's ID in web interface: `Accounts -> Users`)

## SSH into Jail

### SSH into FreeNAS server

Linux / Mac / Unix
```
ssh [YourUserName]@[IpAddressOfYourFreenasBox]
```

Windows - install [PuTTY](https://putty.org/)

### Enter your jail

Before you enter your jail, you need to find out your jail's ID. While SSH'd into your FreeNAS server, use the `jls` command to get a list of your Jails (`JID` is your Jail's ID).
```
% jls
   JID  IP Address      Hostname        Path
     1                  plex-plexpass   /mnt/plex/iocage/jails/plex-plexpass/root
     4                  deluge          /mnt/plex/iocage/jails/deluge/root

```
In the example above, my deluge Jail's ID is `4`. Now that you have your Jail's ID, use the following command and replace `[JID]` with your Jail's ID:
```
sudo jexec [JID] tcsh
```
**NOTE** - `jexec` will most likely require super user permissions. If user is not `root`, then run the command in `sudo`.

You should now be inside your Jail. Here an example of your terminal when you are inside of your Jail:
```
% sudo jexec 4 tcsh
Password:
root@deluge:/ # 
```
**NOTE** - in the example above, `root` is the user and `deluge` is the name of the Jail I am inside of.

## Installing Deluge Daemon

### Update packages

```
pkg update
pkg upgrade
```

### Download the Portstree Structure
```
portsnap fetch extract
```

### Add User to Jail

Replace `[ID]` with the ID of the user you created for the Jail.
```
pw useradd -n deluge -u [ID] -c "Deluge BitTorrent Client" -s /sbin/nologin -w no
```

### Create a Directory for Deluge Config Files

```
mkdir -p /home/deluge/.config/deluge
chown -R deluge:deluge /home/deluge/
```

### Install Deluge

Make sure to disable GTK2 in the port configuration options (leave other configurations to default unless you know what you're doing).
```
cd /usr/ports/net-p2p/deluge && make WITHOUT_X11=yes install clean
```

### Enable Deluge Daemon on Startup

```
echo 'deluged_enable="YES"' >> /etc/rc.conf
echo 'deluged_user="deluge"' >> /etc/rc.conf
```

### Start and Stop Daemon to Create Config Files

```
service deluged start
service deluged stop
```

## Configure WebUI

Setup the deluge web daemon (delugew).

### Enable Deluge Web Daemon on Startup

```
echo 'deluge_web_enable="YES"' >> /etc/rc.conf
```

### Start Web Daemon

```
service deluge_web start
```

### Connect to WebUI

Head to your browser and connect to your Jail's Deluge web page. In your browser use this URL format: `[IpAddressOfTheJail]:8112`. When prompt, select the running daemon and connect to it. The second prompt will ask for your password (the default password is "deluge"). You can change the default password if you wish.

## Configure Desktop Interface
Install Deluge from this [guide](https://dev.deluge-torrent.org/wiki/UserGuide).

### Create a User

Deluge supports multiple user authentications. To create an admin user, use the following command while replacing `[username]` and `[password]` with your username and password of choice:
```
echo "[username]:[password]:10" >> /home/deluge/.config/deluge/auth
```

### Enable Remote Connections

Stop deluge's daemon:
```
service deluged stop
```

Edit deluge's core configurations
```
ee /home/deluge/.config/deluge/core.conf
```

Inside the `core.conf` file, replace this line:
```
"allow_remote": false,
```
with this line:
```
"allow_remote": true,
```

Click `ESC` + `Enter` key and click `Enter` to save changes.

Start deluge daemon
```
service deluged start
```

### Setup Deluge Desktop

Set Deluge desktop to "Thin Client" mode: `Edit` -> `Preferences`.

After applying "Thin Client" mode, Deluge will restart and prompt you with a connection window. Add your host with this information:
```
hostname : IP address of the jail
username : user name you just created
password : user password​
port     : DO NOT CHANGE THIS!!!
```
Click `add` and then connect to daemon.

## Mount dataset

Use FreeNAS web interface to add mount points for your jail. Shut down your jail and add the mount points. The mount points will act as your storage for all torrents.
