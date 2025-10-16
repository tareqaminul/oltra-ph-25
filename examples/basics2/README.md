# 🧪 NGINX Plus Blue-Green-Red Deployment Test Guide

This guide verifies that the Blue/Green/Red setup on **NGINX Plus** at works correctly — including traffic routing by HTTP header  
and dynamic weighted rollout using the NGINX Plus API.

---

## ⚙️ Environment Overview

| Component | Description | Example Target |
|------------|--------------|----------------|
| **NGINX Plus Gateway** | Routes traffic based on `User` header | `http://10.1.1.11` |
| **Upstream Pools** | `bluebullet`, `greenbullet`, `redbullet`, `bluegreen_bullet` | see config |
| **Header Key** | `User` | decides which upstream serves the request |
| **API Port** | `8080` | NGINX Plus live configuration API |

---

## 🧭 1. Basic Connectivity Tests

### 🔴 Red (default) — No header sent

```bash
curl -i http://10.1.1.11/
