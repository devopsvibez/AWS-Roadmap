# 🌐 Networking for DevOps – Real-World Troubleshooting Notes (2026)

This repository contains practical notes on how networking issues are
debugged in real DevOps and SRE work.

It is written to help you:
- understand where a problem is
- avoid random command execution
- debug issues step by step, the way it’s done in production

You can use this as:
- learning notes
- a debugging reference
- interview revision
- guidance on how to think during incidents

---

## 🧩 Before You Start

Most production systems run on **Linux**.  
Most learners use **Windows laptops**.

Because of this:
- Linux commands are shown as the main reference
- Windows equivalents are mentioned only where useful
- Git Bash is not treated as Linux

The focus is how things work **on servers**, not on local machines.

---

## How to Use This Guide

Networking issues usually show up as:
- timeouts
- connection refused errors
- 502 / 504 responses
- services that work internally but not externally

Each section below starts with **one simple question**.

Start from the top.  
Move step by step.  
When something fails, stop there.

That is usually the real problem.

---

## 1️⃣ Can I reach the machine at all?

### Linux / WSL / Servers
    ping example.com

What this does:
- sends small network packets to the target
- checks whether the target responds

If it works:
- the machine is reachable at a basic level

If it fails:
- network may be down
- routing may be broken
- DNS may be failing
- or ICMP (ping) may be blocked

Ping failing does not always mean the server is down.

### Windows
    ping example.com

---

## 2️⃣ Does the machine have an IP address?

### Linux / WSL / Servers
    ip addr

What this shows:
- network interfaces on the system
- IP addresses assigned to them

What to look for:
- an `inet` IP on an active interface

If there is no IP:
- nothing else will work
- the issue is at the network setup level

### Windows
    ipconfig

---

## 3️⃣ Can this machine reach outside networks?

### Linux / WSL / Servers
    ip route

What this shows:
- how the system decides where to send traffic

What to look for:
- a **default route**

If there is no default route:
- the machine cannot access external systems

### Windows
    route print

---

## 4️⃣ Is DNS resolving correctly?

### Linux / WSL / Servers
    nslookup example.com

What this does:
- asks the DNS server for the IP of a domain

If this fails:
- applications will not reach external services
- the issue is often DNS, not the app

---

    dig example.com

What this is:
- a detailed DNS query tool

Used for:
- checking which record is returned
- viewing TTL values
- debugging inconsistent DNS behaviour

---

    resolvectl status

What this shows:
- which DNS servers the system is using

Useful when:
- DNS works on one server but not another

### Windows
    nslookup example.com

---

## 5️⃣ Is the service actually running?

### Linux / WSL / Servers
    ss -tulnp

What the flags mean:
- `-t` → TCP ports
- `-u` → UDP ports
- `-l` → listening ports only
- `-n` → show numbers instead of names
- `-p` → show the process using the port

What to check:
- is your application port listed?

If the port is missing:
- the service is not running
- networking is not the issue yet

---

    netstat -tulnp

Older systems may still use this.

---

## 6️⃣ Is the port reachable?

### Linux / WSL / Servers
    nc -vz localhost 8080

What this command does:
- `nc` (netcat) tries to connect to a port
- `-v` → verbose output (shows what is happening)
- `-z` → zero-I/O mode (no data sent, just checks the port)

How to read the result:
- connected → port is open
- connection refused → service not listening
- timeout → firewall or security rule blocking it

---

## 7️⃣ Is the application responding correctly?

### Linux / WSL / Servers
    curl http://localhost:8080

What this does:
- sends a basic HTTP request
- checks whether the application responds

---

    curl -v https://example.com

What `-v` means:
- verbose output

This shows:
- DNS resolution
- connection details
- TLS handshake
- request and response headers

Very useful when requests hang or fail silently.

---

    curl -I https://example.com

What `-I` means:
- fetch headers only (no response body)

Useful for:
- checking HTTP status codes
- validating redirects
- verifying SSL configuration

---

## 8️⃣ Where does the network path break?

### Linux / WSL / Servers
    traceroute example.com

What this shows:
- each network hop between your machine and the destination

If it fails early:
- local network or gateway issue

If it fails later:
- upstream or provider issue

---

    tracepath example.com

Similar to traceroute, but works without root access.

### Windows
    tracert example.com

---

## 9️⃣ What is happening on the wire? (Advanced)

### Linux / WSL / Servers only
    tcpdump -i eth0

What this command does:
- captures network packets on an interface

What the flag means:
- `-i eth0` → listen on interface `eth0`

Use this only when:
- higher-level checks look fine
- the issue still exists
- you need packet-level evidence

This is not a beginner command and not a first step.

---

## 🔟 Is the firewall blocking traffic?

### Linux / WSL / Servers
    iptables -L -n

What this command does:
- lists firewall rules

What the flags mean:
- `-L` → list rules
- `-n` → show IPs and ports as numbers (no name resolution)

What to look for:
- DROP or REJECT rules
- unexpected blocks on required ports

---

    ufw status

What this shows:
- whether the Ubuntu firewall is enabled
- which ports are allowed or blocked

This is a very common cause of:
- services working locally
- but failing from outside

---

## Recommended Debugging Order

When something is not working, follow this order:

1. ping  
2. ip addr / ipconfig  
3. ip route  
4. DNS (nslookup / dig)  
5. ss / netstat  
6. nc  
7. curl  
8. traceroute  
9. tcpdump  

Skipping steps usually leads to wrong conclusions.

---

## A Simple Real-World Example

An application was returning 504 errors.

- ping worked  
- IP address was present  
- routing was fine  
- DNS resolution failed  

The application was not the problem.
DNS was misconfigured.

Fixing DNS resolved the issue.

---

## Final Notes

These commands behave the same whether you are:
- on a VM
- inside a container
- on a Kubernetes node

The surrounding tools may change,
but the networking basics remain the same.
Follow @Devopsvibez on instagram & subsribe to it on YouTube 💖
