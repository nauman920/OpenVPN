# OpenVPN for Docker (Auto Setup)

OpenVPN server running inside a Docker container complete with an EasyRSA PKI CA.

This repository provides:

* Docker based OpenVPN server
* Persistent configuration volume
* Client add/remove scripts
* One-command auto installer


#### Upstream Links

* Docker Registry @ https://hub.docker.com/r/kylemanna/openvpn
* GitHub @ https://github.com/kylemanna/docker-openvpn


## Quick Start (Automatic Install)

Run on a fresh Ubuntu server:

    bash install-openvpn.sh <SERVER_PUBLIC_IP>

Example:

    bash install-openvpn.sh 34.210.10.50

The script will:

* install Docker
* pull OpenVPN image
* create volume
* generate configs
* initialize certificates
* start server
* create user scripts
* open firewall


## Manual Setup

### Update system

    sudo apt update && sudo apt upgrade -y


### Install Docker

    sudo apt install docker.io -y


### Pull OpenVPN image

    sudo docker pull kylemanna/openvpn


### Create data volume

    OVPN_DATA="ovpn-data"
    sudo docker volume create --name $OVPN_DATA


### Generate server configuration

Replace with your public / Elastic IP:

    sudo docker run -v $OVPN_DATA:/etc/openvpn --rm \
        kylemanna/openvpn ovpn_genconfig -u udp://<SERVER_IP>


### Initialize PKI

    sudo docker run -v $OVPN_DATA:/etc/openvpn --rm -it \
        kylemanna/openvpn ovpn_initpki


### Start OpenVPN server

    sudo docker run -v $OVPN_DATA:/etc/openvpn -d \
        -p 1194:1194/udp \
        --cap-add=NET_ADMIN \
        --name openvpn-server \
        kylemanna/openvpn


## Client Management

Create directory for client configs:

    mkdir User


### Generate client certificate

    sudo bash add-user.sh

This creates:

    User/CLIENTNAME.ovpn


### Revoke client

    sudo bash delete-user.sh

This will:

* revoke certificate
* regenerate CRL
* restart server
* delete client config


## Firewall

### UFW

    sudo ufw allow 1194/udp


### AWS

Allow Security Group:

    UDP 1194


## Useful Commands

### Running containers

    docker ps


### Restart server

    docker restart openvpn-server


### View logs

    docker logs openvpn-server


## File Structure

    .
    ├── install-openvpn.sh
    ├── add-user.sh
    ├── delete-user.sh
    └── User/
        └── *.ovpn# OpenVPN for Docker

