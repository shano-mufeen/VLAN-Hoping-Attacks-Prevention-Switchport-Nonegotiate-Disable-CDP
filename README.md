# 🛡️ VLAN Hopping Attack Prevention Using Switchport Security & CDP Hardening

## 📌 Project Overview

This project demonstrates how to protect a Cisco switched network against **VLAN Hopping attacks** using practical Layer 2 security configurations.

A VLAN hopping attack attempts to gain access to traffic belonging to VLANs other than the VLAN assigned to the attacker's device.

The lab focuses on two common VLAN hopping techniques:

* 🔀 **Switch Spoofing**
* 🏷️ **Double-Tagging**

The prevention strategy includes:

* 🔒 Explicitly configuring access ports
* 🚫 Disabling DTP negotiation with `switchport nonegotiate`
* 🌐 Assigning a dedicated native VLAN to trunk links
* 🏷️ Tagging the native VLAN on trunks
* 📡 Disabling CDP where appropriate

---

# 📋 Case Study

A company has multiple departments separated using VLANs.

The network contains:

| VLAN | Department  |
| ---: | ----------- |
|   10 | IT          |
|   20 | HR          |
|   30 | Finance     |
|   33 | Native VLAN |

Devices belonging to different VLANs should remain logically separated unless routing is intentionally configured.

An attacker connected to one VLAN attempts to reach another VLAN without going through an authorized Layer 3 device.

### 🎯 Security Requirement

The network should be configured so that:

1. End-device ports cannot dynamically become trunks.
2. Access ports belong to only their intended VLAN.
3. Trunk links use a dedicated native VLAN.
4. The native VLAN is not used for normal end-user traffic.
5. DTP negotiation is disabled where appropriate.
6. CDP is disabled according to the security requirements of the lab.

---

# 🎯 Project Objectives

* Understand VLAN hopping attacks.
* Understand **switch spoofing**.
* Understand **double-tagging**.
* Configure access ports explicitly.
* Disable DTP negotiation on access ports.
* Configure a dedicated native VLAN.
* Configure trunk links with the native VLAN.
* Prevent unnecessary dynamic trunk formation.
* Disable CDP globally in the lab.
* Verify VLAN and trunk configuration.

---

# 🏢 Network Topology

```text id="1e5v6u"
                         ┌──────────────────┐
                         │     SWITCH 1     │
                         │                  │
                         │ VLAN 10 → IT     │
                         │ VLAN 20 → HR     │
                         │ VLAN 30 → Finance│
                         │ VLAN 33 → Native │
                         └───────┬──────────┘
                                 │
                         802.1Q Trunk
                       Native VLAN 33
                                 │
                         ┌───────▼──────────┐
                         │     SWITCH 2     │
                         │                  │
                         │ VLAN 10 → IT     │
                         │ VLAN 20 → HR     │
                         │ VLAN 30 → Finance│
                         │ VLAN 33 → Native │
                         └───────┬──────────┘
                                 │
                              Trunk
                                 │
                         ┌───────▼──────────┐
                         │     SWITCH 3     │
                         │                  │
                         │ VLAN 10/20/30    │
                         │ VLAN 33 Native    │
                         └──────────────────┘

             Access Ports
                  │
          ┌───────┼────────┐
          │       │        │
        VLAN 10 VLAN 20  VLAN 30
          │       │        │
         IT      HR     Finance
```

---

# 🧠 What Is VLAN Hopping?

**VLAN hopping** is a Layer 2 attack technique in which an attacker attempts to send or receive traffic associated with a VLAN that the attacker's device should not normally access.

Normally:

```text
VLAN 10 ─── 🚫 ─── VLAN 20
```

Different VLANs require Layer 3 routing for normal communication.

A VLAN hopping attack attempts to bypass that intended separation at Layer 2.

---

# ⚠️ VLAN Hopping Attack Techniques

## 1. 🔀 Switch Spoofing

Switch spoofing takes advantage of **Dynamic Trunking Protocol (DTP)** behavior.

If an access-facing interface is allowed to negotiate a trunk, an attacker may attempt to make the interface operate as a trunk.

### Attack Concept

```text id="2v08n7"
Attacker
   │
   │ DTP Negotiation
   ▼
Switch Port
   │
   ▼
Potential Trunk
   │
   ├── VLAN 10
   ├── VLAN 20
   └── VLAN 30
```

