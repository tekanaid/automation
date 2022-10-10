# Automation
This is mainly a docker-compose file to set up some home automation stuff. The docker-compose.yml is for x86 machines and the docker-compose-rpi.yml is for the ARM architecture.

## Synology

Migrating Apps between volumes
https://veducate.co.uk/synology-moving-a-package-between-volumes/

Also need to update the sym links
sudo rm /var/services/homes 	sudo ln -s /volume1/homes /var/services/homes 
sudo rm /var/services/music 	sudo ln -s /volume1/music /var/services/music 
sudo rm /var/services/pgsql 	sudo ln -s /volume1/@database/pgsql /var/services/pgsql 
sudo rm /var/services/photo 	sudo ln -s /volume1/photo /var/services/photo 

May need to restart this service:
sudo systemctl restart pgsql-adapter

Run a 
```bash
diff /volume2 /volume1 | grep -i "Only in /volume2"
```

Then start copying folders over with permissions:
```bash
sudo cp -rp /volume2/@ActiveBackup /volume1/@ActiveBackup
sudo cp -rp /volume2/@ActiveBackup-GSuite /volume1/@ActiveBackup-GSuite
sudo cp -rp /volume2/@autoupdate /volume1/@autoupdate
sudo cp -rp /volume2/@cloudsync /volume1/@cloudsync
sudo cp -rp /volume2/@deleted_subvol /volume1/@deleted_subvol
```


## DuckDNS for x86:

Make sure to replace the below under the duckdns section with your own parameters:

TOKEN: your own token

SUBDOMAINS: your own subdomains

## DuckDNS for ARM:

Make sure to replace the below under the duckdns section with your own parameters:

TOKEN: your own token

DOMAIN: your own full domain including the .duckdns.org

You can also just use the following docker run command instead of docker-compose:
```docker run -d --name=duckdns -e TZ=America/Toronto -e DOMAIN=yousry.duckdns.org -e TOKEN=e11e2bb7-a87b-4e65-b1c9-ca5c1c503828 --restart=always tekanaid/duckdns-rpi:110```

## OpenVPN-Access Server for x86:

Make sure to replace the ens160 in the docker-compose file with the proper interface that you get an IP on for the server
INTERFACE: ens160

__Setting up the application__

The admin interface is available at https://<ip>:943/admin with a default user/password of admin/password

During first login, make sure that the "Authentication" in the webui is set to "Local" instead of "PAM". Then set up the user accounts with their passwords (user accounts created under PAM do not survive container update or recreation).

The "admin" account is a system (PAM) account and after container update or recreation, its password reverts back to the default. It is highly recommended to block this user's access for security reasons. In my experience, doing 3) below is sufficient and I couldn't delete the admin user from the gui.
1) Set another user as an admin,
2) Delete the "admin" user in the gui,
3) Modify the as.conf file under config/etc and replace the line boot_pam_users.0=admin with #boot_pam_users.0=admin (this only has to be done once and will survive container recreation)

You can find more information here:
https://hub.docker.com/r/linuxserver/openvpn-as/

## OpenVPN for ARM:

### Define the environment variable below

OVPN_DATA="ovpn-data"

### Initialize the $OVPN_DATA container that will hold the configuration files and certificates
docker volume create --name $OVPN_DATA
docker run -v $OVPN_DATA:/etc/openvpn --rm evolvedm/openvpn-rpi ovpn_genconfig -u udp://VPN.SERVERNAME.COM
docker run -v $OVPN_DATA:/etc/openvpn --rm -it evolvedm/openvpn-rpi ovpn_initpki
### Start OpenVPN server process
docker run -v $OVPN_DATA:/etc/openvpn -d -p 1194:1194/udp --cap-add=NET_ADMIN --restart=always --name openvpn_server evolvedm/openvpn-rpi

### Generate a client certificate without a passphrase
docker run -v $OVPN_DATA:/etc/openvpn  --rm -it evolvedm/openvpn-rpi easyrsa build-client-full CLIENTNAME nopass

### Retrieve the client configuration with embedded certificates
docker run -v $OVPN_DATA:/etc/openvpn --rm evolvedm/openvpn-rpi ovpn_getclient CLIENTNAME > CLIENTNAME.ovpn

This was taken from the following links:
https://github.com/kylemanna/docker-openvpn
https://hub.docker.com/r/evolvedm/openvpn-rpi 

## Install Armbian for OrangePI

Download the image from here:
https://www.armbian.com/orange-pi-zero/

Get balena etcher to flash the sd card from here:
https://www.balena.io/etcher/

## Download Docker

Normally as you usually do it. I used Ubuntu installation because Debian at the time didn't have the repo for 22.04 Jammy.

## Wireguard

Use the `docker-compose-rpi.yml` file.

To see the peer QR code to use with the Android wireguard app:
```bash
docker exec -it wireguard /app/show-peer 1
```
or
```bash
docker exec -it wireguard /app/show-peer samMobile
```

### Good Guides for Wireguard
This is a good guide for using wireguard as a container on a pi:
https://www.addictedtotech.net/home-vpn-using-wireguard-docker-on-a-raspberry-pi-4/

This is a good guide for using wireguard directly on a pi and works with orangepi too:
https://www.wundertech.net/setup-wireguard-on-a-raspberry-pi-vpn-setup-tutorial/

## Static IP and Networking for the OrangePI Armbian

Add static ip and DNS to the OrangePI

```bash
sudo cat /etc/netplan/armbian-default.yaml
network:
  ethernets:
      eth0:
          dhcp4: false
          addresses: [192.168.1.99/24]
          routes:
          - to: default
            via: 192.168.1.254
          nameservers:
            addresses: [192.168.1.80]
  version: 2
  renderer: NetworkManager
```
  
  Then run:
  
  ```bash
  sudo netplan apply
  ```
  
  and check with:
  
  ```bash
  ip a
  ```
