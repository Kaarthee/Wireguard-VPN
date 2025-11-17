# WireGuard VPN – Detailed Project Documentation

## Overview
This project demonstrates a complete, manually configured WireGuard VPN setup using:
- Kali Linux (as the VPN server)
- Windows (as the VPN client)
- VirtualBox networking
- NAT, routing, and forwarding rules

The goal of the project is to understand how a VPN works at a low level by manually:
- Generating keys
- Creating config files
- Enabling forwarding
- Adding firewall rules
- Connecting cross-platform clients
- Documenting the setup end-to-end

---

## 1. Server Setup (Kali Linux)

### 1.1 Install WireGuard
sudo apt update
sudo apt install wireguard -y

graphql
Copy code

### 1.2 Create WireGuard directory and keys
sudo mkdir -p /etc/wireguard
cd /etc/wireguard
umask 077
wg genkey | tee server_private.key | wg pubkey > server_public.key

bash
Copy code

### 1.3 Create server configuration (wg0.conf)
File: `/etc/wireguard/wg0.conf`

[Interface]
Address = 10.10.0.1/24
ListenPort = 51820
PrivateKey = (server private key)

PostUp = sysctl -w net.ipv4.ip_forward=1; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE; sysctl -w net.ipv4.ip_forward=0

SaveConfig = true

shell
Copy code

### 1.4 Enable IP forwarding permanently
sudo sysctl -w net.ipv4.ip_forward=1

shell
Copy code

### 1.5 Bring WireGuard online
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0

yaml
Copy code

---

## 2. Client Setup (Windows)

### 2.1 Install WireGuard client (official GUI)
Download from: https://www.wireguard.com/install/

### 2.2 Create the client configuration

[Interface]
PrivateKey = (client private key)
Address = 10.10.0.2/24
[Peer]
PublicKey = (server public key)
PersistentKeepalive = 25

yaml
Copy code


---


### 3.1 NAT masquerading

shell
Copy code
### 3.2 Forward rules
sudo iptables -A FORWARD -i wg0 -o eth0 -j ACCEPT

yaml
Copy code


## 4. VirtualBox Network Configuration

### Server (Kali VM)
- Adapter 1 → **Bridged Adapter**
- Promiscuous mode: Allow All

Kali receives LAN IP like `192.168.x.x`.

- Communicates directly with Kali via LAN.

---

## 5. Testing the VPN

### From Windows:
ping 10.10.0.1

shell
Copy code

### Check if traffic is routed through Kali
tracert 1.1.1.1

yaml
Copy code

Output shows:
1 10.10.0.1
2 ISP Router
...

yaml
Copy code

This confirms the VPN tunnel is active.

---

## 6. Repository Structure

Wireguard-VPN/
│
├── server/
│ └── wg0.conf
│
├── client/
│ ├── client1.conf
│ └── README.md
│
└── docs/
└── documentation.md

yaml
Copy code

---

## 7. Future Enhancements
Planned upgrades:
- Add authentication logging
- Add per-client firewall restrictions
- Implement QR-code provisioning
- Setup a dashboard (Flask or Node.js)
- Build a script to auto-generate all keys & configs
- Add multi-client support

---

## Conclusion
This project provides a complete, working implementation of a WireGuard VPN server and client from scratch.  
It demonstrates command-line configuration, routing, NAT, networking via VirtualBox, and secure connection setup.
### Client (Windows host)
- Adapter connected to → Your home WiFi NIC (Intel(R) Wireless-AC)
This is crucial to make Windows ↔ Kali communication possible.
---
sudo iptables -A FORWARD -i eth0 -o wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT

sudo iptables -t nat -A POSTROUTING -s 10.10.0.0/24 -o eth0 -j MASQUERADE
## 3. Routing and Firewall Rules (Kali)
### 2.3 Import into WireGuard GUI → Activate tunnel
AllowedIPs = 0.0.0.0/0, ::/0
Endpoint = (server LAN IP):51820
DNS = 1.1.1.1

