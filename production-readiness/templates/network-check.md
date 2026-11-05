# Client-IP diagnostic endpoint (admin only)

Purpose: see, on the deployed stack, which address your app would use for 0…5 trusted proxy hops and whether the CDN's client-IP header is usable. Everything is **masked**; nothing is logged.

## Behaviour

`GET /api/admin/network-check` (administrators only) returns:

```json
{
  "configuredHops": 1,
  "forwardedForEntries": 5,
  "hasCdnClientIp": true,
  "cdnClientIpMasked": "203.0.113.x",
  "trustCdnHeader": false,
  "cdnClientIpValid": true,
  "resolvedSource": "trust-proxy-hops",
  "resolvedMasked": "10.0.0.x",
  "candidates": [
    { "hops": 0, "masked": "…", "looksInternal": true,  "current": false },
    { "hops": 1, "masked": "10.0.0.x", "looksInternal": true, "current": true },
    { "hops": 3, "masked": "198.51.100.x", "looksInternal": false, "current": false }
  ]
}
```

(Addresses above are documentation ranges.)

## Sketch (Express + TypeScript)

```ts
import type { Request } from 'express';
import net from 'node:net';

const mask = (ip: string) =>
  net.isIPv4(ip) ? ip.replace(/\.\d+$/, '.x') : ip.replace(/:[^:]*$/, ':x');

const looksInternal = (ip: string) =>
  /^(10\.|127\.|192\.168\.|172\.(1[6-9]|2\d|3[01])\.|169\.254\.|fc|fd|::1)/i.test(ip);

// Resolve the address you would get if you trusted `hops` proxies from the right of X-Forwarded-For.
function atHops(req: Request, hops: number): string {
  const chain = String(req.headers['x-forwarded-for'] ?? '')
    .split(',').map((s) => s.trim()).filter(Boolean);
  const socket = req.socket.remoteAddress ?? '';
  const full = [...chain, socket];
  return full[Math.max(0, full.length - 1 - hops)] ?? socket;
}

// Single valid IP only: reject lists, repeated headers and zone ids.
export function parseCdnIp(value: unknown): string | null {
  if (typeof value !== 'string' || value.includes(',') || value.includes('%')) return null;
  const ip = value.trim();
  return net.isIP(ip) ? ip : null;
}
```

Wire it **after** `app.set('trust proxy', hops)`, behind your admin guard, and call it from two networks. Never expose it publicly and never write the raw addresses to logs.

## Decision rule

- If hops 1–2 are internal and the CDN header holds a different public address per network: trust the CDN header (and write down that the origin must only be reachable through the CDN).
- If there is no CDN header: choose the smallest hops value that yields a public address that differs per network.
- If the address is the same from different networks: you are rate-limiting everyone as one; fix before launch.
