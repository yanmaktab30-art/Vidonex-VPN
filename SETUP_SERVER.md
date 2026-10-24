# WireGuard VPS Server Setup Guide

This guide explains how to deploy a WireGuard VPN server on a fresh Ubuntu 22.04 LTS VPS.
All commands must be run as root or with sudo on your server.

> IMPORTANT: This guide is for informational purposes only.
> Do NOT run these commands on infrastructure you do not own or control.
> You are responsible for securing your server and complying with applicable laws.

---

## 1. Requirements

- A fresh Ubuntu 22.04 LTS VPS (any cloud provider: DigitalOcean, Vultr, Hetzner, etc.)
- A public IPv4 address for the VPS
- SSH access to the server
- A domain name (optional but recommended)

---

## 2. Install WireGuard

```bash
apt update && apt upgrade -y
apt install -y wireguard wireguard-tools
```

---

## 3. Enable IP Forwarding

```bash
echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
echo "net.ipv6.conf.all.forwarding=1" >> /etc/sysctl.conf
sysctl -p
```

---

## 4. Generate Server Keys

```bash
cd /etc/wireguard
umask 077
wg genkey | tee server_private.key | wg pubkey > server_public.key
cat server_private.key   # Keep this SECRET - never share
cat server_public.key    # This goes into client configs
```

---

## 5. Create Server Configuration

Replace VARIABLES with your actual values:
- SERVER_PRIVATE_KEY: contents of server_private.key
- SERVER_PUBLIC_IP: your VPS public IP address
- WAN_INTERFACE: your server network interface (check with: ip route | grep default)

```bash
cat > /etc/wireguard/wg0.conf << 'WGCONF'
[Interface]
Address = 10.0.0.1/24
ListenPort = 51820
PrivateKey = SERVER_PRIVATE_KEY

# NAT/masquerade rules - replace eth0 with your WAN interface
PostUp = iptables -t nat -A POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE
PostUp = ip6tables -t nat -A POSTROUTING -s fd42::/64 -o eth0 -j MASQUERADE
PostDown = iptables -t nat -D POSTROUTING -s 10.0.0.0/24 -o eth0 -j MASQUERADE
PostDown = ip6tables -t nat -D POSTROUTING -s fd42::/64 -o eth0 -j MASQUERADE

DNS = 1.1.1.1, 1.0.0.1

# Peers will be added below this line
WGCONF
```

Restrict file permissions:
```bash
chmod 600 /etc/wireguard/wg0.conf
chmod 600 /etc/wireguard/server_private.key
```

---

## 6. Start WireGuard

```bash
systemctl enable wg-quick@wg0
systemctl start wg-quick@wg0
systemctl status wg-quick@wg0
```

---

## 7. Configure Firewall (UFW)

```bash
ufw allow 22/tcp          # SSH (keep this or you will lose access!)
ufw allow 51820/udp       # WireGuard
ufw enable
ufw status
```

---

## 8. Generate a Client Configuration

For each client (phone, tablet, laptop), generate unique keys:

```bash
cd /etc/wireguard
umask 077
wg genkey | tee client1_private.key | wg pubkey > client1_public.key
```

Add the client as a peer in wg0.conf:
```bash
wg set wg0 peer CLIENT1_PUBLIC_KEY allowed-ips 10.0.0.2/32
```

Or add to /etc/wireguard/wg0.conf permanently:
```
[Peer]
PublicKey = CLIENT1_PUBLIC_KEY
AllowedIPs = 10.0.0.2/32
```

Create the client .conf file (this is what you import into Vidonex VPN):
```
[Interface]
Address = 10.0.0.2/24
PrivateKey = CLIENT1_PRIVATE_KEY
DNS = 1.1.1.1, 1.0.0.1

[Peer]
PublicKey = SERVER_PUBLIC_KEY
Endpoint = YOUR_SERVER_PUBLIC_IP:51820
AllowedIPs = 0.0.0.0/0, ::/0
PersistentKeepalive = 25
```

> SECURITY: The client .conf file contains the client private key.
> Transfer it securely (e.g., via QR code locally, or encrypted transfer).
> NEVER send it in plain email, SMS, or chat.

---

## 9. Import Configuration into Vidonex VPN

1. Transfer the .conf file to your Android device (USB cable recommended)
2. Open Vidonex VPN
3. Tap "Servers" in the bottom navigation
4. Tap the import icon (folder icon, top right)
5. Select the .conf file
6. The server appears in your list — tap it, then return to Dashboard and Connect

---

## 10. Revoking a Client

```bash
wg set wg0 peer CLIENT1_PUBLIC_KEY remove
# Also remove the [Peer] block from /etc/wireguard/wg0.conf
# Then delete client1_private.key and client1_public.key
```

---

## 11. Diagnostics and Troubleshooting

Check WireGuard status:
```bash
wg show wg0
```

View active peers and traffic:
```bash
wg show wg0 transfer
```

Check service logs:
```bash
journalctl -u wg-quick@wg0 -f
```

Test connectivity from client:
```bash
ping 10.0.0.1    # Server internal IP
curl ifconfig.me  # Should show server public IP when VPN connected
```

Common issues:
- No handshake: check firewall allows UDP port 51820
- No internet: check IP forwarding and iptables masquerade rules
- DNS leaks: ensure DNS = is set in client [Interface] section
- Timeout: verify server public IP and port in client [Peer] Endpoint

---

## 12. Security Hardening Checklist

- [ ] Disable password SSH authentication (use keys only)
- [ ] Keep server updated: apt update && apt upgrade regularly
- [ ] Use a non-default WireGuard port (change 51820 to another value)
- [ ] Rotate client keys periodically
- [ ] Monitor /var/log/auth.log for unauthorized access attempts
- [ ] Consider fail2ban for SSH protection
- [ ] Use Cloudflare or similar DNS resolver for privacy
- [ ] Restrict SSH to specific source IPs if possible

---

## Disclaimer

This guide is provided as a convenience reference. You are solely responsible for
the security, legality, and proper configuration of your server infrastructure.
Vidonex provides no warranties, SLAs, or support for third-party VPS deployments.