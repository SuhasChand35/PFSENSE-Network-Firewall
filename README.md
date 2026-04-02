# PFSENSE-Network-Firewall

## 1. Objectives

- Stand up a pfSense firewall in VirtualBox with separate WAN (bridged) and LAN (internal) segments  
- Put Kali Linux on the WAN side to simulate an external attacker  
- Put Ubuntu Desktop on the LAN side as the victim host  
- Demonstrate DoS traffic with `hping3` and capture it using Wireshark  
- Mitigate the attack using pfSense firewall rules and analyze the resulting logs  

## Topology & IP Plan

| Device / Segment              | VirtualBox Adapter | Interface | IP / Subnet                          | Role                              |
|------------------------------|--------------------|-----------|--------------------------------------|-----------------------------------|
| Home LAN (physical)          | —                  | —         | 192.168.178.0/24                     | Provides Internet & bridges to lab |
| pfSense WAN                  | Bridged            | vtnet0    | e.g., 192.168.178.254/24             | Edge firewall toward home LAN     |
| Kali Linux (Attacker)        | Bridged            | eth0      | e.g., 192.168.178.200/24             | External attacker host            |
| Lab LAN (`LabNet`)           | Internal Network   | —         | 192.168.1.0/24                       | Isolated subnet behind pfSense    |
| pfSense LAN                  | Internal (`LabNet`)| vtnet1   | 192.168.1.1/24 (DHCP: .100–.199)     | Default GW & DHCP/DNS relay       |
| Ubuntu Desktop (Victim)      | Internal (`LabNet`)| eth0     | 192.168.1.100/24                     | Victim workload                   |

##  pfSense VM Setup (VMWare)

###  Virtual Machine Configuration

| Setting        | Value                          |
|----------------|--------------------------------|
| Name           | pfSense                        |
| OS Type        | FreeBSD (64-bit)               |
| CPU / RAM      | 2 vCPU / 2 GB RAM              |
| Disk           | 20 GB VDI (Dynamically Allocated) |

###  Network Configuration

| Adapter   | Mode              | Purpose                     |
|----------|------------------|-----------------------------|
| Adapter 1 | Bridged Adapter  | Connect to physical network (WAN) |
| Adapter 2 | Internal Network (`LabNet`) | Isolated LAN segment |

###  Installation Setup

- Attach the **pfSense ISO**:
  - Go to **Settings → Storage**
  - Add the downloaded pfSense ISO file to the optical drive

- Start the VM to begin installation

##  pfSense Configuration

###  Initial Setup

- Install pfSense (accept default options)
- From the console menu:
  - Select **1) Assign Interfaces**
  - Assign:
    - `vtnet0` → WAN (Bridged)
    - `vtnet1` → LAN (`LabNet`)
- Leave WAN as **DHCP**
  - pfSense will automatically get an IP from the home router
  - Note the WAN IP from console (e.g., `192.168.178.58`)

---

##  5.1 Accessing pfSense from WAN

>  Temporary step for lab/demo purposes only

### Disable pfSense Firewall (temporary)
```bash
pfSense -d
```
## Launching the DoS (hping3)

- Start Wireshark on Ubuntu  
- sudo apt update && sudo apt install -y wireshark  
- sudo wireshark & # capture on eth0  

- Start the flood from Kali  
- sudo hping3 -1 --flood 192.168.1.100 # ICMP flood  
- or SYN flood: sudo hping3 --flood -S -p 80 192.168.1.100  

- View packets spike in Wireshark

## Blocking the Attack & Verifying

- Now we will navigate to the pfSense WebGUI and add a new security rule to block traffic from Kali to the Ubuntu machine  

- pfSense Firewall ▸ Rules ▸ WAN ▸ Add (TOP):  
  - Action: Block  
  - Source: Kali-IP  
  - Destination: 192.168.1.100/24 (Ubuntu IP)  
  - Log: ✓  
  - Description: "Block Kali DoS"  

- Apply – traffic halts, check Wireshark on the Ubuntu machine and you shouldn't see the flood anymore  

- On pfSense, go to the firewall rule and check the logs where logging is enabled to verify that malicious flood traffic is being blocked  
