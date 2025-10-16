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
## 🚀 3. Gradual Rollout (Dynamic Weighted Promotion)
You can shift live traffic share from blue → green using the API, without reloading NGINX Plus!

### Example: move to 50/50 split
```bash
curl -X PATCH -d '{"weight":50}' http://10.1.1.11:8080/api/6/http/upstreams/bluegreen_bullet/servers/0
curl -X PATCH -d '{"weight":50}' http://10.1.1.11:8080/api/6/http/upstreams/bluegreen_bullet/servers/1
```
### Example: promote green to 100%
```bash
curl -X PATCH -d '{"weight":0}'  http://10.1.1.11:8080/api/6/http/upstreams/bluegreen_bullet/servers/0
curl -X PATCH -d '{"weight":100}' http://10.1.1.11:8080/api/6/http/upstreams/bluegreen_bullet/servers/1
```
### Verify updated weights
```bash
curl -s http://10.1.1.11:8080/api/6/http/upstreams/bluegreen_bullet/servers/ | jq '.[] | {server,weight}'
```
## 🔄 4. Rollback Procedure (Instant Recovery)
```bash
curl -X PATCH -d '{"weight":95}' http://10.1.1.11:8080/api/6/http/upstreams/bluegreen_bullet/servers/0
curl -X PATCH -d '{"weight":5}'  http://10.1.1.11:8080/api/6/http/upstreams/bluegreen_bullet/servers/1
```
### Confirm:
```bash
curl -s http://10.1.1.11:8080/api/6/http/upstreams/bluegreen_bullet/servers/ | jq '.[] | {server,weight}'
```
## 🩺 5. Health Check and Observability
### Check NGINX Plus active health and runtime stats:
```bash
curl -s http://10.1.1.11:8080/api/6/http/upstreams/bluegreen_bullet | jq .
```

