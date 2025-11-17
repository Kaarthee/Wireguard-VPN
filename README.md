WireGuard VPN Server Configuration (Kali Linux)

This document describes the server-side configuration process for deploying a WireGuard VPN on a Kali Linux virtual machine. It includes all commands executed along with formal, technical explanations suitable for documentation or academic submission.

1. System Preparation
1.1 Update Package Index
sudo apt update


This command refreshes the APT package index to ensure that the latest metadata and package versions are available prior to installation. It prevents issues caused by outdated repository information.

2. WireGuard Installation
sudo apt install wireguard -y


This installs the WireGuard kernel module and user-space tools (wg, wg-quick).
The -y flag automatically confirms the installation prompt.

3. Directory Setup and Permission Hardening
sudo mkdir -p /etc/wireguard
cd /etc/wireguard
sudo umask 077


Explanation:

/etc/wireguard is the standard location for WireGuard configuration files.

umask 077 ensures that newly created files are readable and writable only by the root user (rw-------).
This is essential because WireGuard private keys must not be accessible to other users.

4. Generating Server Keys
sudo wg genkey | sudo tee server_private.key | sudo wg pubkey > server_public.key
sudo chmod 600 server_private.key


Explanation:

wg genkey produces a cryptographically secure private key.

tee server_private.key writes the private key to a file.

wg pubkey derives the public key from the private key.

chmod 600 restricts access to the private key so only the root user can read or modify it.

The following files are created:

server_private.key – confidential and must remain on the server.

server_public.key – used by clients.

5. Creating the WireGuard Server Configuration File

Open the configuration file with:

sudo nano /etc/wireguard/wg0.conf


Insert the following template (with the appropriate key values substituted):

[Interface]
Address = 10.10.0.1/24
ListenPort = 51820
PrivateKey = <server_private_key_here>

PostUp   = sysctl -w net.ipv4.ip_forward=1; \
           iptables -t nat -A POSTROUTING -s 10.10.0.0/24 -o eth0 -j MASQUERADE; \
           iptables -A FORWARD -i wg0 -o eth0 -j ACCEPT; \
           iptables -A FORWARD -i eth0 -o wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT

PostDown = iptables -t nat -D POSTROUTING -s 10.10.0.0/24 -o eth0 -j MASQUERADE; \
           iptables -D FORWARD -i wg0 -o eth0 -j ACCEPT; \
           iptables -D FORWARD -i eth0 -o wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT; \
           sysctl -w net.ipv4.ip_forward=0

SaveConfig = true

[Peer]
PublicKey = <client_public_key_here>
AllowedIPs = 10.10.0.2/32

Explanation of Configuration Fields
Interface Section

Address
Defines the server’s internal VPN IP address (gateway for the VPN subnet).

ListenPort
Specifies the UDP port on which the WireGuard server accepts connections.

PrivateKey
Contains the server’s private key in plain text. Permissions must remain restricted.

PostUp
Commands executed automatically when the interface is brought up:

Enables IPv4 packet forwarding.

Creates a NAT (masquerading) rule for outbound traffic via eth0.

Allows forwarding between the WireGuard interface (wg0) and the external interface (eth0).

Allows return traffic for already established connections.

PostDown
Reverses all firewall and forwarding changes applied by PostUp.

SaveConfig
Enables WireGuard to write runtime modifications back into the configuration file.

Peer Section

PublicKey
The public key of the client allowed to connect to this server.

AllowedIPs
Specifies which internal VPN IP address belongs to this client.
The CIDR /32 represents a single IP (10.10.0.2).

6. Securing the Configuration File
sudo chmod 600 /etc/wireguard/wg0.conf


This ensures that the configuration file, which contains private keys, is accessible only to root.

7. Starting and Enabling the WireGuard Service
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0


Explanation:

wg-quick@wg0 is a systemd service that brings up the wg0 interface using the configuration file.

enable ensures the VPN starts automatically at boot.

start activates the VPN immediately.

Verify status:

sudo systemctl status wg-quick@wg0
sudo wg show

8. Manual Firewall and Routing Commands (Optional)

These were used during testing and troubleshooting:

sudo sysctl -w net.ipv4.ip_forward=1

sudo iptables -t nat -A POSTROUTING -s 10.10.0.0/24 -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i wg0 -o eth0 -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT


In the final deployment, all these rules are handled automatically by PostUp and PostDown in wg0.conf.