## 🔹 POC 1: NGINX Nodes

### 🎯 Objective
Validate how ingress traffic is handled by NGINX nodes — including TLS termination, HTTP/S routing, and load balancing — within the Uptycs architecture.

---

### 🔧 Purpose in Architecture
NGINX nodes act as reverse proxies at the edge of the platform, managing all incoming traffic from external clients (agents, UI, APIs) and routing them to the appropriate backend services.

---

### 🧪 POC Tasks

1. **Deploy NGINX**
   - Use an Ubuntu/CentOS VM or container to install and run NGINX.
   - Configure basic reverse proxy rules for `/ui`, `/api`, and `/login`.

2. **Simulate Backend Services**
   - Mock services with different ports (e.g., Node.js or Python Flask apps simulating the login service, UI, and API).
   - Setup `upstream` blocks in NGINX config.

3. **Enable TLS Termination**
   - Generate a self-signed certificate or use Let’s Encrypt (for public lab).
   - Configure `ssl_certificate` and `ssl_certificate_key` in `nginx.conf`.

4. **Load Balancing**
   - Define multiple upstream backend nodes and enable round-robin or IP hash load balancing.

5. **Test Failure Handling**
   - Bring down a backend and observe how NGINX handles retries and timeouts.

---

### ✅ Success Criteria

- ✅ Incoming requests are routed correctly to mock services.
- ✅ TLS termination is successful, and HTTPS works.
- ✅ NGINX balances traffic across multiple upstream services.
- ✅ Downtime of one backend does not affect availability.
- ✅ Access logs show accurate source and request info.

---

### 📋 Output Artifacts

- `nginx.conf` with proxy and SSL config
- Screenshot of mock service dashboard/API served through NGINX
- Sample request/response headers captured via `curl` or browser dev tools
- Access log snippet showing traffic routing

---

### 📌 Notes

- This setup mirrors the real-world placement of NGINX in Uptycs as deployed outside Kubernetes on standalone VMs.
- For production-like simulation, integrate NGINX with Docker Compose and mock Uptycs components.

---

