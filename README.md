# OpenVPN for Docker

[![Build Status](https://travis-ci.org/pankajthakur888/docker-openvpn.svg)](https://travis-ci.org/pankajthakur888/docker-openvpn)
[![Docker Stars](https://img.shields.io/docker/stars/pankajthakur888/openvpn.svg)](https://hub.docker.com/r/pankajthakur888/openvpn/)
[![Docker Pulls](https://img.shields.io/docker/pulls/pankajthakur888/openvpn.svg)](https://hub.docker.com/r/pankajthakur888/openvpn/)
[![ImageLayers](https://images.microbadger.com/badges/image/pankajthakur888/openvpn.svg)](https://microbadger.com/#/images/pankajthakur888/openvpn)
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2Fpankajthakur888%2Fdocker-openvpn.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2Fpankajthakur888%2Fdocker-openvpn?ref=badge_shield)

OpenVPN server in a Docker container complete with an EasyRSA PKI CA.

Extensively tested on [Digital Ocean $5/mo node](http://bit.ly/1C7cKr3) and has
a corresponding [Digital Ocean Community Tutorial](http://bit.ly/1AGUZkq).

#### Upstream Links

* Docker Registry @ [pankajthakur888/openvpn](https://hub.docker.com/r/pankajthakur888/openvpn/)
* GitHub @ [pankajthakur888/docker-openvpn](https://github.com/pankajthakur888/docker-openvpn)

## Quick Start

* Pick a name for the `$OVPN_DATA` data volume container. It's recommended to
  use the `ovpn-data-` prefix to operate seamlessly with the reference systemd
  service.  Users are encouraged to replace `example` with a descriptive name of
  their choosing.

      OVPN_DATA="ovpn-data-example"

* Initialize the `$OVPN_DATA` container that will hold the configuration files
  and certificates.  The container will prompt for a passphrase to protect the
  private key used by the newly generated certificate authority.

      docker volume create --name $OVPN_DATA
      docker run -v $OVPN_DATA:/etc/openvpn --rm pankajthakur888/openvpn ovpn_genconfig -u udp://VPN.SERVERNAME.COM
      docker run -v $OVPN_DATA:/etc/openvpn --rm -it pankajthakur888/openvpn ovpn_initpki

* Start OpenVPN server process

      docker run -v $OVPN_DATA:/etc/openvpn -d -p 1194:1194/udp --cap-add=NET_ADMIN pankajthakur888/openvpn

* Generate a client certificate without a passphrase

      docker run -v $OVPN_DATA:/etc/openvpn --rm -it pankajthakur888/openvpn easyrsa build-client-full CLIENTNAME nopass

* Retrieve the client configuration with embedded certificates

      docker run -v $OVPN_DATA:/etc/openvpn --rm pankajthakur888/openvpn ovpn_getclient CLIENTNAME > CLIENTNAME.ovpn

## Next Steps

### More Reading

Miscellaneous write-ups for advanced configurations are available in the
[docs](docs) folder.

### Systemd Init Scripts

A `systemd` init script is available to manage the OpenVPN container.  It will
start the container on system boot, restart the container if it exits
unexpectedly, and pull updates from Docker Hub to keep itself up to date.

Please refer to the [systemd documentation](docs/systemd.md) to learn more.

### Docker Compose

If you prefer to use `docker-compose` please refer to the [documentation](docs/docker-compose.md).

## Debugging Tips

* Create an environment variable with the name DEBUG and value of 1 to enable debug output (using "docker -e").

        docker run -v $OVPN_DATA:/etc/openvpn -p 1194:1194/udp --cap-add=NET_ADMIN -e DEBUG=1 pankajthakur888/openvpn

* Test using a client that has openvpn installed correctly

        $ openvpn --config CLIENTNAME.ovpn

* Run through a barrage of debugging checks on the client if things don't just work

        $ ping 8.8.8.8    # checks connectivity without touching name resolution
        $ dig google.com  # won't use the search directives in resolv.conf
        $ nslookup google.com # will use search

* Consider setting up a [systemd service](/docs/systemd.md) for automatic
  start-up at boot time and restart in the event the OpenVPN daemon or Docker
  crashes.

## How Does It Work?

Initialize the volume container using the `pankajthakur888/openvpn` image with the
included scripts to automatically generate:

- Diffie-Hellman parameters
- a private key
- a self-certificate matching the private key for the OpenVPN server
- an EasyRSA CA key and certificate
- a TLS auth key from HMAC security

The OpenVPN server is started with the default run cmd of `ovpn_run`.

The configuration is located in `/etc/openvpn`, and the Dockerfile
declares that directory as a volume. It means that you can start another
container with the `-v` argument, and access the configuration.
The volume also holds the PKI keys and certs so that it could be backed up.

To generate a client certificate, `pankajthakur888/openvpn` uses EasyRSA via the
`easyrsa` command in the container's path.  The `EASYRSA_*` environmental
variables place the PKI CA under `/etc/openvpn/pki`.

Conveniently, `pankajthakur888/openvpn` comes with a script called `ovpn_getclient`,
which dumps an inline OpenVPN client configuration file.  This single file can
then be given to a client for access to the VPN.

To enable Two Factor Authentication for clients (a.k.a. OTP) see [this document](/docs/otp.md).

