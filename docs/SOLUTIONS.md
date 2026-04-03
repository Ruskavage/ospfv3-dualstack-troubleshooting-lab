# Solutions Reference

> ⚠️ **Spoiler Warning** — attempt to resolve all faults independently before reading this file.

---

## Fault F1 — R1-ABR: OSPFv3 Process-ID Mismatch

**Symptom:** OSPFv3 adjacency never forms between R1 and R3. `show ipv6 ospf neighbor` shows no entry for R3.

**Root Cause:** `ipv6 ospf 2 area 1` on G0/1 — process 2 instead of 1.

**Fix:**
```
interface GigabitEthernet0/1
 no ipv6 ospf 2 area 1
 ipv6 ospf 1 area 1
```

---

## Fault F2 — R1-ABR: IPv6 Unicast-Routing Missing

**Symptom:** R1 does not forward any IPv6 packets. All IPv6 pings through R1 fail.

**Root Cause:** `ipv6 unicast-routing` is missing globally.

**Fix:**
```
ipv6 unicast-routing
```

---

## Fault F3 — R2-ABR: OSPFv3 Router-ID Not Configured

**Symptom:** OSPFv3 process on R2 fails to start or uses an unstable router-id. Adjacencies flap.

**Root Cause:** `router-id` statement missing under `ipv6 router ospf 1`.

**Fix:**
```
ipv6 router ospf 1
 router-id 2.2.2.2
```

---

## Fault F4 — R2-ABR: IPv4 OSPF Area Mismatch on G0/1

**Symptom:** IPv4 routes for Area 2 subnets missing from all routers. IPv4 connectivity to WS2/WS3 fails.

**Root Cause:** `ip ospf 1 area 0` on G0/1 instead of `area 2`.

**Fix:**
```
interface GigabitEthernet0/1
 no ip ospf 1 area 0
 ip ospf 1 area 2
```

---

## Fault F5 — R3-Area1: DHCPv4 Excluded Range Covers Entire Pool

**Symptom:** WS1 cannot obtain an IPv4 DHCP address. `show ip dhcp binding` shows no entries.

**Root Cause:** `ip dhcp excluded-address 192.168.1.1 192.168.1.254` excludes every usable address.

**Fix:**
```
no ip dhcp excluded-address 192.168.1.1 192.168.1.254
ip dhcp excluded-address 192.168.1.1 192.168.1.10
```

---

## Fault F6 — R4-Area1: LAN Interface Shutdown

**Symptom:** WS1 has no connectivity at all. `show interface GigabitEthernet0/1` shows `administratively down`.

**Root Cause:** `shutdown` applied to the WS1-facing interface.

**Fix:**
```
interface GigabitEthernet0/1
 no shutdown
```

---

## Fault F7 — R4-Area1: DHCPv6 Server and ND Managed-Flag Missing

**Symptom:** WS1 gets a SLAAC address but no stateful DHCPv6 address. `show ipv6 dhcp binding` shows no WS1 entry.

**Root Cause:** `ipv6 dhcp server AREA1-V6-WS1` not applied to G0/1. `ipv6 nd managed-config-flag` also missing so clients don't attempt DHCPv6.

**Fix:**
```
interface GigabitEthernet0/1
 ipv6 dhcp server AREA1-V6-WS1
 ipv6 nd managed-config-flag
 ipv6 nd other-config-flag
```

---

## Fault F8 — R5-Area2 + R6-Area2: OSPF Timer Mismatch

**Symptom:** OSPFv2 and OSPFv3 adjacency between R5 and R6 never reaches FULL state.

**Root Cause:** R5 G0/1 uses hello 10 / dead 40. R6 G0/0 uses hello 30 / dead 120.

**Fix:** Normalise both sides to the same values (IOS default: hello 10, dead 40 recommended):
```
! On R5 GigabitEthernet0/1:
 no ip ospf hello-interval 10
 no ip ospf dead-interval 40
 no ipv6 ospf hello-interval 10
 no ipv6 ospf dead-interval 40

! On R6 GigabitEthernet0/0:
 no ip ospf hello-interval 30
 no ip ospf dead-interval 120
 no ipv6 ospf hello-interval 30
 no ipv6 ospf dead-interval 120
```

---

## Fault F9 — R5-Area2: ip helper-address Points to Non-Existent Host

**Symptom:** WS2 cannot obtain an IPv4 DHCP address. DHCP discover packets are forwarded to 192.168.3.99 which does not exist.

**Root Cause:** `ip helper-address 192.168.3.99` on G0/2. The DHCP pool AREA2-WS2 is local on R5 and needs no relay.

**Fix:**
```
interface GigabitEthernet0/2
 no ip helper-address 192.168.3.99
```

---

## Fault F10 — R6-Area2: Default Route Not Originated

**Symptom:** WS3 (192.168.3.0/24 and 2001:db8:2:300::/64) can reach R6 but nothing beyond Area 2.

**Root Cause:** `default-information originate always` missing from both OSPFv2 and OSPFv3 on R6.

**Fix:**
```
router ospf 1
 default-information originate always

ipv6 router ospf 1
 default-information originate always
```

---

## Final Verification Checklist

After applying all fixes, confirm:

```
! All adjacencies FULL
show ip ospf neighbor
show ipv6 ospf neighbor

! Complete routing tables on all routers
show ip route ospf
show ipv6 route ospf

! DHCP bindings present for all three workstations
show ip dhcp binding
show ipv6 dhcp binding

! End-to-end ping from WS1 to WS2 and WS3 (IPv4 and IPv6)
ping 192.168.2.x source 192.168.1.x
ping 2001:db8:2:200::x source 2001:db8:1:100::x
```
