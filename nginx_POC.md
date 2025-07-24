
# NGINX POC for Uptycs Architecture - SRE Transition

## Objective

Demonstrate NGINX use cases simulating the role of an edge reverse proxy in Uptycs Cloud environment. This includes TLS termination, HTTP/S routing, load balancing, basic authentication, caching, and rate limiting.

---

## 🔧 Purpose in Architecture

In the Uptycs architecture, NGINX nodes are deployed outside the Kubernetes cluster on virtual machines. They handle incoming traffic and route requests to internal services such as:

- Agent ingestion
- UI dashboards
- API endpoints

---

## Setup Overview

### Step 1: Install NGINX

Install and enable NGINX as a systemd service on a Linux VM

```bash
sudo install nginx -y       # or apt install nginx -y
sudo systemctl enable nginx
sudo systemctl start nginx
```

![Screenshot](images/imgae_12.png)

Check configuration syntax before applying changes:

```bash
sudo nginx -t
```

---

### ✅ Step 2: Create Mock Backend Services

Simulate backend services using Flask:

| Service  | URL Path | Port  |
|----------|----------|-------|
| UI       | `/ui`    | 5001  |
| API      | `/api`   | 5002  |
| Login    | `/login` | 5003  |

Example Flask template:

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    return "Service is live."

if __name__ == '__main__':
    app.run(port=5001)  # Change port per service
```

---

### ✅ Step 3: Reverse Proxy Configuration

Update `/etc/nginx/nginx.conf` or your custom site config:

```nginx
location /ui {
    proxy_pass http://localhost:5001;
}

location /api {
    proxy_pass http://localhost:5002;
}

location /login {
    proxy_pass http://localhost:5003;
}
```

Validate with `curl http://<vm-ip>/ui`, `/api`, and `/login`.

---

### ✅ Step 4: Add Basic Auth to `/secure`

1. Install `htpasswd` utility:
   ```bash
   sudo yum install httpd-tools -y
   htpasswd -c /etc/nginx/.htpasswd testuser
   ```

2. Secure `/secure` route in NGINX:
   ```nginx
   location /secure {
       auth_basic "Restricted Access";
       auth_basic_user_file /etc/nginx/.htpasswd;
       proxy_pass http://localhost:5005;
   }
   ```

3. Update Flask service:
   ```python
   @app.route('/secure')
   def secure():
       return "Secure service is live."
   ```

4. Test:
   ```bash
   curl -u testuser http://<vm-ip>/secure
   ```

---

### ✅ Step 5: Add Rate Limiting

```nginx
limit_req_zone $binary_remote_addr zone=one:10m rate=10r/s;

location /api {
    limit_req zone=one burst=5 nodelay;
    proxy_pass http://localhost:5002;
}
```

Run stress test:

```bash
for i in {1..20}; do curl -s -o /dev/null -w "%{http_code}
" http://<vm-ip>/api; done
```

Expected: HTTP `200` initially, followed by `503` for rate-limited requests.

---

### ✅ Step 6: Load Balancing

Start two backend services on ports 6001 and 6002:

```nginx
upstream backend {
    server localhost:6001;
    server localhost:6002;
}

location /lb {
    proxy_pass http://backend;
}
```

Verify round-robin behavior using `curl` loop.

---

### ✅ Step 7: Caching

1. Add headers in Flask:

```python
@app.route('/secure')
def secure():
    return "Secure service with caching.", 200, {'Cache-Control': 'public, max-age=30'}
```

2. Add caching config in NGINX:

```nginx
proxy_cache_path /tmp/nginx_cache levels=1:2 keys_zone=my_cache:10m max_size=10m;

location /secure {
    proxy_cache my_cache;
    add_header X-Cache-Status $upstream_cache_status;
    proxy_pass http://localhost:5005;
}
```

---

## ✅ Final Outputs

- `nginx.conf` with full reverse proxy, auth, caching, rate limit, and load balancing
- Screenshots of mock service hits
- Sample `curl` outputs with headers and response codes
- `.htpasswd` file for secure endpoint
- Python files for mock services

---

## 📌 Notes

- This testbed simulates Uptycs' real NGINX role in handling edge traffic.
- It can be extended with TLS, monitoring (e.g., Prometheus + nginx-exporter), or Docker Compose for automation.
