# UFW Firewall Documentation and Script Automation

- [UFW Firewall Documentation and Script Automation](#ufw-firewall-documentation-and-script-automation)
  - [Introduction](#introduction)
  - [Enabling the Firewall](#enabling-the-firewall)
  - [UFW DynDNS Automation Script](#ufw-dyndns-automation-script)
- [UFW – Uncomplicated Firewall Command Reference](#ufw--uncomplicated-firewall-command-reference)
  - [1. Basic UFW Commands](#1-basic-ufw-commands)
    - [Enabling / Disabling the Firewall](#enabling--disabling-the-firewall)
  - [2. Allow / Deny Rules](#2-allow--deny-rules)
    - [Managing Access by Port, Service, or IP](#managing-access-by-port-service-or-ip)
  - [3. Application Profiles](#3-application-profiles)
    - [Using Predefined Service Rules](#using-predefined-service-rules)
  - [4. Logging \& Monitoring](#4-logging--monitoring)
    - [Tracking Firewall Activity](#tracking-firewall-activity)
  - [5. Advanced Rules](#5-advanced-rules)
    - [Adding More Specific Access Rules](#adding-more-specific-access-rules)
  - [6. IPv6 Considerations](#6-ipv6-considerations)
    - [Working with IPv6 Traffic](#working-with-ipv6-traffic)
  - [7. Common Use Cases](#7-common-use-cases)

## Introduction

This document provides a comprehensive guide for managing your firewall using UFW (Uncomplicated Firewall). It covers:

- Activating the firewall and setting initial rules.
- A custom Bash script that dynamically updates firewall rules based on DNS resolution.
- How to schedule the script using cron.
- A reference of common UFW commands.

---

## Enabling the Firewall

Before proceeding, ensure that the firewall is active and configured correctly:

```bash
sudo ufw enable
sudo ufw status
````

Sample Rules
```bash
sudo ufw allow from 210.211.212.213
sudo ufw allow from 210.211.212.213 proto tcp to any port 22
```

Show active rules
```bash
sudo ufw status numbered
```

Delete Rule
```bash
sudo ufw delete 1
```

## UFW DynDNS Automation Script

```bash
#!/bin/bash

HOST="support.domain.com"
PORT=22

# Alte Regeln entfernen (optional – vorsichtig, könnte andere Regeln betreffen)
for i in $(sudo ufw status numbered | grep '22/tcp' | awk -F'[][]' '{print $2}' | sort -rn); do yes y | sudo ufw delete "$i"; done

# IPs vom Host auflösen (nur IPv4)
ips=$(dig +short "$HOST" | grep -E '^[0-9]+\.[0-9]+\.[0-9]+\.[0-9]+$')

if [[ -z "$ips" ]]; then
    echo "❌ Keine gültigen IPs für $HOST gefunden."
    exit 1
fi

for ip in $ips; do
    echo "➡️ Erlaube Zugriff von $ip auf Port $PORT"
    ufw allow from "$ip" to any port "$PORT" proto tcp
done
```

```bash
sudo crontab -e
```

```bash
*/1 * * * * sudo /home/ubuntu/ufw-dns-host.sh
```

# UFW – Uncomplicated Firewall Command Reference

## 1. Basic UFW Commands

### Enabling / Disabling the Firewall

| Command                     | Description                                 |
|-----------------------------|---------------------------------------------|
| `sudo ufw enable`           | Enable UFW and start enforcing rules        |
| `sudo ufw disable`          | Disable UFW (rules are not enforced)        |
| `sudo ufw status`           | Show current status and rules               |
| `sudo ufw status verbose`   | Show detailed status with rule policies     |
| `sudo ufw reload`           | Reload UFW to apply recent changes          |
| `sudo ufw reset`            | Reset all UFW rules to default              |

---

## 2. Allow / Deny Rules

### Managing Access by Port, Service, or IP

| Command                                       | Description                                      |
|-----------------------------------------------|--------------------------------------------------|
| `sudo ufw allow <port>`                       | Allow incoming traffic on specified port         |
| `sudo ufw allow <port>/<protocol>`            | Allow TCP/UDP traffic (e.g., `80/tcp`)           |
| `sudo ufw allow from <IP>`                    | Allow traffic from a specific IP address         |
| `sudo ufw allow from <IP> to any port <port>` | Allow IP to access specific port                 |
| `sudo ufw allow <service>`                    | Allow traffic for a known service (e.g. `ssh`)   |
| `sudo ufw deny <port>`                        | Deny traffic on a specific port                  |
| `sudo ufw delete allow <port>`                | Remove allow rule for specific port              |

---

## 3. Application Profiles

### Using Predefined Service Rules

| Command                          | Description                                      |
|----------------------------------|--------------------------------------------------|
| `sudo ufw app list`              | List available application profiles              |
| `sudo ufw app info <name>`       | Show details about a specific application profile|
| `sudo ufw allow <app-name>`      | Allow traffic for the app profile                |

---

## 4. Logging & Monitoring

### Tracking Firewall Activity

| Command                                | Description                                     |
|----------------------------------------|-------------------------------------------------|
| `sudo ufw logging on`                  | Enable UFW logging                              |
| `sudo ufw logging off`                 | Disable UFW logging                             |
| `sudo ufw logging low`                 | Set logging level to low                        |
| `sudo ufw logging medium`              | Set logging level to medium                     |
| `sudo ufw logging high`                | Set logging level to high                       |
| `sudo ufw logging full`                | Set logging level to full                       |
| `sudo less /var/log/ufw.log`           | View UFW log (if enabled)                       |

---

## 5. Advanced Rules

### Adding More Specific Access Rules

| Command                                                  | Description                                    |
|----------------------------------------------------------|------------------------------------------------|
| `sudo ufw allow proto tcp from <IP> to any port <port>`  | Allow traffic from IP to port using TCP        |
| `sudo ufw allow in on <interface> to any port <port>`    | Allow traffic on specific network interface    |
| `sudo ufw route allow in on <intf> out on <intf>`        | Allow routed traffic between interfaces        |

---

## 6. IPv6 Considerations

### Working with IPv6 Traffic

| Command                              | Description                                       |
|--------------------------------------|---------------------------------------------------|
| Edit `/etc/default/ufw`              | Set `IPV6=yes` to enable IPv6 firewalling         |
| `sudo ufw status verbose`            | Shows IPv6 rules if enabled                       |

---

## 7. Common Use Cases

| Use Case                             | Command                                           |
|--------------------------------------|--------------------------------------------------|
| Allow SSH                            | `sudo ufw allow ssh`                             |
| Allow HTTP and HTTPS                 | `sudo ufw allow 80,443/tcp`                      |
| Deny all incoming, allow outgoing    | `sudo ufw default deny incoming`<br>`sudo ufw default allow outgoing` |
| Block specific IP                    | `sudo ufw deny from <IP>`