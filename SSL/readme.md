# 🔐 SSL & Hosting

Simple reference for domain, hosting, SSL/TLS certificates, and HTTPS deployment.

---

## 🌐 1. Domain vs Hosting vs SSL

| Element     | Purpose                                             |
| ----------- | --------------------------------------------------- |
| **Domain**  | Address of the website (`example.com`)              |
| **Hosting** | Server where the application runs                   |
| **SSL/TLS** | Encrypts communication between users and the server |

Example:

```text
example.com
     │
     ▼
    DNS
     │
     ▼
   Server
     │
     ▼
   Nginx
     │
     ├── Frontend
     └── Backend/API
```

---

## 🔒 2. SSL/TLS

SSL is commonly used to refer to **TLS (Transport Layer Security)**.

It enables:

```text
http://example.com
```

to become:

```text
https://example.com
```

HTTPS provides:

- 🔐 Encryption

- 🛡️ Data integrity

- ✅ Server/domain authentication

> Use TLS 1.2 or TLS 1.3. Do not use old SSL/TLS protocols.

---

## 📜 3. Types of SSL Certificates

### Single Domain

Protects one domain.

```text
example.com
```

Good for:

- Portfolio

- Company website

- Simple application

---

### Wildcard

Protects a domain and its subdomains.

```text
*.example.com
```

Can cover:

```text
www.example.com
api.example.com
admin.example.com
app.example.com
```

A wildcard certificate does **not normally cover** the root domain:

```text
example.com
```

So check that the certificate also includes the root domain when needed.

---

### SAN / Multi-Domain

Allows several different domains/hostnames in one certificate.

```text
example.com
example.net
api.example.com
```

---

## 🏷️ 4. DV, OV and EV

| Type   | Description             |
| ------ | ----------------------- |
| **DV** | Domain Validation       |
| **OV** | Organization Validation |
| **EV** | Extended Validation     |

For most normal websites and applications, **DV is sufficient**.

---

## 🆓 5. Free SSL — Let's Encrypt

[Let's Encrypt](https://letsencrypt.org/)

Provides free publicly trusted certificates.

Advantages:

- Free

- Trusted by browsers

- Automatic renewal

- Works well with Nginx

- Supports wildcard certificates through DNS validation

For most personal projects and web applications:

```text
Let's Encrypt + Nginx + automatic renewal
```

is enough.

---

## 💳 6. Paid SSL Certificates

Commercial certificates can be useful when you need:

- OV/EV certificates

- Commercial support

- Enterprise requirements

- Specific CA requirements

### SSL Websites

- [CheapSSLsecurity](https://cheapsslsecurity.com/)

- [The SSL Store](https://www.thesslstore.com/)

- [ComodoSSLStore](https://comodosslstore.com/)

- [GoGetSSL](https://www.gogetssl.com/)

### Reviews

- [Trustpilot](https://www.trustpilot.com/)

> Compare the certificate type, domain coverage, renewal price, support, and CA before purchasing.

---

## 🌍 7. DNS

Before SSL installation, your domain must point to your server.

Example:

```text
example.com       A       SERVER_IP
www               CNAME   example.com
api               A       SERVER_IP
```

Typical architecture:

```text
example.com
     │
     ▼
    DNS
     │
     ▼
  Server IP
```

---

## 🚀 8. Typical Production Setup

```text
                    INTERNET
                        │
                        ▼
                  example.com
                        │
                       DNS
                        │
                        ▼
                      Nginx
                    HTTPS :443
                        │
             ┌──────────┴──────────┐
             ▼                     ▼
          Frontend              Backend
          Next.js             Spring Boot
                                  │
                                  ▼
                              Database
```

Nginx handles HTTPS and forwards requests to the application.

---

## 🔐 9. Basic SSL Best Practices

- Use HTTPS everywhere.

- Redirect HTTP → HTTPS.

- Use TLS 1.2 / TLS 1.3.

- Protect your private key.

- Never commit private keys to Git.

- Automate certificate renewal.

- Monitor certificate expiration.

- Keep the server and Nginx updated.

- Do not expose your database directly to the Internet.

- Use a firewall.

- Consider HSTS after HTTPS is correctly configured.

---

## 🔑 10. Important Certificate Files

You may receive:

```text
certificate.crt
fullchain.pem
privkey.pem
```

The most sensitive file is:

```text
privkey.pem
```

**Never expose or commit it to Git.**

Typical Nginx configuration:

```nginx
ssl_certificate     /path/to/fullchain.pem;
ssl_certificate_key /path/to/privkey.pem;
```

---

## 🔄 11. Certificate Renewal

Never rely on manual renewal.

Recommended:

```text
Certificate
     ↓
Automatic renewal
     ↓
Nginx reload
     ↓
Monitoring / alert
```

For Let's Encrypt, use an ACME client such as Certbot or another supported ACME client.

---

## ✅ Quick Checklist

```text
[ ] Domain purchased
[ ] DNS configured
[ ] Server/hosting configured
[ ] Nginx installed
[ ] SSL certificate obtained
[ ] HTTPS configured
[ ] HTTP → HTTPS redirect
[ ] Private key protected
[ ] Automatic renewal enabled
[ ] Certificate expiration monitored
[ ] Firewall configured
[ ] Database not publicly exposed
```

---

## ⭐ Simple Recommendation

For a normal web project:

```text
Domain
   +
VPS
   +
Nginx
   +
Let's Encrypt
   +
Automatic renewal
```

For a project requiring a commercial certificate:

```text
Domain
   +
VPS
   +
Nginx
   +
Commercial SSL
   +
Automatic renewal/monitoring
```

> **SSL is not only about buying a certificate. The important part is configuring HTTPS correctly and managing the certificate throughout its lifecycle.**
