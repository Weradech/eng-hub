---
title: "better-sqlite3 ABI Segfault: Why Your App Dies After a Node Upgrade"
date: 2026-06-29 09:30:00 +0700
categories: [Infrastructure, Node]
tags: [node, sqlite, native-modules, pm2, devops, reliability]
---

> **TL;DR** — `better-sqlite3` is a native Node addon. Its compiled binary is tied to a specific Node ABI version. When the system Node flips (upgrade, nvm default change, package manager update), the mismatch causes either a silent crash (`exit 139`) or an explicit `NODE_MODULE_VERSION` error. Pin your PM2 interpreter to an absolute path so the daemon can't silently switch Node versions.

---

## The Failure Signature

```
[PM2][ERROR] App [my-app] exited unexpectedly (exit code: 139)
# or
Error: The module '/app/node_modules/better-sqlite3/build/Release/better_node_sqlite3.node'
was compiled against a different Node.js version using
NODE_MODULE_VERSION 115. This version of Node.js requires
NODE_MODULE_VERSION 108.
```

Exit code 139 is `SIGSEGV` — a segmentation fault. The process tried to execute code compiled for a different memory layout. The Node ABI version is a compact proxy for that layout: every major Node release that changes the V8 or libuv ABI bumps the number.

| Node version | ABI (MODULE_VERSION) |
|-------------|---------------------|
| 18.x | 108 |
| 20.x | 115 |
| 22.x | 127 |

---

## Why It Flip-Flops

On a server managed with `nvm`, `n`, or a system package manager that allows parallel Node versions, the "active" Node is determined by `$PATH` at the time a process starts. PM2's daemon captures the Node binary from `$PATH` when it first launches. If the daemon restarts (server reboot, `pm2 kill`, startup script) under a different `$PATH`, it uses a different Node binary.

```
Boot sequence:
  /etc/systemd/system/pm2.service runs: ExecStart=/usr/bin/pm2-runtime
  /usr/bin/pm2-runtime → which node → /home/ball/.nvm/versions/node/v20/bin/node (ABI 115)

Node upgrade:
  nvm alias default 18
  PM2 daemon restarts (e.g., after server reboot)
  /home/ball/.nvm/versions/node/v18/bin/node (ABI 108) ← different node
  better-sqlite3 was compiled for ABI 115 → crash
```

The flip-flop is especially painful because it can happen between a deployment and the next reboot — the app works fine after deploy, then dies silently days later when the server restarts.

---

## The Durable Fix

### 1. Pin the interpreter in `ecosystem.config.js`

```js
module.exports = {
  apps: [{
    name: 'my-app',
    script: 'src/index.js',
    interpreter: '/usr/bin/node',      // absolute path, immune to PATH changes
    env_production: {
      NODE_ENV: 'production'
    }
  }]
}
```

Use `which node` on the version you want to pin, then hardcode that path.

```bash
nvm use 20 && which node
# /home/ball/.nvm/versions/node/v20.14.0/bin/node

# Pin to that exact binary:
interpreter: '/home/ball/.nvm/versions/node/v20.14.0/bin/node'
```

### 2. Rebuild the native module for the target Node

After pinning, rebuild so the `.node` binary matches:

```bash
# Make sure you're on the target node version
node --version   # should match your pinned interpreter

# Rebuild in the app directory
npm rebuild better-sqlite3

# Verify the ABI matches
node -e "require('better-sqlite3')" && echo "OK"
```

If the server has no internet access (offline compile), use `--build-from-source` with the source tarball pre-downloaded:

```bash
npm rebuild better-sqlite3 --build-from-source
```

### 3. Lock Node version in the repo

Add a `.nvmrc` or `.node-version` file at the repo root:

```
20.14.0
```

This makes `nvm use` and CI tools automatically switch to the correct version, reducing human error during future deployments.

---

## Recovery When Already Broken

```bash
# 1. Confirm the ABI mismatch
node -e "require('better-sqlite3')" 2>&1 | grep MODULE_VERSION

# 2. Switch to the correct node version
nvm use 20   # or whatever the app was compiled for

# 3. Rebuild the native module
cd /path/to/app && npm rebuild better-sqlite3

# 4. Restart with the pinned ecosystem config
pm2 delete my-app
pm2 start ecosystem.config.js --only my-app --env production

# 5. Verify
pm2 logs my-app --lines 20
```

---

## Prevention Checklist

- [ ] `interpreter` in `ecosystem.config.js` is an absolute path, not `"node"`
- [ ] `.nvmrc` at repo root matches the interpreter path
- [ ] `npm rebuild` is part of the deployment script, not just `npm install`
- [ ] After any system Node upgrade: `npm rebuild` + `pm2 delete + start`
- [ ] Health check includes a DB probe (`db.prepare("SELECT 1").get()`) so ABI crashes are caught immediately, not after the first real request
