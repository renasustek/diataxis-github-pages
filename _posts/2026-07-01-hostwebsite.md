---
title: "From Domain to Inbox: A Complete Guide to Hosting on DigitalOcean, Nginx, and Sending Newsletters"
date: 2026-07-01
layout: post
---

Building a web presence is more than just writing code. It involves connecting a global web of domains, servers, and email protocols. 
If you are setting up a website on DigitalOcean using Nginx and planning to send newsletters via a service like Resend, 
understanding how these pieces fit together is essential.

This guide walks you through the entire workflow: from purchasing a domain to securing it with SSL, a
nd finally, ensuring your emails actually land in your users' inboxes.

## 1. The Foundation: Domains and DNS

### What Does "Owning" a Domain Mean?
Technically, you don't "buy" a domain permanently, you lease the exclusive right to use it (usually for 1 to 10 years) 
from a Domain Registrar, such as Namecheap. Owning a domain simply means you have registered a custom, 
human-readable address. By itself, it doesn't include a website it just gives you the address.

### The Domain Name System (DNS)
Computers communicate using IP addresses , whilst humans use words. DNS is the internet's GPS. When 
someone types your domain into their browser, the DNS translates it into the machine-readable IP address of the server where your website lives.

### Name Servers and the Chain of Command
When you register a domain, your registrar gives you default Name Servers. Name Servers act as the master directories for your domain.

If you are hosting on DigitalOcean, you must log into your registrar
and change the Name Servers to point to DigitalOcean. You must do this at the registrar because 
they hold the "Master Keys". The internet's highest authorities only recognise the registrar as your authorised 
representative. Once you point the Name Servers to DigitalOcean, Namecheap’s job in routing traffic is completely finished.

*Note on DNS Propagation:* When you change Name Servers, it doesn't update globally in an instant.
Internet Service Providers (ISPs) worldwide cache DNS records. Propagation is the waiting period 
(usually a few minutes to a few hours) for these servers to clear their old memories and fetch the new instructions.

### DNS Records: The Filing Cabinet
Once DigitalOcean is your Name Server, you will use it to manage your DNS Records—specific, line-by-line instructions for routing traffic:
* **A Record (Address Record):** Points your domain directly to the IP address of your DigitalOcean Droplet.
* **CNAME (Canonical Name):** Points a subdomain (like `www.yourdomain.com`) to your main domain.
* **MX (Mail Exchanger):** Tells other email providers where to deliver emails sent to `@yourdomain.com`.
* **TXT (Text Record):** Used to attach readable notes to your domain, primarily for verifying ownership and enforcing email security.

## 2. Hosting the Website: DigitalOcean and Nginx

With your DNS routing traffic to DigitalOcean, you need a place for that traffic to land. This is your **Droplet** (a virtual server). On this server, you will run **Nginx**, a highly efficient web server.

If the Droplet is the plot of land, Nginx is the traffic director at the front door. 
It receives the incoming requests from the browser, finds the correct HTML files or web applications on your server,
and serves them back to the user.

### Workflow: A User Visits Your Website
1.  **The Browser:** A user types your URL.
2.  **The ISP Resolver (The Local Library):** The user's computer asks their ISP for the IP address.
3.  **The Global Phonebook:** The ISP asks the global root servers, which point to the `.com` registry, which checks your registrar. The registrar says, "Go ask DigitalOcean's Name Servers."
4.  **DigitalOcean Name Servers:** The ISP arrives at DigitalOcean. DigitalOcean looks at your **A Record** and provides the IP address of your Droplet.
5.  **The Droplet & Nginx:** The browser connects directly to your Droplet's IP. Nginx processes the request and securely serves your website files.

## 3. Securing the Connection: SSL/TLS

Whenever you see `https://` or a padlock icon, you are seeing TLS (Transport Layer Security, the modern replacement for SSL) in action. If DNS is the GPS, the SSL/TLS Certificate is the armoured truck securely transporting data.

An SSL certificate provides **Authentication** (proving your server is who it claims to be) and **Encryption** (scrambling data so it cannot be intercepted).

### The TLS Handshake
This happens in milliseconds before a webpage loads, using both Asymmetric (two keys) and Symmetric (one shared key) encryption:
1.  **The "Hello":** The browser connects and lists its supported encryption types.
2.  **The ID Check:** Nginx replies with the chosen encryption type, the SSL Certificate, and the server's Public Key.
3.  **Verification:** The browser checks with a Certificate Authority to ensure the certificate is valid and trusted.
4.  **The Secret Handshake:** The browser creates a temporary, super-fast "Session Key", locks it using the server's Public Key, and sends it back.
5.  **Secure Connection:** The server unlocks the Session Key using its hidden Private Key. Both parties now use this shared Session Key to encrypt and
6.  decrypt all website data quickly and securely.

## 4. Setting up the Newsletter: Resend and Email Security

Email and DNS are permanently linked for two reasons: **Flexibility** 
(so you can host your website on DigitalOcean but let a specialised service handle emails) and **Security** (to stop spam).

To use a service like Resend, you must add three crucial TXT records to your DigitalOcean DNS:

* **SPF (Sender Policy Framework) - The Guest List:** A record listing the exact IP addresses (like Resend) authorised to send mail on behalf of your domain.
* **DKIM (DomainKeys Identified Mail) - The Wax Seal:** A digital signature attached to your emails proving they weren't tampered with in transit.
* **DMARC (Domain-based Message Authentication, Reporting, and Conformance) - The Bouncer's Instructions:** Tells receiving servers (like Gmail) to reject any email that fails SPF or DKIM checks.

### Workflow: Sending a Newsletter via Resend
1.  **The Resend Server:** You hit send. Resend constructs the email, stamps it with a DKIM digital signature, and sends it to the recipient's provider (e.g., Gmail).
2.  **The Gatekeeper:** Gmail halts the email. It sees it claims to be from `you@yourdomain.com` but physically originated from Resend.
3.  **DNS Interrogation:** Gmail checks your DigitalOcean Name Servers for your SPF, DKIM, and DMARC records.
4.  **The Verification Match:**
    * *SPF Check:* DigitalOcean confirms Resend is authorised.
    * *DKIM Check:* Gmail uses your public key to unlock the wax seal, proving the message wasn't altered.
5.  **The Inbox Delivery:** Because the security checks passed, Gmail's DMARC filter gives the green light. Your newsletter bypasses the spam folder and lands in the primary inbox.
