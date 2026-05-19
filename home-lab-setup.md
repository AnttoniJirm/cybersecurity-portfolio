# Cybersecurity Home Lab Setup

**Platform:** VirtualBox 7.2  
**Host OS:** Windows 11  
**Date:** May 2026  

---

## Objective

Build an isolated virtual lab environment to safely practice cybersecurity attacks and defenses without risk to the host machine or home network.

---

## Virtual Machines

| VM | OS | Role | IP Address |
|----|----|------|------------|
| Kali Linux | Kali 2026.1 | Attacker | 192.168.20.11 |
| Ubuntu | Ubuntu 26.04 | Victim/Defender | 192.168.20.10 |

---

## Network Configuration

### Why not NAT?

By default, VirtualBox assigns **NAT** to every VM. NAT gives the VM internet access through the host machine — useful for downloading tools, but dangerous when practicing attacks or running malware, as there is a risk of affecting the host network.

### Why Internal Network?

**Internal Network** isolates the VMs in their own private network. They can communicate with each other but:
- Cannot access the internet
- Cannot access the host machine
- Cannot reach the home LAN

This makes it the safest option for attack/defense practice.

### Network Diagram 

┌─────────────────────────────────────┐
│         Internal Network            │
│              "labnet"               │
│                                     │
│  ┌──────────────┐  ┌─────────────┐  │
│  │  Kali Linux  │  │   Ubuntu    │  │
│  │ 192.168.20.11│◄─►│192.168.20.10│  │
│  │  (Attacker)  │  │  (Victim)   │  │
│  └──────────────┘  └─────────────┘  │
│                                     │
│      ✗ No internet access           │
│      ✗ No access to host machine    │
└─────────────────────────────────────┘

---

## Static IP Configuration

### Kali Linux
Edited `/etc/network/interfaces`: 

auto eth0
iface eth0 inet static
address 192.168.20.11
netmask 255.255.255.0 

Applied with: sudo systemctl restart networking 

### Ubuntu
Configured via **Settings → Network → IPv4 → Manual**:
- Address: 192.168.20.10
- Netmask: 255.255.255.0
- Gateway: (empty)

---

## Connectivity Test

Verified communication between VMs using ping from Kali to Ubuntu: 
ping 192.168.20.10 
Result: 
64 bytes from 192.168.20.10: icmp_seq=1 ttl=64 time=2.21 ms
64 bytes from 192.168.20.10: icmp_seq=2 ttl=64 time=1.96 ms
64 bytes from 192.168.20.10: icmp_seq=3 ttl=64 time=1.61 ms 

✅ Both machines are communicating successfully on the isolated internal network.

---

## Key Takeaways

- VirtualBox network modes serve different purposes — choosing the right one is a security decision, not just a technical one
- Static IPs are essential when machines need to reliably find each other on a network
- Isolating lab environments from production networks is a fundamental security practice used in enterprise SOC environments
