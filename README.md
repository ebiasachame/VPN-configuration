# VPN-configuration
Site-to-Site VPN Between Cisco Routers (R1 & R3)
This repository documents the configuration of a site-to-site IPsec VPN between two Cisco routers (R1 and R3) using ISAKMP/IKEv1 with pre-shared keys and OSPF for dynamic routing. The initial issue prevented ping connectivity between the LANs (192.168.1.0/24 and 192.168.3.0/24), which was resolved by correcting the ISAKMP peer configuration.

Topology
[ R1 (192.168.1.0/24) ] -- (10.1.1.0/30) --> [ Internet ] <-- (10.2.2.0/30) -- [ R3 (192.168.3.0/24) ]
R1:

LAN: 192.168.1.2/24 (GigabitEthernet0/0)

WAN: 10.1.1.2/30 (Serial0/1/0)

R3:

LAN: 192.168.3.1/24 (GigabitEthernet0/0)

WAN: 10.2.2.2/30 (Serial0/1/0)

Configuration Highlights
1. VPN Setup (IPsec with ISAKMP)
IKE Phase 1 (ISAKMP Policy):

Encryption: AES-256

Authentication: Pre-shared key (vpnpa55)

Diffie-Hellman Group: 5 (1536-bit)

IKE Phase 2 (IPsec Transform Set):

ESP Encryption: AES

ESP Integrity: SHA-HMAC

Crypto Maps:

Applied to Serial interfaces (Serial0/1/0 on both routers)

Traffic matched via ACL 110 (192.168.1.0/24 ↔ 192.168.3.0/24)

2. OSPF Dynamic Routing
OSPF Process ID 101 (Area 0)

Advertises LAN and WAN networks

Passive interfaces set for LAN ports

Issue & Resolution
❌ Problem: Ping Failure Between R1 & R3
Root Cause:

On R3, the ISAKMP key was incorrectly configured with the wrong peer IP (10.1.1.1 instead of 10.1.1.2).

The crypto map had an extra peer entry (10.1.1.1).

✅ Fix Applied
bash
R3(config)# no crypto isakmp key vpnpa55 address 10.1.1.1
R3(config)# crypto isakmp key vpnpa55 address 10.1.1.2
R3(config)# crypto map VPN-MAP 10 ipsec-isakmp
R3(config-crypto-map)# no set peer 10.1.1.1
R3(config-crypto-map)# set peer 10.1.1.2
After correction:
✔ VPN tunnel established successfully
✔ Ping between 192.168.1.0/24 and 192.168.3.0/24 working

Verification Commands
bash
# Check VPN status
show crypto isakmp sa      # Check IKE Phase 1
show crypto ipsec sa       # Check IPsec Phase 2

# Verify OSPF
show ip ospf neighbor      # Check adjacency
show ip route ospf         # Verify routes

# Test connectivity
ping 192.168.3.1 source 192.168.1.2   (From R1)
ping 192.168.1.2 source 192.168.3.1   (From R3)
Lessons Learned
Always double-check peer IPs in VPN configurations.

Use show crypto commands to troubleshoot IPsec tunnels.

Ensure OSPF adjacency is established for dynamic routing.
