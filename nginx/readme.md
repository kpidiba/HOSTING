Absolutely. I’d simplify your original guide and reorganize it around **what you actually need to understand as a developer**, while keeping practical configurations you can copy when deploying Angular, Next.js, Laravel, or Spring Boot.

# 🐋 NGINX — Practical Developer Guide

> A practical guide to understanding and using NGINX with Angular, React, Next.js, Laravel, Spring Boot and APIs.

---

## 📖 1. What is NGINX?

**NGINX** (pronounced "engine-x") is a high-performance web server.

It can act as:

- 🌐 Web server

- 🔄 Reverse proxy

- ⚖️ Load balancer

- 🔒 HTTPS/SSL termination point

- 💾 HTTP cache

- 📦 Static file server

- 🛡️ Security layer in front of applications

The most important thing to understand as a developer is:

> **NGINX usually sits in front of your application and decides what to do with incoming HTTP requests.**

For example:

```text
                    Internet
                       │
                       │ HTTPS
                       ▼
                ┌─────────────┐
                │    NGINX    │
                └──────┬──────┘
                       │
          ┌────────────┼────────────┐
          │            │            │
          ▼            ▼            ▼
       Angular      Spring Boot   Laravel
       :4200          :8080       PHP-FPM
```

---

# 🌐 2. How a Web Request Reaches Your Application

Suppose a user opens:

```text
https://example.com
```

The simplified process is:

```text
Browser
   │
   │ HTTPS request
   ▼
DNS
   │
   │ example.com → server IP
   ▼
Server
   │
   ▼
NGINX
   │
   │ decides where request goes
   ▼
Application
   │
   ▼
Response
   │
   ▼
NGINX
   │
   ▼
Browser
```

NGINX is therefore often the **entry point** to your server.

---

# 🔄 3. What is a Reverse Proxy?

This is one of the most important NGINX concepts.

A **reverse proxy** is a server that receives requests from clients and forwards them to another application/server.

For example:

```text
Browser
   │
   │ GET /api/users
   ▼
NGINX
   │
   │ proxy_pass
   ▼
Spring Boot :8080
   │
   ▼
Database
```

The browser does not need to know that Spring Boot is running on port `8080`.

The user simply accesses:

```text
https://example.com/api/users
```

NGINX internally forwards the request to:

```text
http://localhost:8080
```

### Why use a reverse proxy?

It allows you to:

- hide internal application ports

- use domains and subdomains

- provide HTTPS

- route requests to different applications

- load balance multiple servers

- centralize security configuration

- serve static files efficiently

- handle compression and caching

- control access to applications

---

# 🧠 4. The Reverse Proxy Mental Model

Think of NGINX as a receptionist.

```text
                         NGINX
                      ┌─────────┐
                      │Reception│
                      └────┬────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
       Frontend           API             Admin
       Angular         Spring Boot       Laravel
        :4200             :8080           :8000
```

The browser says:

```text
GET /api/users
```

NGINX says:

> `/api/` belongs to the API. I'll forward it to Spring Boot.

The browser says:

```text
GET /
```

NGINX says:

> `/` belongs to the frontend. I'll serve the Angular application.

---

# 🏗️ 5. Typical Production Architecture

A common architecture looks like this:

```text
                         INTERNET
                            │
                            │ HTTPS :443
                            ▼
                    ┌─────────────────┐
                    │      NGINX      │
                    │ Reverse Proxy   │
                    │ SSL Termination │
                    └────────┬────────┘
                             │
             ┌───────────────┼────────────────┐
             │               │                │
             ▼               ▼                ▼
          Angular        Spring Boot       Laravel
          Static          REST API          PHP-FPM
           Files            :8080
             │               │                │
             └───────────────┼────────────────┘
                             │
                             ▼
                         Database
```

For a modern frontend + backend application:

```text
app.example.com
        │
        ▼
      NGINX
        │
        ▼
     Angular


api.example.com
        │
        ▼
      NGINX
        │
        ▼
   Spring Boot
```

---

# ⚙️ 6. NGINX Architecture

