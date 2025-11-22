# Web-App-Deployment-with-Load-Balancing-Monitoring

A highly available and production-ready **Web Application Architecture** deployed on **Microsoft Azure**, featuring **application gateway, health monitoring, observability, and fault-tolerance** using Azure App Service and Application Gateway.

---

## ✨ Key Features

| Capability | Description |
|----------|-------------|
| 🔁 High Availability | Two redundant App Service instances deployed in active–active mode |
| ⚖️ Layer 7 Load Balancing | Azure Application Gateway distributing traffic via Round-Robin |
| 🩺 Self-Healing | Custom health probes automatically remove unhealthy backends |
| 🔐 Network Security | App Services securely integrated with VNet (no public inbound) |
| 📊 Monitoring & Alerts | Full observability using Log Analytics Workspace + KQL |
| 🧩 Deployment | Simple code deployment using Local Git / Azure Portal |

> **Frontend instances are visually distinguished:**  
> **Blue** = `app-frontend-01` • **Orange** = `app-frontend-02`

---

## 🛠 Tech Stack

| Component | Service |
|----------|---------|
| Cloud Provider | Microsoft Azure |
| Compute | Azure App Service (S1 Plan) |
| Load Balancer | Application Gateway v2 |
| Networking | Azure Virtual Network (VNet) |
| Monitoring | Azure Monitor + Log Analytics + KQL |
| Deployment | Local Git / Azure Portal |

---

## 🧱 Deployment & Configuration Steps

### **📌 Phase 1 — Compute & Content**
- Created two App Services: `app-frontend-01` & `app-frontend02`
- Upload different `index.html` files to verify load balancing:
  - Blue background (Instance 01)
  - Orange background (Instance 02)

### **📌 Phase 2 — Networking**
- Created **VNet: `web-app-vnet`**
  - Subnet 1: `ag-subnet` → for Application Gateway
  - Subnet 2: `app-service-subnet` → for App Services
- Enabled **VNet Integration** for both App Services

### **📌 Phase 3 — Load Balancer Configuration**
- Provisioned **Application Gateway v2** with Public IP
- Configured:
  - Backend pool → both App Services
  - HTTP Settings:
    - ❌ Cookie-based affinity disabled
    - ✔ Host header = *Pick host name from backend target*
  - Custom Health Probe:
    - Path = `/index.html` (Fixes 404 errors)
- Result: Round-Robin load balancing & real-time health evaluation

### **📌 Phase 4 — Observability**
| Resource | Purpose |
|---------|----------|
| Log Analytics Workspace | Central logging & query insights |
| Diagnostic Settings | Application Gateway & App Services |
| Alerts | CPU > 80% & High Latency (> 30s) |

---

## 🧪 Testing & Validation

### **1️⃣ Load Balancing Test**
1. Open the Application Gateway Public IP  
2. Observe **Blue** or **Orange** page  
3. Refresh / Open in Incognito → Page switches (Round-Robin)

### **2️⃣ Log Analytics Queries (KQL)**

📌 **App Service Traffic & Errors**
```kql
AppServiceHTTPLogs
| where TimeGenerated > ago(24h)
| project TimeGenerated, CsUriStem, ScStatus, CsHost
| summarize count() by ScStatus

```
📌 **Application Gateway Response Time**
```kql
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.NETWORK" and Category == "ApplicationGatewayAccessLog"
| summarize AverageResponseTime = avg(clientResponseTime) by requestUri_s
```

📌 **Shows AlwaysOn health-check pings**
```kql
AppServiceHTTPLogs
| where TimeGenerated > ago(1h)
| take 100
```

**Troubleshooting Experience (Lessons Learned)**

| Issue Encountered                | Resolution                                                                 |
| -------------------------------- | -------------------------------------------------------------------------- |
| Health Probe returning `404`     | Updated path to `/index.html`                                              |
| Host header mismatch             | Enabled *Pick host name from backend target*                               |
| Browser caching affected results | Verified via Incognito / Hard Refresh                                      |
| Alerts not triggered initially   | Scoped CPU alert to **App Service Plan** instead of individual App Service |