Once a port becomes a trunk, multiple VLANs may potentially become reachable through that connection.

### Prevention

Explicitly configure the port as an access port:

```cisco
switchport mode access
```

and disable DTP negotiation:

```cisco
switchport nonegotiate
```

---

# 2. 🏷️ Double-Tagging

Double-tagging involves adding **two VLAN tags** to a frame.

Conceptually:

```text id="4u6qde"
Outer Tag → Native VLAN
Inner Tag → Target VLAN
```

The attacker attempts to exploit how VLAN tags are processed as the frame moves through trunk links.

### Concept

```text id="v7x0zz"
Attacker
VLAN 30
   │
   │ Double-Tagged Frame
   │
   ├── Outer Tag → Native VLAN
   └── Inner Tag → Target VLAN
              │
              ▼
        Target VLAN
```

### Prevention Strategy

A common mitigation is to:

* Use a dedicated native VLAN.
* Do not use the native VLAN for normal end-user traffic.
* Keep the native VLAN consistent across the trunk design.
* Explicitly tag the native VLAN where supported/configured.

---

# 🛡️ VLAN Hopping Prevention Strategy

```text id="y7o5tu"
                VLAN HOPPING PREVENTION
                         │
       ┌─────────────────┼─────────────────┐
       │                 │                 │
       ▼                 ▼                 ▼
 Access Port        Native VLAN       CDP / DTP
 Hardening          Hardening         Hardening
       │                 │                 │
       ▼                 ▼                 ▼
Mode Access        VLAN 33           No DTP
Nonegotiate        Dedicated         CDP Disabled
       │                 │                 │
       └─────────────────┼─────────────────┘
                         ▼
                  🔒 Layer 2 Security
```

---

# 🛠️ Step 1 — Build the Topology

Create the multi-switch topology in **Cisco Packet Tracer**.

The topology should contain:

* 3 Cisco switches
* Multiple VLANs
* End devices
* Inter-switch trunk links
* Access ports for end devices

---

# 🏷️ Step 2 — Create and Name VLANs

Create the production VLANs and a dedicated native VLAN.

## Switch 1

```cisco
enable
configure terminal

vlan 10
name IT
exit

vlan 20
name HR
exit

vlan 30
name FINANCE
exit

vlan 33
name NATIVE
exit
```

## Switch 2

```cisco
vlan 10
name IT
exit

vlan 20
name HR
exit

vlan 30
name FINANCE
exit

vlan 33
name NATIVE
exit
```

## Switch 3

```cisco
vlan 10
name IT
exit

vlan 20
name HR
exit

vlan 30
name FINANCE
exit

vlan 33
name NATIVE
exit
```

### Important

The native VLAN should be **consistent across the trunk links**.

In this project:

```text
Native VLAN = 33
```

---

# 🔌 Step 3 — Configure Access Ports

Access ports should be explicitly configured as access interfaces.

Example for VLAN 10:

```cisco
interface range fa0/5-24
switchport mode access
switchport access vlan 10
switchport nonegotiate
exit
```

On other switches, assign the appropriate departmental VLAN.

### VLAN 20

```cisco
switchport mode access
switchport access vlan 20
switchport nonegotiate
```

### VLAN 30

```cisco
switchport mode access
switchport access vlan 30
switchport nonegotiate
```

---

# 🚫 Why `switchport nonegotiate`?

DTP is used by Cisco switches to dynamically negotiate trunking.

The objective of an end-device access port is:

```text
PC
 │
 ▼
Access Port
 │
 ├── One VLAN
 └── No Dynamic Trunk Negotiation
```

Therefore:

```cisco
switchport mode access
switchport nonegotiate
```

provides explicit control over the interface.

---

# 🌐 Step 4 — Configure Trunk Ports

Identify the interfaces connecting switch-to-switch.

Example:

```cisco
interface range fa0/1-4
switchport mode trunk
```

These interfaces carry multiple VLANs between switches.

---

# 🏷️ Step 5 — Configure the Native VLAN

Configure VLAN 33 as the native VLAN on the trunk.

```cisco
interface range fa0/1-4
switchport mode trunk
switchport trunk native vlan 33
```

Repeat the configuration on the connected switches.

### Verify

```cisco
show interfaces trunk
```

You should see VLAN 33 listed as the native VLAN.

