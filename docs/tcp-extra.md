
## TCP connention

```sh

```

```sh
docker run -v ovpn-data:/etc/openvpn --rm pankajthakur888/openvpn ovpn_genconfig -u tcp://$(hostname -I | awk '{print $1}'):443
```

```sh
docker run -v ovpn-data:/etc/openvpn --rm -it pankajthakur888/openvpn ovpn_initpki
```

```sh
docker run -v ovpn-data:/etc/openvpn -d -p 443:1194/tcp --cap-add=NET_ADMIN pankajthakur888/openvpn
```

```sh
docker run -v ovpn-data:/etc/openvpn --rm -it pankajthakur888/openvpn easyrsa build-client-full pankaj nopass
```

```sh
docker run -v ovpn-data:/etc/openvpn --rm pankajthakur888/openvpn ovpn_getclient pankaj > pankaj.ovpn
```
