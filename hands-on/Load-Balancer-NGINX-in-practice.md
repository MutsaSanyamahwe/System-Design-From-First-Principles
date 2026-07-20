# Hands-On: Load Balancing with NGINX

In this guide, you'll extend your NGINX reverse proxy into a load balancer that distributes traffic across multiple instances of the same Express application. By the end, you'll be able to prove — by watching logs in real time — that requests are actually being spread across different backend processes.

**What you'll build:**

```
                              ┌──> Express instance #1 (:3001)
Client --HTTPS--> NGINX  ---> ├──> Express instance #2 (:3002)
                              └──> Express instance #3 (:3003)
```

---

## Prerequisites

- Completion of the [Reverse Proxy with NGINX](./Reverse-Proxy-in-practice.md) guide, or an existing NGINX + Express setup
- Node.js and npm installed
- `sudo` access

---

## Step 1: Run multiple instances of the Express app

A load balancer only makes sense if there's more than one backend to balance across. Update your app so each instance can report which one handled a request — this makes it easy to visually confirm balancing is working.

`app.js`:

```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

app.get('/', (req, res) => {
  res.send(`Hello from Express instance running on port ${PORT}`);
});

app.listen(PORT, () => {
  console.log(`Express instance listening on port ${PORT}`);
});
```

Start three separate instances on three separate ports, each in its own terminal (or background process):

```bash
PORT=3001 node app.js
```
```bash
PORT=3002 node app.js
```
```bash
PORT=3003 node app.js
```

Confirm each one works independently:

```bash
curl http://localhost:3001
curl http://localhost:3002
curl http://localhost:3003
```

You should get a different port number back in each response.

> **Tip:** For anything beyond a quick test, run these under `pm2` so they survive terminal closure and restart on crash:
> ```bash
> npm install -g pm2
> pm2 start app.js --name api-1 -- --env PORT=3001
> pm2 start app.js --name api-2 -- --env PORT=3002
> pm2 start app.js --name api-3 -- --env PORT=3003
> ```

---

## Step 2: Define an upstream block in NGINX

NGINX groups backend servers using an `upstream` block, which you then reference from `proxy_pass` instead of pointing at a single server.

Edit your site config:

```bash
sudo nano /etc/nginx/sites-available/express-demo
```

Add an `upstream` block above the `server` block, and point `proxy_pass` at it:

```nginx
upstream express_backend {
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
}

server {
    listen 80;
    server_name your-domain.com;

    location / {
        proxy_pass http://express_backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Test and reload:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

By default, NGINX uses **round robin**: requests are handed to each server in the list in turn.

---

## Step 3: Verify the load balancing is actually happening

Send several requests in a row and watch the port number change:

```bash
for i in {1..6}; do curl -s http://your-domain.com; echo; done
```

Expected output — cycling through your three instances:

```
Hello from Express instance running on port 3001
Hello from Express instance running on port 3002
Hello from Express instance running on port 3003
Hello from Express instance running on port 3001
Hello from Express instance running on port 3002
Hello from Express instance running on port 3003
```

If you see the same port every time, double check that `nginx -t` reported no errors and that you reloaded (not just edited) the config.

---

## Step 4: Explore load balancing strategies

NGINX supports several algorithms, set as a directive at the top of the `upstream` block.

**Round robin (default)** — no directive needed, requests cycle evenly:
```nginx
upstream express_backend {
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
}
```

**Least connections** — sends the next request to whichever backend currently has the fewest active connections. Good when requests take variable time to process:
```nginx
upstream express_backend {
    least_conn;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
}
```

**IP hash** — routes a given client IP to the same backend consistently. Useful when your app keeps session state in memory on a specific instance:
```nginx
upstream express_backend {
    ip_hash;
    server 127.0.0.1:3001;
    server 127.0.0.1:3002;
    server 127.0.0.1:3003;
}
```

**Weighted round robin** — sends more traffic to servers you mark as more capable:
```nginx
upstream express_backend {
    server 127.0.0.1:3001 weight=3;
    server 127.0.0.1:3002 weight=1;
    server 127.0.0.1:3003 weight=1;
}
```
Here, instance 3001 receives roughly 3x the traffic of the other two.

After changing the strategy, always re-test:

```bash
sudo nginx -t && sudo systemctl reload nginx
```

---

## Step 5: Handle a backend going down

Load balancing should also mean resilience — if one instance crashes, traffic should route around it automatically. NGINX does this via passive health checks.

Stop one instance to simulate a crash:

```bash
# find and kill the process listening on 3002
kill $(lsof -t -i:3002)
```

Send more requests:

```bash
for i in {1..6}; do curl -s http://your-domain.com; echo; done
```

NGINX will detect the failed connection to port 3002, mark it temporarily unavailable, and route traffic only to the two healthy instances. You can tune this behavior explicitly:

```nginx
upstream express_backend {
    server 127.0.0.1:3001;
    server 127.0.0.1:3002 max_fails=3 fail_timeout=30s;
    server 127.0.0.1:3003;
}
```

- `max_fails=3` — mark the server down after 3 consecutive failed attempts
- `fail_timeout=30s` — how long to consider it down before retrying it

Restart the stopped instance and confirm it rejoins the rotation:

```bash
PORT=3002 node app.js
```

---

## Step 6: Confirm it end-to-end with logs

Watch the NGINX access log alongside your three app terminals to see requests being distributed live:

```bash
sudo tail -f /var/log/nginx/access.log
```

Each log line shows the client request; cross-reference the response body (the port number) against which terminal printed nothing new — since NGINX, not each app, is the one deciding where each request goes.

---

## Recap

-  Ran multiple instances of the same Express app on different ports
-  Grouped them into an NGINX `upstream` block
-  Verified round-robin distribution by watching responses cycle across ports
-  Compared load balancing strategies: round robin, least connections, IP hash, weighted
-  Configured passive health checks so a downed instance is automatically routed around
-  Confirmed the behavior live using logs from both NGINX and the app instances
