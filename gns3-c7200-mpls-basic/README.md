# Basic MPLS (LDP) Lab — GNS3 with Cisco 7200

A clean, beginner-friendly MPLS lab you can run in **GNS3** using **Cisco 7200** routers.

This lab demonstrates the classic MPLS data-plane behavior:

- **Ingress PE** pushes a label
- **P (core)** swaps labels
- **Egress PE** pops the label (or PHP occurs one hop before)

> This is a **basic MPLS transport** lab (LDP + OSPF). It is **not** an MPLS L3VPN/VRF lab.

---

## Topology

```
CE1 — PE1 — P — PE2 — CE2
```

Color legend (in the diagram):
- **CE** (Customer Edge): blue
- **PE** (Provider Edge): green
- **P** (Provider core): red

See: [`diagrams/basic-mpls-topology.png`](diagrams/basic-mpls-topology.png)

---

## Addressing Plan

| Link | Network | Left | Right |
|---|---|---:|---:|
| CE1–PE1 | 10.0.10.0/24 | CE1: 10.0.10.1 | PE1: 10.0.10.2 |
| PE1–P | 10.0.12.0/24 | PE1: 10.0.12.1 | P: 10.0.12.2 |
| P–PE2 | 10.0.23.0/24 | P: 10.0.23.2 | PE2: 10.0.23.1 |
| PE2–CE2 | 10.0.20.0/24 | PE2: 10.0.20.2 | CE2: 10.0.20.1 |

Loopbacks (provider):
- PE1 Lo0: **1.1.1.1/32**
- P   Lo0: **2.2.2.2/32**
- PE2 Lo0: **3.3.3.3/32**

---

## Interface Mapping (matches your GNS3 output)

These configs match the interfaces you showed:

| Device | CE-facing | Core-facing |
|---|---|---|
| CE1 | Fa0/0 | — |
| PE1 | Fa0/0 (to CE1) | Fa1/0 (to P) |
| P   | Fa0/0 (to PE1) | Fa1/0 (to PE2) |
| PE2 | Fa1/0 (to CE2) | Fa0/0 (to P) |
| CE2 | Fa0/0 | — |

If your interface numbers differ in the future, run:

```cisco
show ip int brief
```

…and swap interface names in the configs.


## Build Steps (GNS3)

1. Add **5x Cisco 7200** routers (CE1, PE1, P, PE2, CE2).
2. Ensure each PE/P router has **at least 2 FastEthernet** interfaces:
   - In GNS3: Right-click router → **Configure** → **Slots** → add a FE module (e.g., `PA-FE-TX`) to additional slots.
3. Cable the routers in a line: `CE1—PE1—P—PE2—CE2`.
4. Start devices and paste the configs from `configs/` (one file per router).
5. Save on each router:

```cisco
write memory
```

---

## What Runs Where (by design)

### Customer edges (CE1/CE2)
- Only IP
- Static routes toward the provider edge

### Provider core (PE1/P/PE2)
- **OSPF area 0** on core links + loopbacks
- **CEF enabled**
- **MPLS + LDP** enabled on **core-facing** interfaces only (PE↔P links)

---

## Verification

### 1) Check core routing (OSPF)
On PE1 / P / PE2:

```cisco
show ip ospf neighbor
show ip route ospf
```

You should see neighbors on the PE↔P links.

### 2) Check LDP neighbors
On PE1 / P / PE2:

```cisco
show mpls ldp neighbor
```

You should see:
- PE1 ↔ P
- P ↔ PE2

### 3) Check MPLS forwarding
On PE1 / P / PE2:

```cisco
show mpls forwarding-table
```

You should see labels assigned for FECs (routes).

### 4) End-to-end ping
From CE1:

```cisco
ping 10.0.20.1
```

From CE2:

```cisco
ping 10.0.10.1
```

---

## Troubleshooting (fast)

### A) CE link doesn’t ping (PE2 → 10.0.20.1 fails)
1. Verify both sides are `up/up`:
   ```cisco
   show ip int brief
   ```
2. Confirm correct subnet and masks on both ends.
3. Make sure the cable is on the interface you configured.
4. Bounce the interface:
   ```cisco
   conf t
   int faX/Y
    shut
    no shut
   end
   ```

### B) OSPF neighbors aren’t forming
- Check IPs/masks on core links
- Ensure both sides are in area 0
- Verify `router-id` is set and loopback is up

### C) LDP neighbors missing
- Confirm `ip cef` is enabled
- Confirm `mpls ip` is on core-facing interfaces
- Confirm `mpls ldp router-id Loopback0 force` on PE/P

---

## Files

- `configs/` — copy/paste IOS configs per router
- `diagrams/` — topology diagram PNG
- `docs/` — notes and quick commands

---

## License
MIT (use freely)
