# Firewall-on-Windows-Linux
Configure and test basic firewall rules to allow or block traffic.

This repository provides **safe, educational examples** of configuring and testing basic firewall rules to **allow or block traffic**.
It is intended for a **home lab** or isolated test environment only.

# Lab Setup (Recommended)

1. Use at least two virtual machines (VMs) on an isolated network:
   - VM A: "client" (tester)
   - VM B: "server" (target for tests)

2. Snapshot VMs before making firewall changes.

3. Use an internal-only virtual network (no internet) if you want full isolation,
   or restrict internet using NAT and firewalling.

4. Install required tools:
   - Linux: iptables/nftables, ufw, 
   - Windows: PowerShell (Admin), OpenSSH (optional), Test-NetConnection
  
   # Safety & Testing Rules

- Only run tests in environments you control.
- Do NOT point blocking rules at real production IPs you don't own.
- Keep an out-of-band management connection (console) in case rules cut off access.
- Use time-limited changes or keep a recovery plan (reboot, rescue ISO).
- Document each change and how to revert it.

  # Testing Plan - Allow / Block Scenarios

## Scenario 1: Allow HTTP to server
- Apply rule to allow TCP port 80 on server.
- From client: `curl http://192.168.20.83/` or `nc <server-ip> 80`
- Expect: connection succeeds.

## Scenario 2: Block a specific source IP
- Apply rule on server to DROP or BLOCK traffic from client IP.
- From blocked client: attempt ping, nc, curl.
- Expect: connection fails.

## Scenario 3: Allow only SSH from management IP
- Apply rule permitting SSH only from `192.168.20.54`.
- From another client IP: SSH should fail.

## Validation tools
- `tcpdump` / `wireshark` on server to confirm packet arrival.
- `ss -tulwn` or `netstat -tulpen` to confirm services listening.

## What's inside
- `configs/` - Example firewall configurations (iptables, nftables, ufw, Windows PowerShell)
- # UFW example commands (Ubuntu/Debian)
# Run as root / with sudo

# Reset UFW to defaults
sudo ufw reset

# Default deny incoming, allow outgoing
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH from a single IP
sudo ufw allow from 192.168.20.54 to any port 22 proto tcp

# Allow HTTP/S from anywhere
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Deny a specific IP
sudo ufw deny from 198.168.20.53

# Enable UFW
sudo ufw enable

# Check status
sudo ufw status verbose

# PowerShell example to manage Windows Defender Firewall rules
# Run in an elevated PowerShell prompt

# Allow inbound HTTP
New-NetFirewallRule -DisplayName "Allow HTTP Inbound" -Direction Inbound -LocalPort 80 -Protocol TCP -Action Allow

# Allow inbound SSH (if OpenSSH server installed) from a specific IP
New-NetFirewallRule -DisplayName "Allow SSH from LabIP" -Direction Inbound -LocalPort 22 -Protocol TCP -RemoteAddress 192.168.
20.54 -Action Allow

# Block a specific remote address
New-NetFirewallRule -DisplayName "Block Bad IP" -Direction Inbound -RemoteAddress 198.168.20.55 -Action Block

# List rules we added (filter by DisplayName)
Get-NetFirewallRule -DisplayName "Allow HTTP Inbound","Allow SSH from LabIP","Block Bad IP" | Format-Table

#!/bin/bash
# Simple iptables examples (Linux). Run as root.
# WARNING: these commands change firewall rules immediately.

echo "Flushing existing rules..."
iptables -F
iptables -X

# Default policies: drop everything (simple learning baseline)
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -P OUTPUT ACCEPT

# Allow loopback
iptables -A INPUT -i lo -j ACCEPT

# Allow established/related
iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT

# Allow SSH (port 22) from a specific IP (replace 192.168.20.54)
iptables -A INPUT -p tcp -s 192.168.20.54 --dport 22 -m conntrack --ctstate NEW -j ACCEPT

# Allow HTTP from anywhere
iptables -A INPUT -p tcp --dport 80 -m conntrack --ctstate NEW -j ACCEPT

# Block a specific IP (example)
iptables -A INPUT -s 198.168.20.55 -j DROP

echo "Current filter table:"
iptables -L -n -v

# nftables simple example (Linux)
table inet filter {
    chain input {
        type filter hook input priority 0;
        policy drop;

        # Allow loopback
        iif "lo" accept

        # Allow established
        ct state established,related accept

        # Allow SSH from specific IP (replace 192.168.20.54)
        ip saddr 192.168.20.54 tcp dport 22 ct state new accept

        # Allow HTTP
        tcp dport 80 ct state new accept

        # Block specific IP
        ip saddr 198.168.20.55 drop
    }

    chain forward { policy drop; }
    chain output { policy accept; }
}




## Safety & scope
- Do **not** run these commands on production systems or on networks you do not control.
- Use virtual machines or isolated lab network.
- Prefer taking snapshots/checkpoints before applying rules.

