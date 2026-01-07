# 🔌 Inter-VLAN Routing Configuration

![Cisco](https://img.shields.io/badge/Platform-Cisco_IOS-blue?style=flat-square&logo=cisco)
![Network](https://img.shields.io/badge/Category-Networking-success?style=flat-square)
![Status](https://img.shields.io/badge/Status-Active-green?style=flat-square)

A comprehensive guide for configuring **Inter-VLAN routing** (specifically Router-on-a-Stick) between Cisco switches and routers. This guide covers the essential commands to trunk VLANs from a Layer 2 switch to a Layer 3 router.


## 🗺 Topology Example

Below is the specific topology used for this configuration guide:

<p align="center">
  <img width="800" alt="topology" src="https://github.com/user-attachments/assets/e9ffe70e-0865-41e4-9ec4-8d3aebff17ad" />
</p>

---

## 🔨 Switch Configuration

### 1. Creating VLANs
Define the Virtual LANs that will exist on the network.

```text
enable
configure terminal
vlan <vlan_number>
 name <vlan_name>
exit
```

**Example:**
```text
vlan 10
 name Sales
exit
```

### 2. Setting up Access Ports
Access ports connect end devices (computers, printers, etc.) to a specific VLAN.


```text
interface <interface_id>
 switchport mode access
 switchport access vlan <vlan_number>
exit
```

**Example:**
```text
interface FastEthernet0/1
 switchport mode access
 switchport access vlan 10
exit
```

### 3. Setting up Trunk Ports
Trunk ports carry traffic for multiple VLANs using 802.1Q tagging. This is usually the uplink to the Router.

```text
interface <interface_id>
 switchport mode trunk
 switchport trunk allowed vlan <vlan_list>
exit
```

**Example:**
```text
interface GigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30
exit
```

---

## 🚀 Router Configuration

Configure router **sub-interfaces** to handle routing between the VLANs defined on the switch.

### Sub-Interface Configuration

```text
enable
configure terminal
interface <interface_id>.<vlan_number>
 encapsulation dot1q <vlan_number>
 ip address <gateway_ip> <subnet_mask>
exit
```

**Example:**

```text
Configure Gateway for VLAN 10
interface GigabitEthernet0/0.10
 encapsulation dot1q 10
 ip address 192.168.10.1 255.255.255.0
exit

Configure Gateway for VLAN 20
interface GigabitEthernet0/0.20
 encapsulation dot1q 20
 ip address 192.168.20.1 255.255.255.0
exit
```

```text
interface GigabitEthernet0/0
 no shutdown
exit
```

---

## 🔎 Useful Commands

### Switch Commands

| Command | Description |
| :--- | :--- |
| `show vlan brief` | Display VLAN database and port assignments. |
| `show interface status` | View physical status, speed, duplex, and VLANs. |
| `show interfaces trunk` | Verify which ports are trunking and which VLANs are allowed. |
| `show running-config` | View the active configuration in RAM. |
| `write memory` | Save configuration to NVRAM. |

### Router Commands

| Command | Description |
| :--- | :--- |
| `show ip interface brief` | Quick overview of IP addresses and interface status (Up/Down). |
| `show ip route` | View the routing table to verify connected subnets. |
| `show ip protocols` | Verify active routing protocols (if any). |
| `copy run start` | Save current configuration to startup. |

---
*Created for network documentation purposes.*
