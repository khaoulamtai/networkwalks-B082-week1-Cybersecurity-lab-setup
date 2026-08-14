# Cybersecurity Lab Environment Setup

**Building a virtual cybersecurity laboratory using VirtualBox and Kali Linux ARM64**

## 📌 Project Overview

This project focuses on setting up a **virtual cybersecurity and penetration-testing laboratory** using VirtualBox and Kali Linux.

The purpose of the lab is to create a controlled environment where cybersecurity tools, network scanning, reconnaissance, vulnerability assessment, and other security-testing activities can be practiced safely.

The laboratory uses a private **NAT Network** so that additional target machines can be added in future projects.

---

## 🎯 Objectives

The main objectives of this project are to:

* Install and configure VirtualBox.
* Install the correct Kali Linux version for an Apple Silicon Mac.
* Create a private NAT Network.
* Configure Kali Linux networking.
* Assign a static IPv4 address to Kali.
* Verify network connectivity and DNS resolution.
* Create a clean VM snapshot.
* Document troubleshooting and configuration steps.
* Prepare the environment for future cybersecurity projects.

---

## 🛡️ Purpose of the Lab

The lab is designed for cybersecurity learning and authorized security testing.

Future exercises may include:

* Network reconnaissance
* Port scanning
* Vulnerability assessment
* Packet analysis
* Web security testing
* Exploitation practice
* Security-tool experimentation

> ⚠️ **Important:** All testing must be performed only against systems that you own or have explicit permission to test.

---

## 🧾 Prerequisites