---

# 🔐 Step 6 — Tag the Native VLAN

For stronger protection against native-VLAN-related attacks, the native VLAN can be explicitly tagged where the platform and configuration support it.

On Cisco IOS platforms that support the command:

```cisco
interface range fa0/1-4
switchport trunk native vlan tag
```

### Why?

The goal is to avoid relying on untagged native-VLAN traffic across the trunk.

> **Packet Tracer support can vary by switch model and IOS image. If this command is unavailable in your Packet Tracer device, retain the dedicated native VLAN configuration and verify the supported security features on that platform.**

---

# 🚫 Step 7 — Disable DTP on Trunk Ports Where Appropriate

For statically configured trunks, DTP negotiation can also be disabled:

```cisco
interface range fa0/1-4
switchport mode trunk
switchport nonegotiate
```

This makes the trunk configuration explicit rather than dynamically negotiated.

### Final Trunk Configuration

```cisco
interface range fa0/1-4
switchport mode trunk
switchport trunk native vlan 33
switchport nonegotiate
```

---

# 📡 Step 8 — Disable CDP

The lab also demonstrates disabling **Cisco Discovery Protocol (CDP)**.

To disable CDP globally:

```cisco
no cdp run
```

Repeat on the switches used in the lab.

### Verify

```cisco
show cdp
```

or:

```cisco
show cdp neighbors
```

Depending on the IOS/Packet Tracer implementation, the output should indicate that CDP is disabled or that no CDP neighbors are available.

---

# 🔍 Step 9 — Verify VLAN Configuration

Use:

```cisco
show vlan brief
```

Verify:

```text
VLAN 10 → IT
VLAN 20 → HR
VLAN 30 → FINANCE
VLAN 33 → NATIVE
```

---

# 🌐 Step 10 — Verify Trunk Configuration

Use:

```cisco
show interfaces trunk
```

Check:

* Trunk status
* Native VLAN
* Allowed VLANs
* Encapsulation

Example:

```text
Port      Mode      Status      Native VLAN
Fa0/1     trunk     trunking    33
```

---

# 🔍 Step 11 — Verify DTP Status

Use:

```cisco
show interfaces fa0/5 switchport
```

Check that the interface is configured as an access port.

You can also inspect the running configuration:

```cisco
show running-config interface fa0/5
```

Expected:

```text
switchport mode access
switchport access vlan 10
switchport nonegotiate
```

---

# 🧪 Security Testing

## Test 1 — Access Port Cannot Dynamically Become a Trunk

Connect an unauthorized device to an access port.

The port is explicitly configured:

```text
Access Mode
     ↓
One VLAN
     ↓
DTP Disabled
     ↓
🚫 No Dynamic Trunk Negotiation
```

This reduces the risk of switch spoofing through DTP negotiation.

---

# 🧪 Test 2 — Verify VLAN Separation

Connect devices to different VLANs.

Example:

```text
PC-A → VLAN 10
PC-B → VLAN 20
PC-C → VLAN 30
```

Without an appropriate Layer 3 routing configuration:

```text
VLAN 10 🚫 VLAN 20
VLAN 20 🚫 VLAN 30
VLAN 10 🚫 VLAN 30
```

This demonstrates the normal Layer 2 separation between VLANs.

---

# 🧪 Test 3 — Verify Native VLAN Configuration

Run:

```cisco
show interfaces trunk
```

Confirm:

```text
Native VLAN = 33
```

The same native VLAN should be configured consistently on both ends of each trunk.

---

# 🧰 Troubleshooting

## ❌ Problem 1 — Access Port Is Operating as a Trunk

Check:

```cisco
show interfaces fa0/5 switchport
```

Configure:

```cisco
interface fa0/5
switchport mode access
switchport nonegotiate
```

---

## ❌ Problem 2 — Native VLAN Mismatch

Check:

```cisco
show interfaces trunk
```

If one side uses VLAN 33 and the other uses a different native VLAN, configure both sides consistently:

```cisco
interface fa0/1
switchport trunk native vlan 33
```

---

## ❌ Problem 3 — Devices Cannot Communicate Within the Same VLAN

Check:

```cisco
show vlan brief
```

Verify that the interfaces are assigned to the correct VLAN.

Also verify the trunk:

```cisco
show interfaces trunk
```

---

