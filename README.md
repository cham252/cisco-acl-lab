# Configure Numbered Standard IPv4 ACLs

**Lab:** GLAB-125.4.2 — Per Scholas Cybersecurity Program  
**Score:** 100/100 · 4/4 Items Correct  
**Date:** April 21, 2026  
**Platform:** Cisco Packet Tracer · IOS CLI  

---

## Objective

Deploy two numbered standard ACLs across a three-router topology to enforce specific network isolation policies. Verify enforcement with six structured connectivity tests and confirm active match counters on both routers.

**Part 1:** Plan ACL implementation — analyze policies, determine filtering criteria, choose interface placement.  
**Part 2:** Configure, apply, and verify — write ACL statements, apply to correct interfaces with correct direction, validate with `show access-lists` and live pings.

---

## Topology

```
                         192.168.20.0/24
                         WebServer: 192.168.20.254
                              |
                             R2 (ACL 1 out — G0/0)
                            / \
              10.1.1.0/30  /   \  10.2.2.0/30
                          /     \
                        R1 ---- R3 (ACL 1 out — G0/0)
                       / \        \
     192.168.10.0/24  /   \        \ 192.168.30.0/24
     PC1: .10.10     /     \        PC3: .30.10
                    /  192.168.11.0/24
                        PC2: .11.10
```

Three-router topology. EIGRP pre-configured. ACLs applied outbound on G0/0 of R2 and R3.

---

## Addressing Table

| Device    | Interface | IP Address      | Subnet Mask     | Default Gateway |
|-----------|-----------|-----------------|-----------------|-----------------|
| R1        | G0/0      | 192.168.10.1    | 255.255.255.0   | N/A             |
| R1        | G0/1      | 192.168.11.1    | 255.255.255.0   | N/A             |
| R1        | S0/0/0    | 10.1.1.1        | 255.255.255.252 | N/A             |
| R1        | S0/0/1    | 10.3.3.1        | 255.255.255.252 | N/A             |
| R2        | G0/0      | 192.168.20.1    | 255.255.255.0   | N/A             |
| R2        | S0/0/0    | 10.1.1.2        | 255.255.255.252 | N/A             |
| R2        | S0/0/1    | 10.2.2.1        | 255.255.255.252 | N/A             |
| R3        | G0/0      | 192.168.30.1    | 255.255.255.0   | N/A             |
| R3        | S0/0/0    | 10.3.3.2        | 255.255.255.252 | N/A             |
| R3        | S0/0/1    | 10.2.2.2        | 255.255.255.252 | N/A             |
| PC1       | NIC       | 192.168.10.10   | 255.255.255.0   | 192.168.10.1    |
| PC2       | NIC       | 192.168.11.10   | 255.255.255.0   | 192.168.11.1    |
| PC3       | NIC       | 192.168.30.10   | 255.255.255.0   | 192.168.30.1    |
| WebServer | NIC       | 192.168.20.254  | 255.255.255.0   | 192.168.20.1    |

---

## Network Policies

### R2 — WebServer Protection
- **Deny:** 192.168.11.0/24 (PC2) access to WebServer (192.168.20.254)
- **Permit:** All other traffic
- **Placement:** G0/0 outbound on R2 (interface facing 192.168.20.0/24)

### R3 — PC3 Network Isolation
- **Deny:** 192.168.10.0/24 (PC1) access to 192.168.30.0/24 (PC3)
- **Permit:** All other traffic
- **Placement:** G0/0 outbound on R3 (interface facing 192.168.30.0/24)

> **Standard ACL placement rule:** Standard ACLs filter on source IP only. Placing them near the source blocks that traffic to ALL destinations. Place them outbound on the interface closest to the destination to limit blast radius.

---

## Configuration

See `configs/` directory for full IOS command files.

### R2 — ACL Configuration

```
R2> enable
R2# configure terminal

! Deny source network 192.168.11.0/24 — wildcard 0.0.0.255
R2(config)# access-list 1 deny 192.168.11.0 0.0.0.255

! Override implicit deny-all — permit all other traffic
R2(config)# access-list 1 permit any

! Verify before applying
R2# show access-lists

! Apply outbound on G0/0 — traffic leaving toward WebServer LAN
R2(config)# interface GigabitEthernet0/0
R2(config-if)# ip access-group 1 out
```

