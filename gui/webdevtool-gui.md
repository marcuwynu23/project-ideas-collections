# HostScope

## A Native Windows Developer Tool for HTTP Debugging & Virtual Host Mapping

---

## 1. One-line Summary

**HostScope** is a native Windows desktop developer tool designed to
test and debug HTTP behavior with a strong focus on **virtual host
routing, header inspection, and local-to-production parity**, without
relying on curl, Postman, or full browsers.

It allows developers to simulate requests such as:

```http
Host: app1.project.com
→ http://localhost
```

---

## 2. Problem Statement

Modern web systems heavily depend on:

- Virtual hosting
- Reverse proxies and ingress controllers
- Header-based routing and TLS termination

However, existing tools have major gaps:

Tool Limitation

---

Web Browsers Cannot reliably override Host headers
curl CLI-only, poor visualization
Postman Heavy, cloud-oriented, not host-routing-focused
Browser DevTools Tied to browser networking model
Proxy tools Overkill for day-to-day development

**There is no lightweight, native, developer-first tool focused on
Host/header-level HTTP behavior.**

---

## 3. Target Users

- Backend / Full‑stack developers\
- DevOps & SRE engineers\
- Reverse proxy & ingress maintainers\
- API gateway designers\
- Developers working with:
  - NGINX, Traefik, Envoy
  - Kubernetes Ingress
  - Cloudflare Tunnel / Zero Trust
  - Multi-domain SaaS backends

---

## 4. Core Use Cases

### 4.1 Virtual Host Mapping

Test production routing locally without DNS hacks:

```text
Host: app1.project.com → http://localhost:3000
Host: api.project.com  → http://localhost:8080
```

---

### 4.2 Header-Level HTTP Inspection

Inspect:

- Headers sent vs received
- Redirect chains
- Gateway and proxy mutations

---

### 4.3 Reverse Proxy & Ingress Debugging

Validate:

- Host routing rules
- X-Forwarded-\* headers
- TLS offloading behavior
- HTTP → HTTPS redirects

---

### 4.4 API Debugging Without Browser Noise

No cookies, extensions, caching, or CORS side-effects. Pure HTTP, fully
controlled.

---

## 5. Key Features (MVP)

### Request

- Method selector (GET, POST, PUT, PATCH, DELETE)
- URL input
- Host override field
- Custom request headers
- Raw request body (JSON/text)
- Timeout configuration
- Redirect toggle
- Insecure TLS option (dev-only)

### Response

- Status code & timing
- Final resolved URL
- Full response headers
- Body preview (safe-capped)
- Redirect chain display

### UX

- Native Windows look & feel
- Async non-blocking requests
- Tabbed request/response view
- Offline & local-first

---

## 6. Non-Goals (Intentional Scope)

- No REST collections or workspaces
- No authentication vault
- No cloud sync or telemetry
- No API mocking

**HostScope is a focused HTTP inspection tool, not an API platform.**

---

## 7. Technical Architecture

### Stack

- Language: **Groovy (JVM)**
- Runtime: Java 17+
- UI: Swing (native Look & Feel)
- Networking: `java.net.http.HttpClient`

### Architecture Overview

```text
UI (Swing)
 ├─ Request Builder
 ├─ Response Viewer
 └─ Async Controller
      ↓
HTTP Inspector Core
 ├─ Redirect Engine
 ├─ Header Manager
 ├─ TLS Strategy
 └─ Timing & Metrics
```

---

## 8. Security & Safety Design

- Local-first defaults
- Explicit insecure TLS toggle
- No stored credentials
- No background traffic
- Optional future allow-list mode

---

## 9. Roadmap

### Phase 2 (Power Developer)

- History sidebar
- Cookie jar (opt-in)
- Export as curl / PowerShell
- Raw HTTP wire view

### Phase 3 (DevOps)

- Host profiles
- Parallel request runner
- Request diffing
- Reverse proxy simulation

### Phase 4 (Advanced)

- HTTP/2 & HTTP/3 visibility
- mTLS client certificates
- WebSocket inspection
- Ingress rule validator

---

## 10. Differentiation

Feature HostScope Browser curl Postman

---

Host override ✅ ❌ ✅ ⚠️
Visual headers ✅ ⚠️ ❌ ✅
Local-first ✅ ❌ ✅ ❌
Lightweight ✅ ❌ ✅ ❌
No cloud ✅ ❌ ✅ ❌

---

## 11. Open Source Strategy

- License: MIT or Apache‑2.0
- Clear README & screenshots
- Emphasis on:
  - HTTP fundamentals
  - JVM networking
  - UI threading discipline
  - Test-driven design

Ideal as a **senior developer portfolio project**.

---

## 12. Why This Project Matters

HostScope demonstrates:

- Deep understanding of HTTP
- Developer tooling design
- UX + low-level networking balance
- Intentional scope control
- Production-grade engineering discipline

This is the kind of project that signals **senior-level capability**,
not just feature output.
