---
title: "Tailscale + IPv6 ULA: Why Your Server Stopped Reaching the Internet"
date: 2026-06-29 09:20:00 +0700
categories: [Infrastructure, Networking]
tags: [tailscale, ipv6, networking, devops, reliability]
---

> **TL;DR** — Tailscale assigns Unique Local Addresses (ULA, `fd7a:…`) to every node. On servers where the OS prefers ULA for outbound connections, external HTTPS requests silently route through Tailscale's ULA — which has no default gateway to the internet. The fix is to tell the OS to prefer global IPv4 for external destinations.

---

![Tailscale adds an IPv6 ULA the OS prefers; the IPv6 packet routes into the tunnel to a dead end while the IPv4 path works, so it looks like a flaky ISP. The fix is to rank mapped-IPv4 first.]({{ "/assets/img/2026-06-29/tailscale-ipv6-egress.svg" | relative_url }})
_It looks like a flaky ISP; it's an IPv6 ULA the OS prefers routing into a tunnel that dead-ends._

## The Symptom

Everything looks fine. The server is reachable via SSH. Internal services talk to each other. But external API calls — Odoo cloud, package registries, webhook endpoints — start timing out. No error, no refusal, just silence until the TCP timeout fires.

```
curl https://external-api.example.com
# hangs for 30 seconds, then:
# curl: (28) Connection timed out after 30000 milliseconds
```

Meanwhile, on a machine without Tailscale:

```
curl https://external-api.example.com
# 200 OK in 300ms
```

---

## What's Happening

IPv6 address selection follows [RFC 6724](https://datatracker.ietf.org/doc/html/rfc6724). The OS sorts candidate source addresses by preference rules. ULA addresses (`fc00::/7`) rank higher than IPv4 in the default policy table on most Linux systems.

When Tailscale is installed, it adds a ULA address (`fd7a::/16`) to the network interface. The DNS resolver returns both an IPv4 and an IPv6 address for external hosts. The OS picks the ULA source for the IPv6 destination — and routes the packet into the Tailscale tunnel.

The Tailscale tunnel handles inter-node traffic on your tailnet. It does not provide a default route to the global internet unless you've explicitly configured an exit node. So the packet enters the tunnel and goes nowhere.

```
OS wants to reach 2606:4700::1 (Cloudflare)
Source candidates:
  eth0: 192.168.x.x (IPv4) — preference score: low
  tailscale0: fd7a::xxxx (ULA) — preference score: HIGH ← chosen

Packet → tailscale0 → Tailscale relay → dead end (no exit node)
```

---

## The Fix

### Option 1: Disable IPv6 on Tailscale interface (quick)

```bash
# Tell tailscale not to accept IPv6 routes
sudo tailscale up --accept-routes=false

# Or if you need routes, disable IPv6 on the tailscale0 interface
sudo sysctl -w net.ipv6.conf.tailscale0.disable_ipv6=1
```

Persist via `/etc/sysctl.d/99-tailscale.conf`:

```ini
net.ipv6.conf.tailscale0.disable_ipv6 = 1
```

### Option 2: Prefer IPv4 for external destinations (cleaner)

Edit `/etc/gai.conf` (getaddrinfo configuration):

```
# Uncomment this line to prefer IPv4 for non-local destinations:
precedence ::ffff:0:0/96  100
```

This raises IPv4-mapped addresses above ULA in the selection table without disabling IPv6 globally.

### Option 3: Verify with a probe first

Before changing anything, confirm the hypothesis:

```bash
# See what address the OS would use to reach an external host
ip route get 8.8.8.8
ip route get 2001:4860:4860::8888

# Check if tailscale0 has a ULA address
ip addr show tailscale0 | grep "fd7a"
```

If `ip route get <ipv6 external>` shows `tailscale0` as the interface, you've confirmed the problem.

---

## Why It's Hard to Diagnose

1. **SSH still works** — your SSH client connects via IPv4 (or the Tailscale tunnel for internal nodes), so the server appears healthy.
2. **Internal services still work** — Tailscale ULA routes inter-node traffic correctly. Only external egress is broken.
3. **No error message** — the packet is delivered to Tailscale's relay, which quietly drops it. From the application's perspective it's a network timeout, not a routing error.
4. **Intermittent at first** — if the external host has both A and AAAA records, some requests succeed (IPv4 path) and some fail (IPv6 ULA path), depending on DNS response order.

---

## Monitoring After the Fix

Add a simple connectivity check to your health watchdog:

```bash
# Confirms both IPv4 external egress and Tailscale internal routing work
curl -4 --max-time 5 https://1.1.1.1 -o /dev/null -s -w "%{http_code}" \
  || echo "IPv4 egress FAIL"

ping -c 1 100.x.x.x > /dev/null 2>&1 \
  || echo "Tailscale internal FAIL"
```

If IPv4 egress passes and Tailscale internal passes, the ULA routing issue is resolved.