### R3 — ACL Configuration

```
R3> enable
R3# configure terminal

! Deny source network 192.168.10.0/24 — wildcard 0.0.0.255
R3(config)# access-list 1 deny 192.168.10.0 0.0.0.255

! Permit all other traffic
R3(config)# access-list 1 permit any

! Verify before applying
R3# show access-lists

! Apply outbound on G0/0 — traffic leaving toward PC3 LAN
R3(config)# interface GigabitEthernet0/0
R3(config-if)# ip access-group 1 out
```

---

## Verification Results

### show access-lists Output — After Testing

**R2:**
```
Standard IP access list 1
  10 deny   192.168.11.0 0.0.0.255   (4 match(es))
  20 permit any                       (8 match(es))
```

**R3:**
```
Standard IP access list 1
  10 deny   192.168.10.0 0.0.0.255   (4 match(es))
  20 permit any                       (8 match(es))
```

### Connectivity Test Results — 6/6 Passed

| Source                  | Destination             | Result    | Reason                                              |
|-------------------------|-------------------------|-----------|-----------------------------------------------------|
| PC1 · 192.168.10.10     | PC2 · 192.168.11.10     | PASS      | Neither ACL blocks this traffic path                |
| PC1 · 192.168.10.10     | WebServer · 192.168.20.254 | PASS   | R2 ACL only blocks 192.168.11.0 — PC1 is permitted |
| PC2 · 192.168.11.10     | WebServer · 192.168.20.254 | BLOCKED | R2 ACL deny rule matches 192.168.11.0 outbound G0/0 |
| PC1 · 192.168.10.10     | PC3 · 192.168.30.10     | BLOCKED   | R3 ACL deny rule matches 192.168.10.0 outbound G0/0 |
| PC2 · 192.168.11.10     | PC3 · 192.168.30.10     | PASS      | R3 ACL only blocks 192.168.10.0 — PC2 is permitted |
| PC3 · 192.168.30.10     | WebServer · 192.168.20.254 | PASS   | Neither ACL blocks this traffic path                |

---

## Key Concepts

**Wildcard Masks**  
The inverse of a subnet mask. `255.255.255.0` → `0.0.0.255`. A `0` bit means "must match." A `1` bit means "any value." Used to define the range of IP addresses an ACL statement targets.

**Implicit Deny All**  
Every ACL ends with an invisible `deny any`. Without an explicit `permit any`, all unmatched traffic is dropped — a frequent misconfiguration that cuts off legitimate traffic.

**in vs. out Direction**  
`ip access-group 1 out` filters packets as they *leave* the interface toward the destination LAN. `in` filters packets *entering* from that LAN — a different enforcement point entirely.

---

## Tools & Environment

| Category     | Items                                                      |
|--------------|------------------------------------------------------------|
| Platform     | Cisco Packet Tracer, IOS CLI                               |
| ACL Type     | Standard IPv4 (numbered)                                   |
| Routing      | EIGRP (pre-configured)                                     |
| Interfaces   | GigabitEthernet (LAN), Serial (WAN)                        |
| Concepts     | Wildcard Mask, Implicit Deny, Access-group Outbound, Match Counter Verification, Source-based Filtering |

---

## Repository Structure

```
acl-lab/
├── README.md                        # This file — full lab documentation
├── configs/
│   ├── R2-acl-config.txt            # R2 IOS ACL configuration
│   └── R3-acl-config.txt            # R3 IOS ACL configuration
├── verification/
│   ├── show-access-lists-R2.txt     # R2 show access-lists output
│   ├── show-access-lists-R3.txt     # R3 show access-lists output
│   └── connectivity-test-results.md # Six-test verification table
└── docs/
    └── acl-placement-notes.md       # Notes on standard ACL placement logic
```

---

*Part of Christopher Ham's cybersecurity portfolio — [cham252.github.io](https://cham252.github.io)*  
*Per Scholas NC Cybersecurity with AI Tools · 2026*
