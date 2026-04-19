# MedNurex - India's Premium Medicine E-commerce Platform

![MedNurex](https://via.placeholder.com/1280x400/00a86b/ffffff?text=MedNurex+by+Ak+Sharma)

**30-Min Delivery • Prescription Verified • CDSCO Approved • Fully Secure HTTPS**

Built with ❤️ by **Ak Sharma**

## 🚀 Live Demo
**https://247awsspringbootintership.duckdns.org** (HTTPS Enabled)

---

## ✨ Production Features

- Beautiful Amazon-style UI
- WhatsApp + SMS OTP Login
- Razorpay UPI/Card/COD
- Secure HTTPS with Let’s Encrypt
- Dynamic DNS with DuckDNS
- Docker + Nginx Ready

---

## 📋 DNS & SSL Configuration (Production Setup)

### Why This Setup?
- Custom domain instead of raw IP
- HTTPS mandatory for security & browser trust
- Auto SSL renewal

### 1. DNS Setup - DuckDNS

**Domain Used:** `247awsspringbootintership.duckdns.org`

**Setup Steps:**
1. Go to [DuckDNS.org](https://www.duckdns.org)
2. Login with Google/DuckDNS account
3. Create subdomain: `247awsspringbootintership`
4. Set IP to your server’s public IP (EC2, VPS, etc.)
5. Install DuckDNS updater script on server for dynamic IP

**Verify:**
```bash
ping 247awsspringbootintership.duckdns.org
nslookup 247awsspringbootintership.duckdns.org

# Update system
sudo apt update && sudo apt upgrade -y

# Install Nginx
sudo apt install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx

# Install Certbot
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot

sudo certbot --nginx \
  -d 247awsspringbootintership.duckdns.org \
  --agree-tos \
  --redirect \
  -m your_email@example.com
server {
    listen 80;
    server_name 247awsspringbootintership.duckdns.org;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name 247awsspringbootintership.duckdns.org;

    ssl_certificate /etc/letsencrypt/live/247awsspringbootintership.duckdns.org/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/247awsspringbootintership.duckdns.org/privkey.pem;

    # Security Headers
    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Content-Type-Options "nosniff";

    # Proxy to Backend (Node.js)
    location / {
        proxy_pass http://localhost:3000;   # Next.js frontend
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /api/ {
        proxy_pass http://localhost:5000;   # Express backend
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}

sudo nginx -t
sudo systemctl reload nginx
sudo certbot renew --dry-run
version: '3.9'

services:
  frontend:
    build: ./apps/web
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production

  backend:
    build: ./backend
    ports:
      - "5000:5000"
    environment:
      - MONGO_URI=mongodb://mongodb:27017/mednurex
      - RAZORPAY_KEY_ID=...
      - JWT_SECRET=...

  mongodb:
    image: mongo:7
    ports:
      - "27017:27017"
    volumes:
      - mongodb_data:/data/db

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./certbot:/etc/letsencrypt
    depends_on:
      - frontend
      - backend

volumes:
  mongodb_data:

