# Inter-VLAN Routing Configuration

A comprehensive guide for configuring inter-VLAN routing between Cisco switches and routers.

## Table of Contents
- [Switch Configuration](#switch-configuration)
- [Router Configuration](#router-configuration)
- [Useful Commands](#useful-commands)

---
## Topology Example
![Network Topology](topology.png)

## Switch Configuration

### Creating VLANs

```bash
enable
configure terminal
vlan {number}
name {name}
exit
```

**Example:**
```bash
vlan 10
name Sales
exit
```

### Setting up Access Ports

Access ports connect end devices (computers, printers, etc.) to a specific VLAN.

```bash
interface {interface}
switchport mode access
switchport access vlan {vlan_number}
exit
```

**Example:**
```bash
interface FastEthernet0/1
switchport mode access
switchport access vlan 10
exit
```

### Setting up Trunk Ports

Trunk ports carry traffic for multiple VLANs between switches or between a switch and router.

```bash
interface {interface}
switchport mode trunk
switchport trunk allowed vlan {num1,num2,...}
exit
```

**Example:**
```bash
interface GigabitEthernet0/1
switchport mode trunk
switchport trunk allowed vlan 10,20,30
exit
```

---

## Router Configuration

Configure router subinterfaces for inter-VLAN routing (Router-on-a-Stick).

```bash
enable
configure terminal
interface {interface}.{vlan_number}
encapsulation dot1q {vlan_number}
ip address {gateway_ip} {subnet_mask}
exit
```

**Example:**
```bash
interface GigabitEthernet0/0.10
encapsulation dot1q 10
ip address 192.168.10.1 255.255.255.0
exit

interface GigabitEthernet0/0.20
encapsulation dot1q 20
ip address 192.168.20.1 255.255.255.0
exit
```

**Note:** The IP address should be the default gateway for devices in that VLAN.

Don't forget to enable the physical interface:
```bash
interface GigabitEthernet0/0
no shutdown
exit
```

---

## Useful Commands

### Switch Commands

| Command | Description |
|---------|-------------|
| `show vlan brief` | Display VLAN configuration summary |
| `show interface status` | View interface status and VLAN assignments |
| `show interfaces trunk` | Display trunk port information |
| `no shutdown` | Enable an interface |
| `write memory` | Save configuration to NVRAM |
| `copy running-config startup-config` | Alternative save command |

### Router Commands

| Command | Description |
|---------|-------------|
| `show ip interface brief` | Display IP configuration summary |
| `show ip route` | View routing table |
| `show running-config` | Display current configuration |
| `no shutdown` | Enable an interface |
| `write memory` | Save configuration to NVRAM |

---

