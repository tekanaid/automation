# Automation
This is mainly a docker-compose file to set up some home automation stuff. The docker-compose.yml is for x86 machines and the docker-compose-rpi.yml is for the ARM architecture.

## DuckDNS for x86:

Make sure to replace the below under the duckdns section with your own parameters:

TOKEN: your own token

SUBDOMAINS: your own subdomains

## DuckDNS for ARM:

Make sure to replace the below under the duckdns section with your own parameters:

TOKEN: your own token

DOMAIN: your own full domain including the .duckdns.org

You can also just use the following docker run command instead of docker-compose:
`docker run --name=duckdns -e TZ=America/Toronto -e DOMAIN=yousry.duckdns.org -e TOKEN=e11e2bb7-a87b-4e65-b1c9-ca5c1c503828 --restart=always tekanaid/duckdns-rpi:110`

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
