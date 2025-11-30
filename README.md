

# 🔥 Cisco Zone-Based Firewall (ZPF) Configuration – Router R3

This project implements a **Zone-Based Firewall (ZPF)** on the Cisco router **R3**.
The objective is to secure the network by allowing internal hosts to access external resources while blocking any unauthorized traffic coming from outside the network.

The lab follows a step-by-step configuration approach including router security, HDFS-like zone logic (inside/outside), traffic inspection, and policy enforcement.

---

## 🎯 Objectives

* Configure a **Cisco ZPF** on R3 to protect internal LAN traffic.
* Allow **internal hosts** (PC-C) to access the **external network** (PC-A).
* Block **external hosts** (PC-A, R2) from accessing the internal LAN.
* Verify the firewall behavior using:

  * `ping`
  * `SSH`
  * Web browser access (HTTP)

---

## 🧩 Network Scenario

R3 acts as the **perimeter router** between the **IN-ZONE** (internal network) and the **OUT-ZONE** (external network).
The firewall inspects traffic from inside to outside and denies any unsolicited connections from outside to inside.

---

## 🔐 Security Credentials

| Access Type      | Credential            | Purpose                      |
| ---------------- | --------------------- | ---------------------------- |
| Console          | `ciscoconpa55`        | Physical access to router    |
| VTY (SSH/Telnet) | `ciscovtypa55`        | Remote access to router      |
| Enable secret    | `ciscoenpa55`         | Privileged EXEC mode         |
| Local user       | `Admin` / `Adminpa55` | SSH authentication           |
| SSH to R2        | Admin / Adminpa55     | Remote access test from PC-C |

---

## 🛠️ Configuration Steps

### **1. Initial Router Setup**

* Set console, VTY, and enable passwords
* Configure hostname and IP addressing
* Set up local user accounts
* Enable and secure SSH access
* Configure static or RIP routing

---

### **2. Connectivity Verification (Before Firewall)**

* **Ping** PC-C → PC-A
* **SSH** from PC-C to R2 (`10.2.2.2`)
* **HTTP test** from PC-C to PC-A (`192.168.1.3`)

---

### **3. Enable Cisco Security Package**

Verify and activate securityk9:

```bash
show version
license boot module c1900 technology-package securityk9
```

Save and reload the router.

---

### **4. Create Firewall Zones**

```bash
zone security IN-ZONE
zone security OUT-ZONE
```

---

### **5. Define Traffic (Class-Map + ACL)**

```bash
access-list 101 permit ip 192.168.3.0 0.0.0.255 any

class-map type inspect match-all IN-NET-CLASS-MAP
 match access-group 101
```

---

### **6. Create Firewall Policy (Policy-Map)**

```bash
policy-map type inspect IN-2-OUT-PMAP
 class type inspect IN-NET-CLASS-MAP
  inspect
```

---

### **7. Apply ZPF Using Zone-Pair**

```bash
zone-pair security IN-2-OUT-ZPAIR source IN-ZONE destination OUT-ZONE
 service-policy type inspect IN-2-OUT-PMAP
```

Assign interfaces:

```bash
interface g0/1
 zone-member security IN-ZONE

interface s0/0/1
 zone-member security OUT-ZONE
```

---

### **8. Firewall Testing – Allowed Traffic (IN → OUT)**

* **Ping** PC-A from PC-C
* **SSH** from PC-C → R2
* **HTTP** browsing PC-C → PC-A

Check active sessions:

```bash
show policy-map type inspect zone-pair sessions
```

---

### **9. Firewall Testing – Blocked Traffic (OUT → IN)**

* Ping PC-C from **PC-A → FAIL**
* Ping PC-C from **R2 → FAIL**

The firewall successfully blocks inbound traffic from the OUT-ZONE.

---

## 📄 Deliverables (as required by the lab)

* Router configurations
* ZPF policy and zone definitions
* Connectivity tests (before/after)
* Screenshots of inspection sessions
* Commentary on commands used
* Additional tests from your own exploration

---

📡 Network Topology (ASCII Diagram)

                 ┌───────────────────────┐
                 │       OUT-ZONE        │
                 │   (External Network)  │
                 └──────────┬────────────┘
                            │
                            │ S0/0/1
                       +-----------+
                       |    R3     |   ← Zone-Based Firewall (ZPF)
                       | (Firewall)| 
                       +-----------+
                            │ G0/1
                            │
                 ┌───────────────────────┐
                 │        IN-ZONE        │
                 │    (Internal LAN)     │
                 └──────────┬────────────┘
                            │
                    192.168.3.0/24
                            │
                         +------+
                         | PC-C |
                         +------+

External Network Side:
    PC-A (192.168.1.3)
    R2 (10.2.2.2)

Allowed Traffic:
    IN-ZONE  → OUT-ZONE  (Ping, SSH, HTTP)

Blocked Traffic:
    OUT-ZONE → IN-ZONE  (All unsolicited connections)

## ✅ Summary

This project demonstrates how to secure a Cisco router using **Zone-Based Policy Firewall (ZPF)**:

* 🔒 Only inside → outside traffic is permitted
* 🚫 Outside → inside is fully blocked
* 🔍 SSH/HTTP sessions are dynamically inspected
* 🛡️ The network is protected by modern Cisco security mechanisms

---