NGINX uses an event-driven architecture.

The main processes are:

### Master Process

Responsible for:

- reading configuration

- managing workers

- starting/stopping workers

- reloading configuration

### Worker Processes

Workers handle client requests.

Simplified:

```text
                 Master Process
                       │
            ┌──────────┼──────────┐
            ▼          ▼          ▼
         Worker     Worker      Worker
            │          │          │
            ▼          ▼          ▼
        Requests   Requests   Requests
```

You don't need to memorize the internal implementation.

Just remember:

> **Master manages NGINX. Workers handle requests.**

---

# 📁 7. NGINX Configuration Structure

On many Linux distributions:

```text
/etc/nginx/
├── nginx.conf
├── sites-available/
│   ├── example.com
│   └── api.example.com
├── sites-enabled/
│   ├── example.com -> ../sites-available/example.com
│   └── api.example.com -> ../sites-available/api.example.com
├── conf.d/
└── snippets/
```

### `nginx.conf`

Main NGINX configuration.

### `sites-available/`

Configurations that are available but not necessarily enabled.

### `sites-enabled/`

Configurations currently enabled.

Usually a symbolic link is created:

```bash
sudo ln -s /etc/nginx/sites-available/example.com \
          /etc/nginx/sites-enabled/example.com
```

---

# 🧩 8. Understanding the NGINX Configuration

A typical configuration:

```nginx
server {
    listen 80;

    server_name example.com;

    root /var/www/example;

    location / {
        ...
    }
}
```

The important concepts are:

```text
server
├── listen
├── server_name
├── root
└── location
```

---

# 🌐 9. `server {}`

A `server` block defines how NGINX should handle a particular website/domain.

Example:

```nginx
server {
    listen 80;
    server_name example.com;
}
```

Meaning:

> Listen on port 80 and handle requests for `example.com`.

You can have multiple websites:

```nginx
server {
    listen 80;
    server_name app.example.com;
}

server {
    listen 80;
    server_name api.example.com;
}

server {
    listen 80;
    server_name admin.example.com;
}
```

---

# 🔌 10. `listen`

Defines the port NGINX listens on.

HTTP:

```nginx
listen 80;
```

HTTPS:

```nginx
listen 443 ssl;
```

Common ports:

```text
80   → HTTP
443  → HTTPS
8080 → commonly used by backend applications
3000 → commonly used by Next.js
4200 → commonly used by Angular development server
```

---

# 🌍 11. `server_name`

Defines the domain handled by the server block.

```nginx
server_name example.com;
```

You can also specify multiple names:

```nginx
server_name example.com www.example.com;
```

For subdomains:

```nginx
server_name api.example.com;
```

---

# 📂 12. `root`

Defines the directory containing static files.

Example:

```nginx
root /var/www/my-app/dist;
```

If the browser requests:

```text
/assets/logo.png
```

NGINX may look for:

```text
/var/www/my-app/dist/assets/logo.png
```

---

# 📍 13. What is `$uri`?

`$uri` is an NGINX variable representing the **URI/path of the current request**.

For:

```text
https://example.com/products/123
```

the URI is:

```text
/products/123
```

Therefore:

```nginx
$uri
```

contains approximately:

```text
/products/123
```

Other useful NGINX variables include:

```nginx
$host
$request_uri
$remote_addr
$query_string
$scheme
```

For example:

```text
$host
→ example.com

$uri
→ /products/123

$query_string
→ page=2&sort=name

$scheme
→ https

$remote_addr
→ client IP address
```

---

# 🔎 14. Understanding `location`

`location` defines how NGINX handles requests matching a particular URL path.

Example:

```nginx
location /api/ {
    ...
}
```

This matches:

```text
/api/users
/api/products
/api/login
```

Another example:

```nginx
location / {
    ...
}
```

matches general requests.

You can therefore route different parts of your application:

```text
/api/      → Backend
/admin/    → Admin application
/          → Frontend
```

---

# 🔄 15. `proxy_pass`

`proxy_pass` forwards a request to another server/application.

