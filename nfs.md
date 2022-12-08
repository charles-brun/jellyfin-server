# NFS CONFIGURATION

## NFS Server

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

## NFS Clients

Install `nfs-utils` package :

```
sudo dnf install nfs-utils -y
```

Add this line to `/etc/fstab`, replacing the IP address with your NFS Server's

```
10.104.1.100:/var/nfs   /mnt    nfs     defaults        0       0
```