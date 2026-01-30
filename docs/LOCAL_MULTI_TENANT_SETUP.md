# 🌐 Local Multi-Tenant Domain Setup

This project supports **subdomain-based multi-tenant routing** (e.g. `flyingcat.next`, `coolcars.next`).  
To test this locally, you must map custom domains to your local machine.

---

## 🛠 Step 1: Edit Your Hosts File

Your system’s **hosts file** tells your computer how to resolve domain names.

### 🍎 macOS / Linux

Open Terminal and run:

```bash
sudo vim /etc/hosts
```

Add these lines at the bottom of the file:

```
127.0.0.1   flyingcat.next
127.0.0.1   coolcars.next
```

Save the file and exit.

---

### 🪟 Windows

1. Open **Notepad as Administrator**
2. Open the file:

```
C:\Windows\System32\drivers\etc\hosts
```

3. Add the following lines at the bottom:

```
127.0.0.1   flyingcat.next
127.0.0.1   coolcars.next
```

4. Save the file

---

## 🔄 Step 2: Flush DNS Cache (If Needed)

If the domains don’t work right away, clear your DNS cache.

### macOS

```bash
sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder
```

### Windows

```bash
ipconfig /flushdns
```

---

## ▶ Step 3: Start the Development Server

```bash
pnpm dev
```

---

## 🌍 Step 4: Access Tenant Domains

Open these URLs in your browser:

| Tenant      | URL                          |
|------------|------------------------------|
| flyingcat  | http://flyingcat.next:3000   |
| coolcars   | http://coolcars.next:3000    |

Each domain will be treated as a **separate tenant** by the application.

---

## 🧠 Why This Setup Is Required

Browsers don’t automatically know that custom domains like `flyingcat.next` should point to your local computer.

The hosts file tells your operating system:

> "When this domain is requested, route it to my local machine (127.0.0.1)."

This allows you to test **real multi-tenant subdomain routing locally** without deploying to a live server.

---

## ✅ You’re Done

You can now develop and test tenant-specific behavior locally using real domain-based routing.