Example:

```nginx
location /api/ {
    proxy_pass http://localhost:8080;
}
```

Architecture:

```text
Browser
   │
   │ /api/users
   ▼
 NGINX :80
   │
   │ proxy_pass
   ▼
Spring Boot :8080
```

Spring Boot is running internally on:

```text
localhost:8080
```

but the user accesses:

```text
https://example.com/api/users
```

---

# 📨 16. Proxy Headers

A reverse proxy may need to forward information about the original request.

Common configuration:

```nginx
location /api/ {
    proxy_pass http://localhost:8080;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

These headers help the backend know:

- original host

- client IP

- proxy chain

- original protocol (`http`/`https`)

You don't need to memorize every header.

Understand their purpose.

---

# 📦 17. `try_files`

`try_files` is particularly important for:

- Angular

- React

- Vue

- Laravel

Example:

```nginx
location / {
    try_files $uri $uri/ /index.html;
}
```

Conceptually:

```text
Request
   │
   ▼
Does the requested file exist?
   │
   ├── YES → serve it
   │
   ▼
Does the directory exist?
   │
   ├── YES → serve it
   │
   ▼
Fallback to index.html
```

---

# 🅰️ 18. Why Angular/React/Vue Need `try_files`

Suppose your Angular application contains:

```text
dist/
├── index.html
├── main.js
├── styles.css
└── assets/
```

The user visits:

```text
https://example.com/dashboard
```

There is no physical:

```text
dist/dashboard/
```

The Angular router handles `/dashboard`.

Therefore NGINX needs:

```nginx
try_files $uri $uri/ /index.html;
```

So:

```text
/dashboard
    ↓
NGINX
    ↓
No physical file
    ↓
index.html
    ↓
Angular Router
    ↓
