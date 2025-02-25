# setup-gre-tunnel
these worked for me:P

## Create new route6 object
https://apps.db.ripe.net/db-web-ui/webupdates/create/RIPE/route6
```
mnt-by: Your maintainer 
route6: Your IP prefix (prefix (/48 /44 /melon))
origin: Your ASN (AS49184)
source: RIPE
```
## fill config & setup pathvector

### install pathvector & bird2
```
curl https://repo.pathvector.io/pgp.asc > /usr/share/keyrings/pathvector.asc
```

```
echo "deb [signed-by=/usr/share/keyrings/pathvector.asc] https://repo.pathvector.io/apt/ stable main" > /etc/apt/sources.list.d/pathvector.list
```
```
apt update && apt install -y bird2 pathvector
```

### Setup pathvector
fill this config: /etc/pathvector.yml
follow this https://pathvector.io/docs/examples

run:
```
systemctl enable --now bird
```
```
systemctl status bird
```

then (if first time, wait 24h)
```
pathvector generate
```

to check the status run 
```
pathvector status # Status should be Established to make sure it will be announced (else you won't be able to use your ipv6
```

## use the ips 
(these worked for me)
```
sysctl -w net.ipv6.ip_nonlocal_bind=1
```
```
echo 'net.ipv6.ip_nonlocal_bind = 1' >> /etc/sysctl.conf
```

```
sudo ip tunnel add gre1 mode gre remote THEIR_SERVER_IPV4 local YOUR_SERVER_IPV4 ttl 255
```
no buffer? ```sudo ip tun del gre1```

```
sudo ip link set gre1 up
```

```
sudo ip route replace ::/0 dev gre1
```
```
ip -6 route add default dev gre1
```

```
sudo ip addr add [SUBNET]::2/48 dev gre1 # this will assign 1 ip [SUBNET]::2
```

To assign the entire subnet use lo 
```
ip -6 route add local [SUBNET]::/48 dev lo
```

## Testing
```
ping6 -I [SUBNET]::4 google.com
ping6 -I [SUBNET]::3 google.com
ping6 -I [SUBNET]::2 google.com
```
