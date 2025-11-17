WireGuard Client Configuration (Windows)

This document provides a complete, formal explanation of how the Windows WireGuard client was configured, including the purpose of each parameter, operational behavior, and required verification steps. This section complements the server configuration guide and completes the end-to-end documentation of the VPN implementation.

1. Overview

The client component of this VPN project is responsible for securely initiating a WireGuard tunnel from a Windows host to the WireGuard server running inside a Kali Linux virtual machine.
Communication is established using asymmetric cryptography, a defined tunnel address space, and controlled routing rules.
The client configuration file (client1.conf) is imported into the official WireGuard Windows application.

2. Client Keys

WireGuard requires each peer to have:

A private key (kept secret on the client)

A public key (shared with the server)

2.1 Key Generation (performed on the server)
sudo wg genkey | sudo tee /etc/wireguard/client1_private.key \
    | sudo wg pubkey | sudo tee /etc/wireguard/client1_public.key

2.2 Purpose

The private key authenticates the client to the server.

The public key is stored in the server’s wg0.conf so the server can identify the client.

The private key must never be uploaded to GitHub or shared.

3. Client Configuration File (client1.conf)

Below is the exact configuration used:

[Interface]
PrivateKey = oMvKc+luUPCEO5A9HAl9kn8USV6iKgbf3j8bVZ+fnmo=
Address = 10.10.0.2/24
DNS = 1.1.1.1

[Peer]
PublicKey = pQQSwyO6MT89mOYeWPjflyPqjOnW/Nk9EYJHl4mNti0=
Endpoint = 192.168.1.37:51820
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25


Each parameter is explained in Section 4.

4. Configuration Parameters Explained
4.1 [Interface] Section (Client Local Settings)
PrivateKey

The private key unique to the client.

Used for authentication during the handshake.

Must remain confidential.

Address

The tunnel IP address assigned to the client inside the VPN network.

In this project, the VPN subnet is 10.10.0.0/24.

The client is assigned 10.10.0.2.

DNS

The DNS server the client should use while connected.

1.1.1.1 (Cloudflare) is used for reliability and privacy.

4.2 [Peer] Section (Server Information)
PublicKey

The public key belonging to the server.

The client uses this key to encrypt traffic destined for the server.

Endpoint

The real IP address and UDP port of the server.

In this setup, the VM is bridged, so the server is accessible at:

192.168.1.37:51820

AllowedIPs

Defines what traffic should be routed through the VPN.

0.0.0.0/0, ::/0 means:

All IPv4 traffic

All IPv6 traffic
will be routed through the VPN.

This is known as full-tunnel mode.

PersistentKeepalive

Sends a small packet every 25 seconds.

Prevents NAT devices from dropping the tunnel during inactivity.

Essential when the server or client is behind NAT (which applies here).

5. Importing the Configuration into WireGuard (Windows)
5.1 Steps

Open the WireGuard application.

Select “Add Tunnel” → “Import from File”.

Choose client1.conf.

Confirm the configuration and click “Activate”.

Once active, the WireGuard tunnel interface is created on Windows and assigned the address 10.10.0.2.

6. Verifying the Client Connection
6.1 Verify Handshake

In the WireGuard GUI, the “Latest Handshake” field should show a recent timestamp.

6.2 Verify Tunnel Reachability
ping 10.10.0.1


Expected: replies from the server’s tunnel address.

6.3 Verify that the VPN is routing Internet traffic
Invoke-RestMethod -Uri "https://ifconfig.co/ip"


If configured for full-tunnel routing, the output should show the server’s external IP.

6.4 Traceroute Validation
tracert 1.1.1.1


The first hop must be:

10.10.0.1


which confirms that traffic enters the WireGuard interface before leaving the server.

7. Windows WireGuard Logs (Optional Review)

The WireGuard client provides detailed logs under the Log tab.
These logs can be used to verify:

Successful handshake

Endpoint connection

Routing behavior

Keepalive activity

8. Security Considerations

Do not commit private keys to GitHub under any circumstances.

Use .gitignore to exclude:

*.key
client1.conf


The client’s public key may be safely stored or published, as it cannot compromise security.

When adding multiple clients, each must have unique key pairs and tunnel addresses.

9. Summary

The client configuration enables Windows to securely establish a WireGuard tunnel to the Kali server using asymmetric keys, a defined VPN subnet, and strict routing rules.
All traffic is routed through the tunnel, DNS queries are encrypted, and the persistent keepalive ensures stable connectivity across NAT environments.