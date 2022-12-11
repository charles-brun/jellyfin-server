# JELLYFIN SERVER

Use Jellyfin with a NFS for centralized medias, a reverse proxy and as many servers as you want !

## REQUIRED

For all devices :

- Rocky2 OS installed and updated
- defined static IP on a same LAN

## NFS SERVER

Install `nfs-utils` package and enable its associated services :

```
sudo dnf install nfs-utils -y
sudo systemctl enable --now rcpbind
sudo systemctl start rcpbind
sudo systemctl enable --now nfs
sudo systemctl start nfs-server
```

Create the following folders :

```
sudo mkdir /var/nfs/config -p
sudo mkdir /var/nfs/cache
sudo mkdir /var/nfs/media
sudo mkdir /var/nfs/media2
```

Add this line to `/etc/exports`, replacing the IP address with your jellyfin servers' network address

```        
/var/nfs        10.104.1.0/24(rw,sync,no_subtree_check)
```

Configure your firewall :

```
sudo firewall-cmd --permanent --add-service=nfs
sudo firewall-cmd --permanent --add-service=mountd
sudo firewall-cmd --permanent --add-service=rpc-bind
sudo firewall-cmd --reload
```

## SERVER(S) SETUP

Create a sudo user `server`

Copy the `docker-compose.yml` file on `/home/server/`
Change this line according to your server's IP :

```
    ports:
      - "10.104.1.10:7777:8096"
```

Install `nfs-utils` package :

```
sudo dnf install nfs-utils -y
```

Add this line to `/etc/fstab`, replacing the IP address with your NFS Server's

```
10.104.1.100:/var/nfs   /home/server/nfs    nfs     defaults        0       0
```

Configure your firewall :

```
sudo firewall-cmd --add-port=7777/tcp --permanent
sudo firewall-cmd --restart
```

Run the container on your server(s) :

```
sudo docker compose up -d
```

## PROXY SETUP

Install nginx

Add the `proxy.conf` file to `/etc/nginx/conf.d/`

Change these lines according to the number of servers and their IP :

```
upstream servers {
        server 10.104.1.9:7777;
        server 10.104.1.10:7777;
}
```

Configure your firewall :

```
sudo firewall-cmd --add-port=80/tcp --permanent
sudo firewall-cmd --add-port=443/tcp --permanent
sudo firewall-cmd --restart
```

Create an SSL certificate and a key, then move them to `/srv`. Set Common Name to `jellyfin.server`:

```
openssl req -new -newkey rsa:2048 -days 365 -nodes -x509 -keyout jellyfin-server.key -out jellyfin-server.crt

 > Common Name (eg, your name or your server's hostname) []: jellyfin.server

mv jellyfin-server.* /srv
```

## MONITORING

You can setup monitoring on any device you want.

Install Netdata and get required packages :

```
dnf install epel-release -y
wget -O /tmp/netdata-kickstart.sh https://my-netdata.io/kickstart.sh && sh /tmp/netdata-kickstart.sh
```

Start and enable it :

```
systemctl start netdata
systemctl enable netdata
systemctl status netdata
```

Configure your firewall :

```
firewall-cmd --permanent --add-port=19999/tcp
firewall-cmd --reload
```

Enter the following URL on the browser to access the Netdata dashboard. By default, Netdata works on 19999 port : `http://<Enter Your IP Here>:19999/`

### Configure alarms for Discord

Get a webhook URL as given by Discord. Create a webhook by following the official [Discord documentation](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks)

Edit the file `/etc/netdata/health_alarm_notify.conf` as follow, and change the `DISCORD_WEBHOOK_URL` value :

```
###############################################################################
# sending discord notifications

# note: multiple recipients can be given like this:
#                  "CHANNEL1 CHANNEL2 ..."

# enable/disable sending discord notifications
SEND_DISCORD="YES"

# Create a webhook by following the official documentation -
# https://support.discordapp.com/hc/en-us/articles/228383668-Intro-to-Webhooks
DISCORD_WEBHOOK_URL="https://discord.com/api/webhooks/xxxxxxxxx/xxxxxxxxxxxxxxxxxx"

# if a role's recipients are not configured, a notification will be send to
# this discord channel (empty = do not send a notification for unconfigured
# roles):
DEFAULT_RECIPIENT_DISCORD="alarms"

role_recipients_discord[sysadmin]="test"
role_recipients_discord[dba]="test"
role_recipients_discord[webmaster]="test"
```

