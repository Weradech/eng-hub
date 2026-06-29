---
title: "Why the Camera Fails on Your LAN App: HTTP Is Not a Secure Context"
date: 2026-06-29 19:30:00 +0700
categories: [Infrastructure, Web-Platform]
tags: [web-platform, https, secure-context, getusermedia, browser, devops]
---

> **TL;DR** — Your field app works perfectly on `localhost`, then the camera, clipboard, and a few other APIs go silently dead the moment a phone opens it at `http://172.16.x.x`. Nothing is broken in your code. Browsers gate "powerful features" behind a *secure context* — HTTPS or `localhost`, and a plain-HTTP LAN IP is neither. Here's why, how to detect it, and the three ways to fix it.

---

## The symptom: works on localhost, dead on the LAN

You build a warehouse scanner page. On your dev machine at `http://localhost:3000` the camera opens fine. You deploy it to the server and operators open it on their phones at `http://192.168.1.50:3000`. The camera button does nothing. No error dialog. In the console:

```js
navigator.mediaDevices            // undefined
navigator.mediaDevices.getUserMedia(...)   // TypeError: can't read 'getUserMedia' of undefined
```

It's tempting to chase a camera-permission bug or a device problem. The cause is upstream of all of that: the browser has decided this page isn't allowed to ask for the camera *at all*, because of **where it's served from**.

---

## Secure contexts: the rule browsers don't announce

Browsers restrict "powerful features" to pages running in a [secure context](https://developer.mozilla.org/en-US/docs/Web/Security/Secure_Contexts). A context is secure when the page is delivered over **HTTPS**, or served from **`localhost`** (treated as secure for developer convenience). A plain-HTTP page on any other origin — including a LAN IP like `192.168.1.50` — is **not** secure, so the gated APIs are simply absent.

Features behind that gate include:

| Feature | Behind secure context? |
|---------|------------------------|
| `getUserMedia` (camera / mic) | Yes |
| `navigator.clipboard` | Yes |
| `crypto.subtle` (Web Crypto) | Yes |
| Service Workers | Yes |
| Geolocation (increasingly) | Yes |

This is why `localhost` lies to you: it *is* a secure context, so everything works in dev. The LAN IP is the first time the page runs un-secured, and that's exactly where your users are.

> "It works on localhost but not on the LAN IP" is the fingerprint of a secure-context problem, not a code bug. `localhost` is privileged as secure; a plain-HTTP LAN origin is not. Test on the real origin your users will use, early.
{: .prompt-warning }

---

## Detect it instead of crashing

The failure is ugly because the code assumes the API exists. Feature-detect first and you can show a useful message instead of a dead button:

```js
if (!window.isSecureContext) {
  // page is not in a secure context — gated APIs are unavailable
  showBanner("Camera needs a secure (HTTPS) connection. Ask IT for the HTTPS URL.");
} else if (!navigator.mediaDevices?.getUserMedia) {
  showBanner("This browser doesn't support camera capture.");
} else {
  enableScanner();
}
```

`window.isSecureContext` is a boolean that tells you directly whether the gated APIs are available. Checking it turns a silent `undefined` crash into a clear, actionable message — and lets the rest of the app degrade gracefully (e.g. fall back to manual entry instead of a scanner).

---

## The three fixes, worst to best

**1. Browser flag (dev only).** Chrome accepts an `unsafely-treat-insecure-origin-as-secure` flag listing specific origins to treat as secure. Fine for one developer's machine; unmanageable across a floor of operator phones. Use it to *confirm the diagnosis*, not as a deployment.

**2. Self-signed HTTPS.** Put the app behind HTTPS with a self-signed certificate. It works, but every device has to be told to trust the cert, and the warnings spook non-technical users. Acceptable for a small, fixed set of managed devices.

**3. Real TLS via a reverse proxy or tunnel (best).** Serve the app through something that terminates real TLS — a reverse proxy with a proper certificate, or a tunnel that gives the app an `https://` hostname. The phones get a clean `https://` URL with no warnings, the secure-context gate is satisfied, and you're not managing certificates by hand on every device. For internal apps this is usually the least-effort *durable* answer: one HTTPS hostname for everyone, no per-device trust dance.

---

## The field-UX angle

This bites hardest precisely where it matters most: operators on phones, scanning at a workstation, on a mobile-first page sized for a small screen. The camera *is* the feature. A "secure context" failure there isn't a degraded experience — it's a non-functional app for the exact users it was built for. Which is the real lesson: **test on the origin and device your users actually have, not on `localhost`.** The most important class of bug here is invisible in dev by construction.

---

## The portable checklist

- **`localhost` is a secure context; a plain-HTTP LAN IP is not.** Expect gated APIs to vanish off localhost.
- **Feature-detect with `window.isSecureContext`** and degrade gracefully instead of crashing on `undefined`.
- **Serve internal apps over real HTTPS** (reverse proxy or tunnel) so phones get a clean `https://` origin.
- **Test on the real device and origin early** — the bug is invisible on `localhost` by design.

---

## Related Posts

- [Tailscale + IPv6 Egress]({% post_url 2026-06-29-tailscale-ipv6-egress %}) — another "works locally, breaks on the network" trap
- [Going Paperless on GR→IQC]({% post_url 2026-06-29-paperless-gr-iqc-3phase %}) — the operator-on-a-phone workflows that depend on the camera
