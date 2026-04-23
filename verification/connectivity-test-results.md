# Connectivity Test Results — GLAB-125.4.2

Six structured ping tests run after ACL deployment.  
All results matched expected behavior. Score: 6/6.

---

## Test Matrix

| # | Source                      | Destination                    | Result    | ACL Hit         | Reason                                                 |
|---|-----------------------------|-------------------------------|-----------|-----------------|--------------------------------------------------------|
| 1 | PC1 · 192.168.10.10         | PC2 · 192.168.11.10            | PASS      | None            | Neither ACL blocks this path                           |
| 2 | PC1 · 192.168.10.10         | WebServer · 192.168.20.254     | PASS      | R2 permit any   | R2 ACL blocks .11.0, not .10.0 — PC1 is permitted      |
| 3 | PC2 · 192.168.11.10         | WebServer · 192.168.20.254     | BLOCKED   | R2 deny stmt 10 | R2 ACL denies 192.168.11.0/24 outbound on G0/0         |
| 4 | PC1 · 192.168.10.10         | PC3 · 192.168.30.10            | BLOCKED   | R3 deny stmt 10 | R3 ACL denies 192.168.10.0/24 outbound on G0/0         |
| 5 | PC2 · 192.168.11.10         | PC3 · 192.168.30.10            | PASS      | R3 permit any   | R3 ACL blocks .10.0, not .11.0 — PC2 is permitted      |
| 6 | PC3 · 192.168.30.10         | WebServer · 192.168.20.254     | PASS      | None            | Neither ACL blocks this path                           |

---

## Pre-Test Baseline

Before configuring any ACLs, full connectivity was confirmed by pinging all devices from each PC.  
All pings succeeded. This established the pre-ACL baseline.

---

## Verification Commands Used

```
! Run from each source PC in Packet Tracer
ping 192.168.11.10      ! PC1 → PC2
ping 192.168.20.254     ! PC1 → WebServer
ping 192.168.20.254     ! PC2 → WebServer  (expected: blocked)
ping 192.168.30.10      ! PC1 → PC3        (expected: blocked)
ping 192.168.30.10      ! PC2 → PC3
ping 192.168.20.254     ! PC3 → WebServer

! Run on each router after all pings
show access-lists
```

---

## Match Counter Summary

| Router | Statement | Matches | Interpretation                  |
|--------|-----------|---------|---------------------------------|
| R2     | 10 deny   | 4       | 4 packets from PC2 → WebServer blocked |
| R2     | 20 permit | 8       | PC1 and PC3 traffic to WebServer forwarded |
| R3     | 10 deny   | 4       | 4 packets from PC1 → PC3 blocked |
| R3     | 20 permit | 8       | PC2 and other permitted flows forwarded |

Non-zero match counts on both deny and permit statements confirm the ACLs are correctly applied and actively enforcing policy.
