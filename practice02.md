
## 📌 1. Linux Debugging – Application Not Accessible on Port 8080

### 🔹 Problem

Application is running but not accessible on port `8080`.

### ✅ Solution Steps

```bash
# 1. Check service status
systemctl status <service_name>

# 2. Check logs
journalctl -u <service_name> -f

# 3. Check if port is listening
ss -tulnp | grep 8080

# 4. Test locally
curl localhost:8080

# 5. Check firewall
ufw status
# or
iptables -L
```

### 💡 Key Insight

* Ensure app binds to `0.0.0.0` (not `127.0.0.1`)
* Verify cloud security group if applicable

---

## 📌 2. Linux Task – Compress Recent Log Files

### 🔹 Problem

Find `.log` files modified in last 24 hours and compress them.

### ✅ Solution

```bash
find /var/log -type f -name "*.log" -mtime -1 -print0 | tar -czvf logs.tar.gz --null -T -
```

### 💡 Alternative (simpler)

```bash
tar -czvf logs.tar.gz $(find /var/log -type f -name "*.log" -mtime -1)
```

---

## 📌 3. Shell Script – Service Monitor

### 🔹 Problem

* Check if a service is running
* If not, start it
* Log the action

### ✅ Solution

```bash
#!/bin/bash

SERVICE=$1
LOG_FILE="/var/log/service_monitor.log"

if [ -z "$SERVICE" ]; then
    echo "Usage: $0 <service-name>"
    exit 1
fi

if ! systemctl is-active --quiet "$SERVICE"; then
    sudo systemctl start "$SERVICE"
    echo "$(date) - Started service: $SERVICE" >> $LOG_FILE
else
    echo "$(date) - Service already running: $SERVICE" >> $LOG_FILE
fi
```

---

## 📌 4. Docker – Multi-Stage Build (Node.js App)

### 🔹 Problem

Create an optimized multi-stage Dockerfile.

### ✅ Solution

```dockerfile
# Stage 1: Build
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Stage 2: Production
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --omit=dev

COPY --from=builder /app/dist ./dist

ENV NODE_ENV=production

EXPOSE 3000

CMD ["node", "dist/index.js"]
```

---

## 📌 5. Docker Debugging – App Not Accessible

### 🔹 Problem

Container is running but app not accessible.

### ✅ Solution Steps

```bash
# 1. Check running containers
docker ps

# 2. Check logs
docker logs <container_id>

# 3. Verify port mapping
docker ps

# 4. Enter container
docker exec -it <container_id> sh

# 5. Check listening port
netstat -tulnp | grep 3000

# 6. Test inside container
curl localhost:3000

# 7. Test from host
curl localhost:8080
```

### 💡 Common Issues

* App bound to `localhost` instead of `0.0.0.0`
* Incorrect port mapping
* App crash (check logs)

---

## 📌 6. Linux Task – Top 5 Largest Files

### 🔹 Problem

Find top 5 largest files in `/var`.

### ✅ Solution

```bash
du -ah /var | sort -rh | head -n 5
```

### 💡 More Accurate (files only)

```bash
find /var -type f -exec du -h {} + | sort -rh | head -n 5
```

---

## 📌 7. Shell Task – Count IP Occurrences

### 🔹 Problem

Count occurrences of IP addresses in log file.

### ✅ Solution

```bash
awk '{print $1}' /var/log/app.log | sort | uniq -c | sort -nr
```

### 💡 Generic Version

```bash
grep -oE '([0-9]{1,3}\.){3}[0-9]{1,3}' /var/log/app.log | sort | uniq -c | sort -nr
```

---

## 📌 8. Docker – High CPU Usage Troubleshooting

### 🔹 Problem

Container consuming high CPU.

### ✅ Solution

```bash
# 1. Identify container
docker stats

# 2. Inspect container
docker inspect <container_id>

# 3. Limit CPU (new container)
docker run --cpus="1.0" my-app

# 4. Limit CPU (running container)
docker update --cpus="1.0" <container_id>
```

### 💡 Bonus

```bash
docker run --cpus="1.0" --memory="512m" my-app
```


### Question 9/10 — Networking 🌐

**Your application server can successfully connect to an external API by IP:**

curl http://10.20.30.40:8080

**But this fails:**

curl http://api.example.com:8080

The application itself is healthy.

**What could be causing this, and what commands would you use to troubleshoot it?**

**give me the interview answer**

```
Here’s how I’d answer this in an interview:

“Since the server can connect to the API using its IP address but not the hostname, I would first suspect a DNS resolution issue. I would verify whether api.example.com resolves correctly using commands like nslookup, dig, or getent hosts.

If DNS resolution fails, I would check the server's DNS configuration in /etc/resolv.conf and verify connectivity to the configured DNS server.

If the hostname resolves to an IP successfully, I would compare that IP with 10.20.30.40 and then test connectivity to the resolved address using curl, nc, or telnet. I would also check whether there is any proxy, firewall, or network policy affecting hostname-based access.

So my troubleshooting flow would be: DNS resolution → verify resolved IP → test connectivity → check firewall/proxy/network configuration → check application logs if necessary.”

Useful commands:

nslookup api.example.com
dig api.example.com
getent hosts api.example.com

cat /etc/resolv.conf

nc -zv api.example.com 8080
curl -v http://api.example.com:8080

Interview tip: The key observation is “IP works, hostname fails” → investigate DNS first.
```

#### One important correction: with a 502 Bad Gateway, Nginx itself is usually reachable and running; the more likely issue is that Nginx cannot communicate with its upstream backend.

A stronger troubleshooting flow:
```
nginx -t
systemctl status nginx
tail -f /var/log/nginx/error.log
```
Then check the upstream configuration:
```
grep -R "proxy_pass" /etc/nginx/
```
Verify the backend is actually running and listening:
```
ss -tulnp
systemctl status <backend-service>
curl http://127.0.0.1:<backend-port>
```
If the backend is on another server/container, test:

```
curl http://<backend-ip>:<port>
nc -zv <backend-ip> <port>
```

Then investigate:

Wrong proxy_pass address/port
Backend service down
Backend listening only on 127.0.0.1
Docker/Kubernetes networking/DNS
Firewall/security group
Backend timeout
Recent configuration/deployment changes

Your nc localhost 80 check isn't particularly useful here because Nginx is already confirmed to be running; I'd focus on the Nginx → upstream backend path.
