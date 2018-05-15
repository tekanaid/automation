# Automation
This is mainly a docker-compose file to set up some home automation stuff. 

## DuckDNS:

Make sure to replace the below under the duckdns section with your own parameters:

TOKEN: your own token

SUBDOMAINS: your own subdomains

## OpenVPN:

Make sure to replace the ens160 in the docker-compose file with the proper interface that you get an IP on for the server
INTERFACE: ens160

__Setting up the application__
The admin interface is available at https://<ip>:943/admin with a default user/password of admin/password

During first login, make sure that the "Authentication" in the webui is set to "Local" instead of "PAM". Then set up the user accounts with their passwords (user accounts created under PAM do not survive container update or recreation).

The "admin" account is a system (PAM) account and after container update or recreation, its password reverts back to the default. It is highly recommended to block this user's access for security reasons:
1) Set another user as an admin,
2) Delete the "admin" user in the gui,
3) Modify the as.conf file under config/etc and replace the line boot_pam_users.0=admin with #boot_pam_users.0=admin (this only has to be done once and will survive container recreation)

You can find more information here:
https://hub.docker.com/r/linuxserver/openvpn-as/
