# 🧪 NGINX Plus Blue-Green Deployment Test Guide

This guide verifies that the Blue/Green/Red setup on **NGINX Plus** works correctly — including traffic routing by HTTP header  
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

curl -i -H "User: bluegroup" http://10.1.1.11/

curl -i -H "User: bluegreen" http://10.1.1.11/

for i in {1..100}; do
  curl -s -o /dev/null -w "%{remote_ip}\n" -H "User: bluegreen" http://10.1.1.11/
done | sort | uniq -c
```

## 🧭 2.Testing the NGINX Plus API

### a. Check API root
```bash
curl -s http://10.1.1.11:8080/api/6/ | jq .
```
### b. List all upstreams
```bash
curl -s http://10.1.1.11:8080/api/6/http/upstreams/ | jq .
```
### c. Inspect the blue-green upstream
```bash
curl -s http://10.1.1.11:8080/api/6/http/upstreams/bluegreen_bullet/servers/ | jq .
```
