# jellyfin-server

Create directories in your server :

```
mkdir config
mkdir cache
```

Add the `docker-compose.yml` file

Configure your firewall :

```
sudo firewall-cmd --add-port=8096/tcp --permanent
sudo firewall-cmd --restart
```

Run the container :

```
sudo docker compose up
```


