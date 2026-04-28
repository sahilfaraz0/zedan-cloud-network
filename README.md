# Networking in the Cloud (ZEDAN Enterprises)

**Author:** Sahil Faraz | **Date:** June 2025

---

## 📋 Project Overview

This repository contains the Cisco Packet Tracer network simulations developed for a **Networking in the Cloud** assignment which included *Principles, Implementation, and Performance Enhancement*. It demonstrates the design, implementation, testing, and optimization of a multi-site enterprise cloud network. The network spans three physical zones interconnected over a Frame Relay WAN, secured with Cisco ASA firewalls, and validated through performance and scalability testing. Two iterations are included — a base implementation and an optimized version with NAT, redundancy, and IoT integration.

> Simulated in **Cisco Packet Tracer 8.x** | Part of HND Digital Technologies (Cybersecurity) — Unit 6: Networking in the Cloud

---

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Network Zones](#network-zones)
- [IP Addressing](#ip-addressing)
- [WAN Design — Frame Relay](#wan-design--frame-relay)
- [Routing](#routing)
- [Security — ASA Firewalls](#security--asa-firewalls)
- [Base vs Optimized](#base-vs-optimized)
- [Performance Testing](#performance-testing)
- [Repository Structure](#repository-structure)
- [How to Use](#how-to-use)
- [Tools](#tools)

---

## Architecture Overview

The network follows a **three-zone segmentation model** representing a warehouse hosting core services, an organizational headquarters, and a remote state office. All three zones are connected over a simulated Frame Relay WAN cloud. Each zone has a dedicated router, Layer 2 switch, and (in the optimized build) a Cisco ASA 5505 firewall.


---

## Network Zones

### Warehouse Zone
Hosts the organization's core cloud-facing servers. All server traffic is routed through `WH-RTR` and protected by `WH-ASA` in the optimized build.

| Device | Role |
|--------|------|
| WH-RTR | Zone gateway, Frame Relay hub, static routing |
| WH-SW | Layer 2 access switch for servers and PC1 |
| WH-ASA | Perimeter firewall with NAT *(optimized only)* |
| WEB | Web server — 192.168.100.10 |
| FTP | FTP server — 192.168.100.20 |
| INTERNET | Simulated internet server — 192.168.100.30 |

### Organizational Zone
Represents ZEDAN's headquarters network. Employee PCs and laptops connect through `SW-ORG`. Traffic to the WAN passes through `ASA-ORG` before reaching `R-ORG`.

| Device | Role |
|--------|------|
| R-ORG | Zone router, Frame Relay spoke |
| ASA-ORG (FW-ORG) | Stateful firewall, DHCP for inside clients |
| SW-ORG | Layer 2 access switch |
| ORG-PCs / ORG-LAPs | Employee endpoints — DHCP assigned |

### State Office Zone
Remote office connectivity. PCs receive addresses via DHCP from `STO-RTR`. The optimized build adds `STO-ASA` with dynamic NAT and an IoT server.

| Device | Role |
|--------|------|
| STO-RTR (CL-RTR) | Zone gateway, DHCP server, Frame Relay spoke |
| STO-SW | Layer 2 access switch |
| STO-ASA (CLT-ASA) | Edge firewall with dynamic NAT *(optimized only)* |
| STO-PCs / STO-LAPs | Client endpoints — DHCP assigned |
| IOT | IoT simulation server *(optimized only)* |

---

## IP Addressing

### Warehouse Zone — 192.168.100.0/24

| Device | Interface | IP Address |
|--------|-----------|------------|
| WH-RTR | GigabitEthernet0/0 | 192.168.100.1 |
| WH-RTR | GigabitEthernet0/1 *(optimized)* | 192.168.200.1 |
| WEB | FastEthernet0 | 192.168.100.10 |
| FTP | FastEthernet0 | 192.168.100.20 |
| INTERNET | FastEthernet0 | 192.168.100.30 |
| PC1 | FastEthernet0 | 192.168.100.15 *(static)* |

### Organizational Zone — 192.168.10.0/24 (inside) / 192.168.20.0/24 (outside)

| Device | Interface | IP Address |
|--------|-----------|------------|
| R-ORG | GigabitEthernet0/0 | 192.168.20.2 |
| ASA-ORG | Vlan1 (inside) | 192.168.10.1 |
| ASA-ORG | Vlan2 (outside) | 192.168.20.1 |
| ORG-PCs | — | DHCP → 192.168.10.10–10.31 |

### State Office Zone — 192.168.50.0/24

| Device | Interface | IP Address |
|--------|-----------|------------|
| STO-RTR | GigabitEthernet0/0 | 192.168.50.1 |
| STO-ASA | Vlan1 (inside) *(optimized)* | 192.168.50.2 |
| STO-ASA | Vlan2 (outside) *(optimized)* | 192.0.2.2 |
| STO-RTR | GigabitEthernet0/1 *(optimized)* | 192.0.2.1 |
| STO-PCs | — | DHCP → 192.168.50.x |

### WH-ASA — NAT Boundary *(optimized only)*

| Interface | IP Address |
|-----------|------------|
| Vlan1 (inside) | 192.168.100.2 |
| Vlan2 (outside) | 192.168.200.2 |

### PPP Link *(optimized only)*

| Device | Interface | IP Address |
|--------|-----------|------------|
| WH-RTR | Serial0/0/1 | 10.0.10.1 |
| — | — | 10.0.10.2 (peer) |

---

## WAN Design — Frame Relay

The three zones are interconnected using **Frame Relay point-to-point subinterfaces** over a simulated cloud. `WH-RTR` acts as the hub.

| Router | Subinterface | Local IP | DLCI | Peer |
|--------|-------------|----------|------|------|
| WH-RTR | Serial0/0/0.1 | 10.0.0.1/30 | 201 | R-ORG |
| WH-RTR | Serial0/0/0.2 | 10.0.0.5/30 | 202 | STO-RTR |
| R-ORG | Serial0/0/0.1 | 10.0.0.2/30 | 301 | WH-RTR |
| STO-RTR | Serial0/0/0.1 | 10.0.0.6/30 | 302 | WH-RTR |

All serial interfaces use `encapsulation frame-relay` with explicit `frame-relay interface-dlci` assignments on each point-to-point subinterface.

---

## Routing

All inter-zone routing uses **static routes**. No dynamic routing protocol is configured, keeping the design deterministic and easy to audit.

### WH-RTR

