# JELLYFIN SERVER

## SERVER(S) SETUP

Create directories in your server(s) :

```
mkdir config
mkdir cache
```

Add the `docker-compose.yml` file
Change this line according to your server's IP :

```
    ports:
      - "10.104.1.10:7777:8096"
```

Configure your firewall :

```
sudo firewall-cmd --add-port=7777/tcp --permanent
sudo firewall-cmd --restart
```

Run the container on your server(s) :

```
sudo docker compose up
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