* A Mac with Apple Silicon (M1/M2/M3) — this build was tested on an **Apple M2, 16 GB RAM**
* [VirtualBox](https://www.virtualbox.org/) (ARM64/Apple Silicon build)
* [Kali Linux ARM64 installer ISO](https://www.kali.org/get-kali/) — this build used `kali-linux-2026.2-installer-arm64.iso`
* At least ~20 GB of free disk space for the VM
* Basic familiarity with the Linux command line and networking concepts (IP addressing, gateways, DNS)

---

# 🏗️ Lab Architecture

<img width="2940" height="1912" alt="image" src="https://github.com/user-attachments/assets/fbc74cba-2e3e-403a-9270-f0c0d9d5f9c6" />

---

# ⚙️ Lab Configuration

| Component          | Configuration                           |
| ------------------ | ---------------------------------------- |
| 🖥️ Host OS        | macOS                                   |
| ⚡ CPU              | Apple M2                                |
| 🧠 Host RAM        | 16 GB                                   |
| 🧰 Hypervisor      | VirtualBox                              |
| 🐉 Security OS     | Kali Linux 2026.2 ARM64                 |
| 💿 Kali Installer  | `kali-linux-2026.2-installer-arm64.iso` |
| 🧠 Kali RAM        | 4096 MB *(increased from the initial 2048 MB for smoother performance with tools like Burp Suite and Wireshark)* |
| 🌐 Virtual Network | NAT Network                             |
| 📡 Network Name    | NatNetwork                              |
| 📡 Network Address | 10.0.0.0/24                             |
| 🐧 Kali Interface  | eth0                                    |
| 🐧 Kali IP         | 10.0.0.2/24                             |
| 🚪 Gateway         | 10.0.0.1                                |
| 🌍 DNS             | 8.8.8.8                                 |
| 🔮 Future VM Range | 10.0.0.3–10.0.0.99                      |

---

# 🪜 Lab Setup Procedure

## Step 1. Install VirtualBox

VirtualBox was installed on the MacBook to provide the virtualization environment for Kali Linux.

Because the host system uses an **Apple M2 processor**, an ARM64-compatible Kali Linux installation image was required.

---

## Step 2. Select the Correct Kali Linux Image

One of the first challenges was selecting the correct Kali Linux image.

The host computer uses an **Apple Silicon M2 processor**, which uses the ARM64 architecture.

The Kali Linux installer used for this project was:

```text
kali-linux-2026.2-installer-arm64.iso
```

This was selected because it matches the ARM64 architecture of the Apple M2 system.

### Lesson Learned

The operating-system architecture is important when creating virtual machines.

For example:

```text
Intel/AMD computers → x86_64 / amd64
Apple Silicon Macs → ARM64 / arm64
```

Using an incompatible image can cause installation or boot problems.

---

## Step 3. Create the NAT Network

A dedicated NAT Network was created in VirtualBox.

Configuration:

```text
Network Name: NatNetwork
IPv4 Prefix:  10.0.0.0/24
DHCP:         Enabled
IPv6:         Disabled
```
<img width="2940" height="1912" alt="image" src="https://github.com/user-attachments/assets/b348a438-6bb2-417a-8ed1-61793d47981d" />

A NAT Network was chosen because multiple virtual machines connected to the same network can communicate with each other while also having outbound connectivity.

This will allow vulnerable target machines to be added later.

---

## Step 4. Install Kali Linux

The Kali Linux ARM64 installer ISO was attached to the virtual machine:

```text
kali-linux-2026.2-installer-arm64.iso
```

The Kali VM was allocated:

```text
RAM: 4096 MB
```

The virtual network adapter was configured as:

```text
Adapter 1
Attached to: NAT Network
Network:     NatNetwork
```

---

## Step 5. Configure the Kali Network

The Kali network connection was configured using NetworkManager.

The connection profile used was:

```text
Wired connection 1
```

The IPv4 settings were configured as:

```text
IPv4 Method: Manual
Address:     10.0.0.2/24
Gateway:     10.0.0.1
DNS:         8.8.8.8
```

The `/24` prefix corresponds to:

```text
255.255.255.0
```

The static IP makes it easier to reference Kali consistently in future laboratory exercises.

<img width="2940" height="1912" alt="image" src="https://github.com/user-attachments/assets/3ac20219-d5c8-439b-931b-cf5bb14b3473" />

---

# 🔎 Lab Verification

The following commands were used to verify the configuration.

### Check the IP address

```bash
ip addr show eth0
```

Expected:

```text
inet 10.0.0.2/24
```

### Check the network connection

```bash
nmcli device status
```

Expected:

```text
eth0    ethernet    connected    Wired connection 1
```

### Check the routing table

```bash
ip route
```

Expected to contain:

```text
default via 10.0.0.1
```

### Test the gateway

```bash
ping -c 4 10.0.0.1
```

### Test Internet connectivity

```bash
ping -c 4 8.8.8.8
```

### Test DNS

```bash
nslookup networkwalks.com
```

### Verify Nmap

```bash
nmap --version
```

---

# 🐞 Problems Encountered & Solutions

## Problem 1. Kali `eth0` Did Not Have an IPv4 Address

After manually configuring the network, `eth0` appeared to be running, but the expected IPv4 address was missing.

The interface showed an IPv6 link-local address:

```text
inet6 fe80::.../64
```

but not:

```text
inet 10.0.0.2/24
```

### Investigation

NetworkManager was checked:

```bash
nmcli connection show
```

and:

```bash
nmcli device status
```

The `Wired connection 1` profile existed, but it was not connected to `eth0`.

The connection profile contained the expected settings:

```text
IPv4 Method:     Manual
IPv4 Address:    10.0.0.2/24
IPv4 Gateway:    10.0.0.1
IPv4 DNS:        8.8.8.8
```

However, activating the connection produced:

```text
Connection activation failed:
IP configuration could not be reserved
```

### Solution

The problem was resolved by setting the IPv4 DAD timeout to `0`:

```bash
sudo nmcli connection modify "Wired connection 1" ipv4.dad-timeout 0
```

After applying this configuration, the connection activated successfully and the expected IPv4 address became available.

### Lesson Learned

A network interface being `UP` does not necessarily mean that its IPv4 configuration has been successfully applied.

I also learned the difference between:

```text
eth0
```

which is the network interface, and:

```text
Wired connection 1
```

which is the NetworkManager connection profile used to configure the interface.

---

# 💡 What I Learned

### 1. ARM64 Virtualization

I learned that Apple Silicon Macs use the ARM64 architecture and that the operating-system image must be compatible with that architecture.

### 2. VirtualBox Networking

I learned how virtual network adapters connect Kali Linux to VirtualBox networks.

### 3. NAT Network

I learned why a NAT Network is useful for a cybersecurity lab: multiple VMs can communicate with each other while maintaining external connectivity.

### 4. IPv4 Configuration

I practiced configuring:

```text
IP address
Subnet prefix
Gateway
DNS
```

and verifying the resulting configuration from the Linux command line.

### 5. NetworkManager

I learned how NetworkManager manages network interfaces through connection profiles such as:

```text
Wired connection 1
```

### 6. Troubleshooting

The project helped me practice troubleshooting a real network configuration problem instead of simply following setup instructions.

### 7. Snapshots

A clean VM snapshot provides a known-good baseline that can be restored before future cybersecurity experiments.

---

# 🔐 Security & Ethical Use

This laboratory is intended strictly for **education and authorized security testing**.

All scanning, vulnerability assessment, exploitation, and penetration-testing activities will be performed only against systems that are owned by me, intentionally vulnerable laboratory machines, or systems for which explicit authorization has been provided.

---

# 👤 Author

**Khaoila Benlamtai**

Computer Science Graduate
Cyberspace Master's Student

---

## 📌 Project Information

**Project:** Cybersecurity & Penetration-Testing Lab Setup
**Platform:** Apple Silicon Mac (M2)
**Virtualization:** VirtualBox
**Security OS:** Kali Linux 2026.2 ARM64
**Network:** 10.0.0.0/24 NAT Network
**Kali IP:** 10.0.0.2/24
**Status:** Initial laboratory environment configured successfully