Dashboard Component
```

Without this configuration, refreshing `/dashboard` can result in:

```text
404 Not Found
```

---

# 🅰️ 19. Angular Production Configuration

After:

```bash
ng build
```

you might have:

```text
dist/my-app/browser/
├── index.html
├── main.js
├── styles.css
└── assets/
```

NGINX:

```nginx
server {
    listen 80;

    server_name app.example.com;

    root /var/www/my-app/dist/my-app/browser;

    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

Architecture:

```text
Browser
   │
   ▼
NGINX
   │
   ▼
Angular static files
```

---

# ⚛️ 20. React Production Configuration

For a React SPA:

```nginx
server {
    listen 80;

    server_name app.example.com;

    root /var/www/my-react-app/build;

    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

The important part is:

```nginx
try_files $uri $uri/ /index.html;
```

---

# ▲ 21. Next.js Configuration

Next.js can run as a Node.js server.

For example:

```text
Next.js
   │
   ▼
localhost:3000
```

NGINX can act as a reverse proxy:

```nginx
server {
    listen 80;

    server_name example.com;

    location / {
        proxy_pass http://localhost:3000;

        proxy_http_version 1.1;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Architecture:

```text
Browser
   │
   ▼
NGINX :80/:443
   │
   ▼
Next.js :3000
```

---

# 🐘 22. Laravel + NGINX

Laravel is different because NGINX doesn't execute PHP itself.

The architecture is:

```text
Browser
   │
   ▼
NGINX
   │
   ▼
PHP-FPM
   │
   ▼
Laravel
   │
   ▼
Database
```

Example:

```nginx
server {
    listen 80;

    server_name example.com;

    root /var/www/my-laravel-app/public;

    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;

        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

Important:

> The PHP version in `fastcgi_pass` must match the PHP-FPM version installed on your server.

---

# 🧠 23. Why Laravel Uses `index.php`

Laravel is a framework with a central entry point.

For:

```text
/users
/products
/login
/dashboard
```

Laravel's routing system handles the request.

NGINX therefore sends requests to:

```text
index.php
```

using:

```nginx
try_files $uri $uri/ /index.php?$query_string;
```

Conceptually:

```text
GET /users
     │
     ▼
   NGINX
     │
     ▼
index.php
     │
     ▼
 Laravel Router
     │
     ▼
 Controller
     │
     ▼
 Response
```

---

# 🖥️ 24. Spring Boot + NGINX

Spring Boot normally runs as its own application server:

```text
Spring Boot :8080
```

NGINX can sit in front:

```nginx
server {
    listen 80;

    server_name api.example.com;

    location / {
        proxy_pass http://localhost:8080;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Architecture:

```text
Client
  │
  ▼
NGINX
  │
  ▼
Spring Boot :8080
  │
  ▼
Database
```

---

# 🌎 25. Subdomains

A common production architecture is:

```text
example.com
    │
    ▼
Frontend


api.example.com
    │
    ▼
Backend API


admin.example.com
    │
    ▼
Admin application
```

NGINX can handle all of them:

```nginx
server {
    listen 80;
    server_name example.com;

    root /var/www/frontend;
}

server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:8080;
    }
}

server {
    listen 80;
    server_name admin.example.com;

    root /var/www/admin;
}
```

---

# 🏠 26. Local Subdomains

For local development, you can map domains to your local machine using:

```text
/etc/hosts
```

Example:

```text
127.0.0.1 app.myproject.local
127.0.0.1 api.myproject.local
127.0.0.1 admin.myproject.local
```

Then:

```text
app.myproject.local
        ↓
Angular


api.myproject.local
        ↓
Spring Boot


admin.myproject.local
        ↓
Laravel
```

This is useful for reproducing a production-like environment locally.

---

# 🔒 27. HTTPS / SSL

In production, users should normally access:

```text
https://example.com
```

rather than:

```text
http://example.com
```

NGINX can terminate HTTPS:

```text
Browser
   │
   │ HTTPS
   ▼
NGINX
   │
   │ HTTP/internal HTTPS
   ▼
Application
```

Example:

```nginx
server {
    listen 443 ssl;

    server_name example.com;

    ssl_certificate /etc/nginx/ssl/example.crt;
    ssl_certificate_key /etc/nginx/ssl/example.key;

    ...
}
```

HTTP can redirect to HTTPS:

```nginx
server {
    listen 80;

    server_name example.com;

    return 301 https://example.com$request_uri;
}
```

In production, prefer a trusted certificate such as one obtained through Let's Encrypt rather than a self-signed certificate.

---

# 📦 28. Static Files and Caching

NGINX is very good at serving static files:

```text
.js
.css
.png
.jpg
.svg
.webp
```

You can configure browser caching:

```nginx
location ~* \.(js|css|png|jpg|jpeg|gif|svg|webp)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

This can reduce unnecessary requests.

However, caching should be configured according to your application's asset naming/versioning strategy.

---

# ⚖️ 29. Load Balancing

NGINX can distribute requests across multiple application instances.

For example:

```text
                    NGINX
                      │
             ┌────────┼────────┐
             │        │        │
             ▼        ▼        ▼
         Backend 1 Backend 2 Backend 3
          :8081     :8082     :8083
```

Configuration:

```nginx
upstream backend {
    server localhost:8081;
    server localhost:8082;
    server localhost:8083;
}

server {
    listen 80;

    location / {
        proxy_pass http://backend;
    }
}
```

NGINX can then distribute requests among the servers.

---

# 🛡️ 30. Security Role

NGINX can provide an additional security layer.

Examples:

- HTTPS termination

- request limits

- access restrictions

- hiding internal application ports

- blocking sensitive files

- security headers

- limiting request sizes

- rate limiting

- restricting specific IPs

Example:

```nginx
location ~ /\. {
    deny all;
}
```

This can prevent access to hidden files.

---

# 📊 31. Logs

NGINX usually provides two important logs.

### Access log

Records requests:

```text
GET /api/users
POST /login
GET /assets/main.js
```

### Error log

Records errors and configuration/runtime problems.

Typical locations:

```text
/var/log/nginx/access.log
/var/log/nginx/error.log
```

Follow logs in real time:

```bash
sudo tail -f /var/log/nginx/access.log
```

```bash
sudo tail -f /var/log/nginx/error.log
```

---

# 🐛 32. Debugging NGINX

The most important command:

```bash
sudo nginx -t
```

It checks the configuration syntax.

If successful, you can reload:

```bash
sudo systemctl reload nginx
```

Check status:

```bash
sudo systemctl status nginx
```

Restart:

```bash
sudo systemctl restart nginx
```

### Recommended workflow

After changing configuration:

```text
Edit configuration
       ↓
nginx -t
       ↓
Configuration valid?
       │
       ├── NO → fix error
       │
       └── YES
            ↓
     systemctl reload nginx
```

---

# 🔐 33. File Permissions

NGINX needs permission to read files it serves.

For Laravel, PHP also needs appropriate permissions for directories such as:

```text
storage/
bootstrap/cache/
```

Avoid blindly doing:

```bash
chmod -R 777
```

This is generally a bad practice.

Instead, understand:

- who owns the files

- which user NGINX/PHP-FPM runs as

- which directories need write access

- which files should remain read-only

---

# 🗂️ 34. Useful NGINX Commands

### Test configuration

```bash
sudo nginx -t
```

### Reload configuration

```bash
sudo systemctl reload nginx
```

### Restart

```bash
sudo systemctl restart nginx
```

### Start

```bash
sudo systemctl start nginx
```

### Stop

```bash
sudo systemctl stop nginx
```

### Check status

```bash
sudo systemctl status nginx
```

### View version

```bash
nginx -v
```

---

# 🔍 35. Common Problems

## 404 Not Found

Possible causes:

- wrong `root`

- missing `try_files`

- wrong SPA configuration

- wrong Laravel `public/` directory

- incorrect `location`

For Angular/React/Vue, check:

```nginx
try_files $uri $uri/ /index.html;
```

---

## 502 Bad Gateway

Usually means NGINX cannot communicate with the backend.

For example:

```nginx
proxy_pass http://localhost:8080;
```

but Spring Boot isn't running.

Check:

```bash
sudo systemctl status nginx
```

and verify the application:

```bash
curl http://localhost:8080
```

Also check:

```bash
sudo tail -f /var/log/nginx/error.log
```

---

## 403 Forbidden

Possible causes:

- permissions

- incorrect `root`

- directory access

- missing index file

- NGINX access restrictions

---

## 500 Internal Server Error

The problem may be in the application rather than NGINX.

For Laravel, check:

```text
storage/logs/laravel.log
```

For Spring Boot, check application logs.

---

# 🧠 36. What You Should Actually Memorize

**Do NOT memorize complete NGINX configuration files.**

Memorize the concepts:

| Concept        | Meaning                                     |
| -------------- | ------------------------------------------- |
| NGINX          | Web server / reverse proxy                  |
| Reverse proxy  | Receives requests and forwards them         |
| `server`       | Defines a website/server configuration      |
| `listen`       | Port NGINX listens on                       |
| `server_name`  | Domain handled by the server                |
| `root`         | Directory containing static files           |
| `location`     | URL matching/routing rule                   |
| `proxy_pass`   | Forward request to another server           |
| `try_files`    | Try files, then use fallback                |
| `$uri`         | Current request path                        |
| `index`        | Default document                            |
| `fastcgi_pass` | Sends PHP requests to PHP-FPM               |
| `upstream`     | Defines backend servers                     |
| `ssl`          | HTTPS configuration                         |
| `access_log`   | Request logs                                |
| `error_log`    | Error logs                                  |
| `nginx -t`     | Test configuration                          |
| `reload`       | Reload configuration without stopping NGINX |

---

# 🎯 37. What You DON'T Need to Memorize

You do **not** need to memorize:

```nginx
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
```

or:

```nginx
fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
```

or every possible NGINX directive.

Instead, understand:

> "I need to configure WebSockets."

Then look up the appropriate configuration.

> "I need to connect NGINX to PHP-FPM."

Then look up the appropriate `fastcgi_pass` configuration.

> "I need to proxy Spring Boot."

Then use `proxy_pass`.

This is normal professional practice.

---

# 🧭 38. What You Should Learn in Order

Don't try to learn all NGINX directives at once.

Follow this order:

```text
1. HTTP basics
       ↓
2. Domain → DNS → IP → Port
       ↓
3. What a web server does
       ↓
4. NGINX
       ↓
5. Reverse proxy
       ↓
6. server {}
       ↓
7. location {}
       ↓
8. proxy_pass
       ↓
9. root + static files
       ↓
10. try_files + $uri
       ↓
11. Angular/React SPA deployment
       ↓
12. Laravel + PHP-FPM
       ↓
13. Spring Boot reverse proxy
       ↓
14. Next.js reverse proxy
       ↓
15. HTTPS
       ↓
16. Logs + debugging
       ↓
17. Load balancing
       ↓
18. Caching / performance
       ↓
19. Security
```

---

# 🧩 39. The Most Important Mental Model

If you remember only one diagram, remember this:

```text
                         INTERNET
                            │
                            │ HTTPS
                            ▼
                    ┌───────────────┐
                    │     NGINX     │
                    │               │
                    │ "Where should │
                    │  this request │
                    │     go?"      │
                    └───────┬───────┘
                            │
             ┌──────────────┼───────────────┐
             │              │               │
             ▼              ▼               ▼
          Angular       Spring Boot      Laravel
         Static Files      API            PHP-FPM
             │              │               │
             └──────────────┼───────────────┘
                            │
                            ▼
                         Database
```

NGINX is essentially the **front door**.

The applications are behind the door.

---

# 🚀 40. Real-World Example

Suppose you have:

```text
Frontend:
Angular

Backend:
Spring Boot

Database:
PostgreSQL
```

Your production architecture could be:

```text
                     INTERNET
                         │
                         ▼
               https://myapp.com
                         │
                         ▼
                   ┌─────────┐
                   │  NGINX  │
                   └────┬────┘
                        │
             ┌──────────┴──────────┐
             │                     │
             ▼                     ▼
          Frontend                API
          Angular             Spring Boot
                                :8080
                                   │
                                   ▼
                              PostgreSQL
```

NGINX configuration conceptually:

```nginx
server {
    listen 80;

    server_name myapp.com;

    root /var/www/myapp/dist;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location /api/ {
        proxy_pass http://localhost:8080;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Now:

```text
GET /
        ↓
NGINX
        ↓
Angular


GET /dashboard
        ↓
NGINX
        ↓
index.html
        ↓
Angular Router


GET /api/users
        ↓
NGINX
        ↓
Spring Boot :8080


GET /assets/logo.png
        ↓
NGINX
        ↓
Static file
```

That's the kind of configuration you should be able to **read and understand**, even if you don't remember every line.

---

# 🏆 Final Takeaway

You don't need to memorize NGINX.

You need to be able to answer these questions:

### 1. What is NGINX?

> A web server and reverse proxy commonly placed in front of applications.

### 2. What is a reverse proxy?

> A server that receives client requests and forwards them to backend servers/applications.

### 3. What is `location`?

> A rule that determines how requests matching a URL path are handled.

### 4. What is `proxy_pass`?

> It forwards a request to another application/server.

### 5. What is `$uri`?

> The path of the current request.

### 6. What is `try_files`?

> It tries to find a requested file/directory and provides a fallback when it doesn't exist.

### 7. Why does Angular need `/index.html` fallback?

> Because Angular routing happens in the browser, not through physical directories on the server.

### 8. Why does Laravel use `index.php`?

> Laravel uses a front controller, so requests are passed through `index.php` and Laravel's router handles them.

### 9. Why use NGINX with Spring Boot?

> To expose Spring Boot through a domain/HTTPS and act as a reverse proxy.

### 10. What should you do after modifying NGINX?

```bash
sudo nginx -t
sudo systemctl reload nginx
```

**That's the NGINX knowledge level you should aim for as an application developer.**

The configuration syntax is a reference you can look up. **Understanding the architecture is what you should remember.**