## ❌ Problem 4 — Trunk Is Not Forming

Check:

```cisco
show interfaces trunk
```

Then verify:

```cisco
interface fa0/1
switchport mode trunk
```

If using a statically configured trunk, ensure the connected interface is configured consistently.

---

# 📊 Switch Spoofing vs Double-Tagging

| Feature                        | Switch Spoofing                    | Double-Tagging                                |
| ------------------------------ | ---------------------------------- | --------------------------------------------- |
| Main mechanism                 | DTP negotiation                    | Multiple VLAN tags                            |
| Attack objective               | Attempt to obtain trunk access     | Attempt to reach another VLAN                 |
| Key protection                 | Static access mode + `nonegotiate` | Dedicated native VLAN + native VLAN hardening |
| Layer                          | Layer 2                            | Layer 2                                       |
| Main protocol/feature involved | DTP                                | 802.1Q VLAN tagging                           |

---

# 🔑 Important Cisco Commands

| Purpose                 | Command                            |
| ----------------------- | ---------------------------------- |
| Create VLAN             | `vlan 33`                          |
| Name VLAN               | `name NATIVE`                      |
| Configure access mode   | `switchport mode access`           |
| Assign access VLAN      | `switchport access vlan 10`        |
| Disable DTP negotiation | `switchport nonegotiate`           |
| Configure trunk         | `switchport mode trunk`            |
| Configure native VLAN   | `switchport trunk native vlan 33`  |
| Tag native VLAN         | `switchport trunk native vlan tag` |
| Disable CDP globally    | `no cdp run`                       |
| View VLANs              | `show vlan brief`                  |
| View trunks             | `show interfaces trunk`            |
| View switchport details | `show interfaces fa0/5 switchport` |
| View CDP neighbors      | `show cdp neighbors`               |

---

# 🏗️ VLAN Hopping Prevention Architecture

```text
                         🔒 VLAN SECURITY
                               │
             ┌─────────────────┼─────────────────┐
             │                 │                 │
             ▼                 ▼                 ▼
       ACCESS PORT          TRUNK PORT          CDP
             │                 │                 │
             ▼                 ▼                 ▼
       Mode Access        Native VLAN 33     Disabled
             │            Dedicated VLAN
             │                 │
             ▼                 ▼
       Nonegotiate       Native VLAN
                         Hardening
             │                 │
             └─────────────────┼─────────────────┘
                               ▼
                     🛡️ VLAN HOPPING
                         MITIGATION
```

---

# 🧠 What I Learned

* VLAN hopping attack fundamentals
* Switch spoofing
* Double-tagging
* Dynamic Trunking Protocol (DTP)
* Cisco Discovery Protocol (CDP)
* Access-port hardening
* `switchport nonegotiate`
* Native VLAN security
* 802.1Q trunk configuration
* VLAN segmentation
* Trunk verification
* Layer 2 security practices

---

# 🏆 Project Outcome

Successfully implemented a **VLAN hopping attack prevention lab** using Cisco Packet Tracer.

The project demonstrated how to:

* 🔒 Explicitly configure access ports.
* 🚫 Disable DTP negotiation on access interfaces.
* 🌐 Configure static trunk links.
* 🏷️ Use a dedicated native VLAN.
* 🔐 Apply native VLAN hardening where supported.
* 📡 Disable CDP in the lab environment.
* 🧪 Verify VLAN and trunk configurations.
* 🛡️ Reduce the risk of switch spoofing and double-tagging attacks.

---

# 🛠️ Technologies & Tools

* **Cisco Packet Tracer**
* **Cisco IOS**
* **VLANs**
* **802.1Q Trunking**
* **DTP**
* **CDP**
* **Layer 2 Security**
* **VLAN Hopping Prevention**

---

# 📌 Project Summary

> **This project demonstrates practical Layer 2 security techniques for reducing VLAN hopping risks. Access ports are explicitly configured and prevented from negotiating trunks, trunk links use a dedicated native VLAN, and unnecessary CDP operation is disabled.**

---

# 🏷️ Tags

`#Cisco` `#CCNA` `#Networking` `#CiscoPacketTracer` `#VLAN` `#VLANHopping` `#NetworkSecurity` `#Layer2Security` `#DTP` `#CDP` `#8021Q` `#Trunking` `#NetworkEngineer` `#ITSupport`
