# Packet Tracer - Configure EtherChannel

## 📌 Project Overview
This repository contains the completed configuration guide and documentation for the **Configure EtherChannel** Packet Tracer activity. The lab demonstrates how to bundle redundant physical switch links into single logical links using both **Cisco PAgP** and **IEEE 802.3ad LACP**, as well as optimizing **Spanning Tree Protocol (STP)** root bridge placement to maximize link bandwidth.

---

## 📋 Port Channel Table

| Channel Group | Devices & Ports | Protocol | Negotiation Mode |
| :--- | :--- | :--- | :--- |
| **Port-Channel 1** | **S1:** `F0/21`, `F0/22`<br>**S3:** `F0/21`, `F0/22` | **PAgP** | S1: `desirable`<br>S3: `desirable` |
| **Port-Channel 2** | **S1:** `G0/1`, `G0/2`<br>**S2:** `G0/1`, `G0/2` | **LACP** | S1: `active`<br>S2: `active` |
| **Port-Channel 3** | **S2:** `F0/23`, `F0/24`<br>**S3:** `F0/23`, `F0/24` | **LACP** | S2: `passive`<br>S3: `active` |

---

## ⚙️ Device Configurations

### Part 1: Configure Basic Switch Settings
Assign hostnames and configure all participating EtherChannel interfaces as static trunk ports on **S1**, **S2**, and **S3** before bundling.

#### Switch S1
```ios
Switch> enable
Switch# configure terminal
Switch(config)# hostname S1

S1(config)# interface range FastEthernet 0/21 - 22, GigabitEthernet 0/1 - 2
S1(config-if-range)# switchport mode trunk
S1(config-if-range)# exit
```

#### Switch S2
```ios
Switch> enable
Switch# configure terminal
Switch(config)# hostname S2

S2(config)# interface range GigabitEthernet 0/1 - 2, FastEthernet 0/23 - 24
S2(config-if-range)# switchport mode trunk
S2(config-if-range)# exit
```

#### Switch S3
```ios
Switch> enable
Switch# configure terminal
Switch(config)# hostname S3

S3(config)# interface range FastEthernet 0/21 - 24
S3(config-if-range)# switchport mode trunk
S3(config-if-range)# exit
```

---

### Part 2: Configure an EtherChannel with Cisco PAgP (Port-Channel 1)
> ⚠️ **Best Practice:** Shut down physical ports before adding them to a channel-group to prevent EtherChannel Misconfig Guard from placing interfaces into an `err-disabled` state.

#### Switch S1
```ios
S1(config)# interface range FastEthernet 0/21 - 22
S1(config-if-range)# shutdown
S1(config-if-range)# channel-group 1 mode desirable
S1(config-if-range)# no shutdown
S1(config-if-range)# exit

S1(config)# interface port-channel 1
S1(config-if)# switchport mode trunk
S1(config-if)# exit
```

#### Switch S3
```ios
S3(config)# interface range FastEthernet 0/21 - 22
S3(config-if-range)# shutdown
S3(config-if-range)# channel-group 1 mode desirable
S3(config-if-range)# no shutdown
S3(config-if-range)# exit

S3(config)# interface port-channel 1
S3(config-if)# switchport mode trunk
S3(config-if)# exit
```

---

### Part 3: Configure an 802.3ad LACP EtherChannel (Port-Channel 2)

#### Switch S1
```ios
S1(config)# interface range GigabitEthernet 0/1 - 2
S1(config-if-range)# shutdown
S1(config-if-range)# channel-group 2 mode active
S1(config-if-range)# no shutdown
S1(config-if-range)# exit

S1(config)# interface port-channel 2
S1(config-if)# switchport mode trunk
S1(config-if)# exit
```

#### Switch S2
```ios
S2(config)# interface range GigabitEthernet 0/1 - 2
S2(config-if-range)# shutdown
S2(config-if-range)# channel-group 2 mode active
S2(config-if-range)# no shutdown
S2(config-if-range)# exit

S2(config)# interface port-channel 2
S2(config-if)# switchport mode trunk
S2(config-if)# exit
```

---

### Part 4: Configure a Redundant EtherChannel Link (Port-Channel 3) & Optimize STP

#### Switch S2 (Passive LACP)
```ios
S2(config)# interface range FastEthernet 0/23 - 24
S2(config-if-range)# shutdown
S2(config-if-range)# channel-group 3 mode passive
S2(config-if-range)# no shutdown
S2(config-if-range)# exit

S2(config)# interface port-channel 3
S2(config-if)# switchport mode trunk
S2(config-if)# exit
```

#### Switch S3 (Active LACP)
```ios
S3(config)# interface range FastEthernet 0/23 - 24
S3(config-if-range)# shutdown
S3(config-if-range)# channel-group 3 mode active
S3(config-if-range)# no shutdown
S3(config-if-range)# exit

S3(config)# interface port-channel 3
S3(config-if)# switchport mode trunk
S3(config-if)# exit
```

#### Set S1 as STP Primary Root for VLAN 1
By default, Spanning Tree Protocol may block the higher-bandwidth Gigabit EtherChannel (`Po2`) to prevent switching loops. Setting **S1** as the root bridge ensures `Po1` and `Po2` remain in the forwarding (`FWD`) state.

```ios
S1(config)# spanning-tree vlan 1 root primary
```
*(Alternatively: `S1(config)# spanning-tree vlan 1 priority 24576`)*

---

## 🔍 Verification Commands

Run the following commands on each switch to verify EtherChannel aggregation, trunking status, and STP forwarding states:

1. **Verify EtherChannel Bundle Status:**
   ```ios
   show etherchannel summary
   ```
   *Expected flags:* `Po1(SU)`, `Po2(SU)`, `Po3(SU)` with member ports showing `(P)` (in port-channel).

2. **Verify Logical Trunk Interfaces:**
   ```ios
   show interfaces trunk
   ```
   *Expected output:* `Po1`, `Po2`, and `Po3` listed in `trunking` state instead of individual physical interfaces.

3. **Verify Spanning Tree Forwarding States:**
   ```ios
   show spanning-tree active
   ```
   *Expected output on S1:* Both `Po1` and `Po2` show Role `Desg` and Status `FWD`.