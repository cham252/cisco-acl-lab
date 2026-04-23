# Standard ACL Placement Notes

## The Core Rule

Standard ACLs filter traffic based on **source IP address only** — they cannot inspect destination IP, port, or protocol.

Because of this limitation, where you place a standard ACL matters enormously.

---

## Why Place Near the Destination?

If you place a standard ACL near the **source**, you block that network from reaching **all** destinations — not just the one you intend. This is usually wrong.

**Example:**
- Policy: Block PC2 (192.168.11.0/24) from WebServer only.
- If ACL is applied inbound on R1 G0/1 (near PC2's source): PC2 is blocked from WebServer AND from PC3 AND from everywhere. Too broad.
- If ACL is applied outbound on R2 G0/0 (near WebServer): PC2 is only blocked from crossing into the 192.168.20.0/24 LAN. Correct.

**Rule:** Place standard ACLs outbound on the interface closest to the destination. This minimizes blast radius and enforces only the intended policy.

---

## in vs. out Direction

| Direction | Filters packets...                              | Applied relative to... |
|-----------|--------------------------------------------------|------------------------|
| `in`      | Entering the router from that interface          | The source side        |
| `out`     | Leaving the router out that interface            | The destination side   |

In this lab, both ACLs use `out` on the G0/0 interface facing the destination LAN.

---

## Wildcard Mask Reference

Wildcard masks are the inverse of subnet masks.

| Subnet Mask     | Wildcard Mask   | Meaning                        |
|-----------------|-----------------|--------------------------------|
| 255.255.255.255 | 0.0.0.0         | Match this exact host          |
| 255.255.255.0   | 0.0.0.255       | Match any host in /24 network  |
| 255.255.0.0     | 0.0.255.255     | Match any host in /16 network  |
| 0.0.0.0         | 255.255.255.255 | Match any address (same as `any`) |

Bit logic:
- `0` in wildcard = **must match** the corresponding bit in the ACL address
- `1` in wildcard = **ignore** that bit (any value allowed)

---

## Implicit Deny All

Every ACL — numbered or named, standard or extended — ends with an invisible `deny any` statement.

```
! This is always present, even if not shown:
access-list 1 deny any   ! implicit — cannot be seen in show output
```

If you do not include an explicit `permit any` (or some permit statement covering remaining traffic), all unmatched traffic is silently dropped. This is one of the most common ACL misconfigurations.

**Always add `permit any` at the end of a standard ACL** unless your intent is to block all unmatched traffic.

---

## ACL Verification Workflow

1. Configure ACL statements
2. Run `show access-lists` to verify syntax before applying
3. Apply to interface with `ip access-group [number] [in|out]`
4. Run connectivity tests
5. Run `show access-lists` again — confirm match counters are non-zero on deny rules

Non-zero match counts on deny statements prove the ACL is hitting traffic. Zero matches after testing usually means wrong interface, wrong direction, or wrong ACL number.
