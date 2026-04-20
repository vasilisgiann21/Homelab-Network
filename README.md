# Enterprise-Grade HomeLab Architecture

![Status](https://img.shields.io/badge/Status-Active-success)
![Platform](https://img.shields.io/badge/Platform-Proxmox%20VE-orange)
![Firewall](https://img.shields.io/badge/Firewall-OPNsense-darkred)
![DNS](https://img.shields.io/badge/DNS-AdGuard%20Home-blue)

## 📌 Overview
This repository documents the architecture, deployment, and configuration of an enterprise-grade, bare-metal homelab environment. The primary objective is to establish a **Zero-Trust Secure Lab** for penetration testing and malware analysis, completely isolated from the primary home network, alongside a network-wide DNS Sinkhole for privacy and tracking prevention.

The infrastructure is built from the ground up prioritizing security, robust Layer 2/Layer 3 routing, and disaster recovery.

## 🏗️ Architecture & Network Topology


### The "Router-on-a-Stick" Model
The core routing is handled by a virtualized OPNsense firewall operating in a "Router-on-a-Stick" configuration.
* **WAN Interface:** Connected directly to the ISP router.
* **LAN / Trunk Interface (`vmbr0`):** Connected to a managed Layer 2/3 switch carrying 802.1Q tagged traffic.
* **VLAN 20 (Secure Lab):** A strictly isolated subnet (`10.10.10.0/24`) with strict firewall rules preventing traffic leaks to the primary LAN, intended for Kali Linux and vulnerable target VMs.

## 🛠️ Tech Stack & Core Components

* **Hypervisor:** Proxmox Virtual Environment (VE) on bare-metal.
* **Core Router / Firewall:** OPNsense (FreeBSD-based).
* **DHCP Services:** Kea DHCP (handling subnet allocations).
* **DNS Sinkhole:** AdGuard Home (deployed in a lightweight Debian LXC container).
* **DNS Forwarder:** Unbound DNS.
* **Networking:** 802.1Q VLANs, Managed Switch (PVID & Tagged/Untagged port configuration).

## 🚀 Key Implementations

### 1. Bare-Metal Virtualization & Resource Allocation
Configured Proxmox VE to host core networking services with minimal overhead. The OPNsense firewall acts as the gateway for all internal virtual networks and physical devices via the managed switch.

### 2. Network-Wide DNS Sinkholing (AdGuard Home)
* Deployed inside an unprivileged Debian LXC container (`10.10.10.5`).
* **Port Conflict Resolution:** Purged pre-installed `apache2` services to release Port 80 and ensure uninterrupted AdGuard Web GUI access.
* **Persistence:** Configured `systemd` to automatically enable and start the `AdGuardHome` daemon upon container boot.
* **Blocklists:** Implemented aggressive tracking and malware blocking using the *OISD Blocklist Big* and *Phishing Army* databases.

### 3. DNS Proxy & Forwarding Architecture
To bypass DHCP client limitations and enforce centralized network monitoring, the OPNsense **Unbound DNS** service is configured as a DNS Forwarder. 
* Clients query the OPNsense gateway (`10.10.10.1`).
* OPNsense silently forwards queries to the AdGuard Sinkhole (`10.10.10.5`) on Port 53.
* Prevents DNS loops and fallback leaks to global resolvers (e.g., Google/Cloudflare) at the endpoint level.

### 4. Chaos Engineering & Disaster Recovery
Engineered the hypervisor boot sequence to survive sudden power losses ("pull-the-plug" resilience):
* **Boot Order 1:** OPNsense Firewall (with a 30-second startup delay to initialize virtual networks).
* **Boot Order 2:** AdGuard LXC Container (waits for the network to establish before binding to the IP).
* Restores total network functionality and ad-blocking without manual intervention.

## 🔮 Future Roadmap (Next Steps)
- [ ] **Penetration Testing Environment:** Deploying Kali Linux within VLAN 20.
- [ ] **Target Acquisition:** Setting up Metasploitable and vulnerable boxes for local exploitation practice.
- [ ] **Intrusion Detection:** Implementing Suricata or Zenarmor on OPNsense for deep packet inspection (DPI).
- [ ] **Observability:** Centralizing logs and metrics using a Prometheus/Grafana stack.

---
*Created and maintained by [Vasilis](https://github.com/vasilisgiann21) - Computer Science Student & Aspiring Security Engineer.*
