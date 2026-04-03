# Troubleshooting Quick Reference

## Recommended Workflow — Layer by Layer

Work **bottom-up**. A single shutdown interface can mask multiple apparent OSPF issues above it.

```
Layer 1/2  →  Interface state
Layer 3    →  OSPF adjacencies → OSPF database → Routing table
Layer 3+   →  DHCP pool / relay → End-to-end connectivity
```

---

## Step 1 — Interface State

```
show ip interface brief
show ipv6 interface brief
```

Look for: any interface showing `administratively down`. Fix before continuing.

---

## Step 2 — OSPFv2 Adjacencies

```
show ip ospf neighbor
show ip ospf neighbor detail
show ip ospf interface <intf>
```

Check: neighbour state (should be FULL), hello/dead timers, area assignment.

---

## Step 3 — OSPFv3 Adjacencies

```
show ipv6 ospf neighbor
show ipv6 ospf neighbor detail
show ipv6 ospf interface <intf>
show ipv6 ospf | include Router ID
```

Check: router-id present, process-id consistent, adjacency state FULL.

---

## Step 4 — Routing Tables

```
show ip route ospf
show ipv6 route ospf
show ip route summary
```

Expected: routes for all area subnets visible on all routers. Missing routes = unresolved area/adjacency issue.

---

## Step 5 — DHCP

```
show ip dhcp binding
show ip dhcp pool
show ip dhcp conflict
show ipv6 dhcp binding
show ipv6 dhcp interface <intf>
debug ip dhcp server events
```

Check: bindings exist for all three workstations, pools have available addresses, helper-address is correct if relaying.

---

## Step 6 — End-to-End Ping

```
! From WS1 (after DHCP)
ping 192.168.2.x          ! WS2 IPv4
ping 192.168.3.x          ! WS3 IPv4
ping 2001:db8:2:200::x    ! WS2 IPv6
ping 2001:db8:2:300::x    ! WS3 IPv6
```

---

## Common Fault Signatures

| Symptom | Likely Fault | Check |
|---------|-------------|-------|
| OSPFv3 adj never forms on one link | Process-ID mismatch | `show ipv6 ospf interface` — compare process IDs |
| All IPv6 routing broken through one router | `ipv6 unicast-routing` missing | `show run \| include ipv6 unicast` |
| OSPFv3 won't start / flaps | `router-id` missing | `show ipv6 ospf \| include Router ID` |
| IPv4 routes missing for entire area | Wrong OSPF area on interface | `show ip ospf interface` — check area |
| DHCP offers never sent | Excluded range too broad | `show ip dhcp pool` — check available count |
| WS has no Layer 1 at all | Interface shutdown | `show interface` — check admin state |
| SLAAC works but no DHCPv6 | Missing server binding or ND flag | `show ipv6 dhcp interface` |
| OSPF adj stuck in INIT/2WAY | Hello/dead timer mismatch | `show ip ospf interface` — compare both sides |
| DHCP discover forwarded but no reply | Wrong helper-address | `show run interface` — check helper IP |
| WS can reach local router only | No default route originated | `show ip ospf database external` |
