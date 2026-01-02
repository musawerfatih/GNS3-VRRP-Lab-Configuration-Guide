# GNS3 Lab: High Availability with VRRP, Object Tracking, and NAT - Topology Screenshot Available At The End

This lab demonstrates a redundant gateway architecture using **Virtual Router Redundancy Protocol (VRRP)**. It features stateful failover logic where the primary router monitors its uplink (WAN) status and yields control to the backup router if the internet connection is lost.

## 🌐 Topology Architecture

The network is designed with a dual-homed gateway approach:

* **Internal LAN:** `192.168.10.0/24` with a Virtual IP (VIP) of `.254`.
* **External WAN:** `192.168.134.0/24` (in my case) connecting to a simulated Internet/NAT cloud.
* **Redundancy:** R1 is the **Master** (Priority 120), and R2 is the **Backup** (Priority 100).
* **Router:** R1 & R2 are Cisco Images of C7200 Series "`c7200-advipservicesk9-mz.152-4.S5.image`".

## 🚀 Key Features

### 1. VRRP Priority & Preemption

R1 is configured with a higher priority to ensure it is the preferred path. **Preemption** is enabled by default so that if R1 reboots, it will reclaim the Master role once it is back online.

### 2. Interface Tracking (The "Smart" Failover)

Unlike basic VRRP which only fails if the router dies, this lab uses **Object Tracking**:

* R1 monitors its WAN interface (`g1/0`).
* If `g1/0` goes down, R1 automatically decrements its priority by **30**.
* This drops R1's priority to **90** (lower than R2’s 100), triggering an immediate failover so traffic doesn't hit a "black hole."

### 3. Redundant PAT (Port Address Translation)

Both routers are configured with NAT Overload. Since the PC points to the VIP (`.254`), only the router currently acting as the VRRP Master will process and translate the traffic.

## 🛠️ Step Wise Configuration Details

### Router R1 (Master) - Configuration
#### Interface IPs Configuration on R1
```bash
interface g2/0
 ip address 192.168.10.1 255.255.255.0
 no shut

interface g1/0
 ip address 192.168.134.5 255.255.255.0
 no shut
```
#### VRRP Configuration on R1
```bash
interface g2/0
 vrrp 10 ip 192.168.10.254
 vrrp 10 priority 120
```
#### NAT Configuration on R1
```bash
access-list 1 permit any
interface g2/0
 ip nat inside

interface g1/0
 ip nat inside

ip nat inside source list 1 interface g1/0 overload
```
#### Configure Default Route on R1
```bash
ip route 0.0.0.0 0.0.0.0 192.168.134.2  (In my case - My NAT Interface IP)
```

### Router R2 (Backup) - Configuration
#### Interface IPs Configuration on R2
```bash
interface g2/0
 ip address 192.168.10.2 255.255.255.0
 no shut

interface g1/0
 ip address 192.168.134.6 255.255.255.0
 no shut
```
#### VRRP Configuration on R2
```bash
interface g2/0
 vrrp 10 ip 192.168.10.254
```
#### NAT Configuration on R2
```bash
access-list 1 permit any
interface g2/0
 ip nat inside

interface g1/0
 ip nat inside

ip nat inside source list 1 interface g1/0 overload
```
#### Configure Default Route on R2
```bash
ip route 0.0.0.0 0.0.0.0 192.168.134.2  (In my case - My NAT Interface IP)
```

---

## Configuring IP-SLA on WAN Interfaces of R1 & R2
### Step# 1: Configuring IP-SLA on R1
```bash
ip sla 1
 icmp-echo 192.168.134.2 source-interface g1/0
  frequency 5
ip sla schedule 1 life forever start-time now
```

### Step# 2: Configure Object Tracking on R1
```bash
track 1 ip sla 1 reachability
```

### Step# 3: Apply Tracking to VRRP on R1
```bash
interface g2/0
 vrrp 10 track 1 decrement 30
```

### Step# 1: Configuring IP-SLA on R2
```bash
ip sla 1
 icmp-echo 192.168.134.2 source-interface g1/0
  frequency 5
ip sla schedule 1 life forever start-time now
```

### Step# 2: Configure Object Tracking on R2
```bash
track 1 ip sla 1 reachability
```

### Step# 3: Apply Tracking to VRRP on R1
```bash
interface g2/0
 vrrp 10 track 1 decrement 30
```
---

## 🧪 Verification & Testing

### 1. Check VRRP Status

Run this on both routers to see who is Master and who is Backup:
```bash
show vrrp brief
```

### 2. Verify NAT Translations

Check which router is currently translating traffic:
```bash
show ip nat translations
```
---

## How to Use
1. Download the `.gns3` project file and the associated node configurations.
2. Import the project into GNS3.
3. Ensure you have the required IOU and Cisco IOS images.
4. Start the nodes and test connectivity by pinging 8.8.8.8 from PC-1.
5. Shut down R1's interface to observe R2 taking over the Master role.

## Topology Screenshot
<img width="799" height="597" alt="image" src="https://github.com/user-attachments/assets/eaa48a30-2980-40e9-8f00-221f62cde6d5" />

