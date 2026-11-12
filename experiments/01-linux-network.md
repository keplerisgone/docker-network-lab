# Experiment 01 — Linux Network Fundamentals

**Date:** 2026-11-11  
**Environment:** Arch Linux (Omarchy)  
**Topics:** Routing, ARP, Neighbor Table, ICMP

## 1. Objective

Understand how Linux selects network routes, resolves link-layer addresses, and exchanges IP packets.

## 2. Routing Experiment

Commands:

```bash
ip route get 172.30.1.254
ip route get 8.8.8.8
ip route get 172.17.0.2
```

### Observations

| Destination | Interface | Next Hop | Source IP |
|---|---|---|---|
| 172.30.1.254 | wlo1 | Direct | 172.30.1.8 |
| 8.8.8.8 | wlo1 | 172.30.1.254 | 172.30.1.8 |
| 172.17.0.2 | docker0 | Direct | 172.17.0.1 |

### Analysis

Linux selects a route according to its routing configuration, typically choosing the most specific matching prefix.

- `dev` identifies the outgoing interface.
- `via` identifies the next-hop router.
- `src` identifies the selected source IP address.

A successful route lookup does not necessarily guarantee packet delivery.

## 3. ARP Experiment

Commands:

```bash
ip neigh show dev wlo1
sudo tcpdump -eni wlo1 'arp'
```

### Observations

The gateway `172.30.1.254` appeared in the neighbor table with a MAC address and the `REACHABLE` state.

Captured traffic included ARP requests and replies between hosts on the local network.

Example from the capture:

```text
Request who-has 172.30.1.74 tell 172.30.1.8
Reply 172.30.1.74 is-at e2:1b:25:e6:5b:93
```

### Analysis

ARP resolves an IPv4 address to a link-layer address on an ARP-capable network.

Linux stores resolved addresses in its Neighbor Table, reducing the need for repeated ARP requests.

The capture demonstrated address resolution, although not every captured ARP exchange was caused by our test commands.

## 4. ICMP Experiment

Commands:

```bash
sudo tcpdump -ni wlo1 'icmp and host 172.30.1.254'
ping -c 3 172.30.1.254
```

### Observations

```text
172.30.1.8 > 172.30.1.254: ICMP echo request, seq 1
172.30.1.254 > 172.30.1.8: ICMP echo reply, seq 1
```

Three Echo Requests and three corresponding Echo Replies were captured.

**Result:** 3 packets transmitted, 3 received, 0% packet loss.

### Analysis

The experiment confirmed bidirectional ICMP communication between the host and its default gateway.

Routing determines the outgoing path, while neighbor resolution provides the link-layer address needed for local delivery.

## 5. Conclusion

- Verified Linux routing decisions using `ip route get`.
- Observed IPv4-to-MAC address resolution using ARP.
- Inspected the Linux Neighbor Table.
- Captured ICMP Echo Requests and Replies using tcpdump.

**Remaining:** An active TCP/UDP socket-binding experiment was not performed. Socket inspection was completed previously using `ss`.

