# Hands-On: Reverse Proxy with NGINX

In this guide, we'll set up NGINX as a reverse proxy in front of a simple Express application. By the end, you'll have traffic flowing through NGINX to your app, and you'll understand and be able to see exactly how each request travels through the proxy. You'll also enable HTTPS so the connection between clients and NGINX is encrypted.

**What we'll build:**

```
Client (browser) --HTTPS--> NGINX (reverse proxy) --HTTP--> Express app (localhost:3000)
```

---

## Prerequisites

- A Linux server or VM (Ubuntu/Debian commands shown below; adjust for your distro)
- Node.js and npm installed
- Basic comfort with the terminal
- `sudo` access

---

## Step 1: Install NGINX

```bash
sudo apt update
sudo apt install nginx -y
```

Check that it's running:

```bash
sudo systemctl status nginx
```

Visit `http://your-server-ip` in a browser — you should see the NGINX welcome page. That confirms NGINX itself is working before we touch any configuration.

---

## Step 2: Create a simple Express application

Create a project folder and minimal app so we have something to route traffic to.

```bash
mkdir ~/express-demo && cd ~/express-demo
npm init -y
npm install express
```

Create `app.js`:

```javascript
const express = require('express');
cpnst app = express();
cpnst PORT = 3000;


app.get('/', (req, res) => {
cpnsole.log(`Express app listening on port ${PORT})`;
});
```

Run it:

```bash
node app.js
```

Test locally on the server:

```bash
curl http://localhost:3000
```

You should see the "Hello from Express..." message. Leave this running (or use `pm2`/`nohup` to keep it alive), and open a new terminal for the next steps.
 
---

## Step 3: Configure NGINX as a reverse proxy

NGINX's site configs live in `/etc/nginx/sites-available/`, and are enabled by linking them into `/etc/nginx/sites-enabled/`.

Create a new config file:

 
```bash
sudo nano /etc/nginx/sites-available/express-demo
```

Add the following:

```nginx
server {
    listen 80;
    server_name your-domain.com;  # or your server's IP address
 
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```


**What these directives do:**
 
| Directive | Purpose |
|---|---|
| `proxy_pass` | Forwards the request to your Express app on port 3000 |
| `proxy_set_header Host $host` | Preserves the original `Host` header so Express sees the real domain |
| `proxy_set_header X-Real-IP` | Passes the client's real IP address to Express |
| `proxy_set_header X-Forwarded-For` | Appends a chain of proxy IPs, useful for logging |
| `proxy_set_header X-Forwarded-Proto` | Tells Express whether the original request was HTTP or HTTPS |
| `proxy_http_version 1.1` + `Upgrade`/`Connection` | Needed if you later add WebSocket support |
 
Enable the site and remove the default one:
 
```bash
sudo ln -s /etc/nginx/sites-available/express-demo /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
```

Test the config for syntax errors, then reload:

 
```bash
sudo nginx -t
sudo systemctl reload nginx
```

Now visit `http://your-domain.com` (or your server IP) — you should see the Express response, but this time it came through NGINX on port 80 instead of directly hitting port 3000.
 
---

 
## Step 4: Enable HTTPS

The easiest way to get a free, auto-renewing certificate is Certbot with Let's Encrypt (requires a real domain name pointed at your server; this won't work with a bare IP address).

Install Certbot:

```bash
sudo apt install certbot python3-certbot-nginx -y
```

Run it:
 
```bash
sudo certbot --nginx -d your-domain.com
```

Certbot will:

1. Verify you control the domain
2. Obtain a certificate
3. Automatically edit your NGINX config to add a `listen 433 ssl` block and redirect HTTP -> HTTPS

Your config will now look something like:

```nginx
server {
    listen 443 ssl;
    server_name your-domain.com;
 
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
 
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
 
server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$host$request_uri;
}
```

Certificates are renewed automatically via a system timer; you can verify with:

```bash
sudo certbot renew --dry-run
```
 
---

## Step 5: Understand how a request flows through the proxy

Here's the full journey of a single request, now that HTTPS and the proxy are in place:

1. **Client** sends `https://your-domain.com/` — TLS handshake happens with NGINX, not Express.
2. NGINX terminates SSL (decrypts the request), matches the `server-name` and `location` block.
3. **NGINX** rewrites headers (`Host`, `X-Real-IP`, `X-Forwarded-For`, `X-Forwarded-Proto`) and forwards the plain HTTP request to `localhost:3000`.
4. **Express** receives the request as if it were a normal local HTTP request, processes it, and returns a response.
5. **NGINX** takes that response, re-encrypts it over TLS, and sends it back to the client.

From the client's perspective, everything happened over one secure HTTPS connection. From Express's perspective, it only ever spoke plain HTTP to localhost. NGINX is the only thing that knows both sides.

You can watch this happen live by tailing both logs in separate terminals:
 
```bash
sudo tail -f /var/log/nginx/access.log
```
 
```bash
node app.js   # watch console.log output or add request logging middleware
```

Make a request and watch the entry appear in NGINX's log first, then observe your Express app process it — confirming the traffic really is flowing through the proxy rather than hitting Express directly.

---

## Recap

-  Installed NGINX and confirmed it serves traffic
-  Built a minimal Express app listening on port 3000
-  Configured NGINX to reverse-proxy requests to that app, preserving important headers
-  Enabled HTTPS with Certbot/Let's Encrypt and set up automatic HTTP→HTTPS redirection
-  Traced the full request/response path through the proxy
**Next steps to explore on your own:** load balancing across multiple Express instances, caching static responses at the NGINX layer, and rate limiting with `limit_req`.






