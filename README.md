## SOCKS5 Proxy for AWS VPN Client
This guide provides steps to set up a SOCKS5 proxy using SSH dynamic port forwarding through a bastion host on an AWS VPN client. This setup allows a target client on the LAN to access resources via the bastion host.

### Network setup
- Bastion AWS VPN Client Host: A server with IP (e.g., 192.168.0.254)
- Target Client: A client on the LAN (e.g., 192.168.0.100)
- SOCKS5 Proxy Port: 1080

## 1. Bastion Host Setup for LAN Access

### 1. Create a custom table
Add a new table `lanout` with id 100:

```bash
echo "100 lanout" | sudo tee -a /etc/iproute2/rt_tables
```

### 2. Add a route for your server in that table
```bash
ip route add 192.168.0.100/32 dev enx00e04c1b2c80 table lanout
```

Check:
```bash
ip route show table lanout
```

### 3. Add a destination rule with high priority:
```bash
ip rule add to 192.168.0.100/32 lookup lanout priority 100
```

## 2. Taget Client Setup for SOCKS5 Proxy Setup SSH

### 1. Setup proxychains4
Install proxychains4:

```bash
sudo apt install proxychains4
```

### 2. Configure proxychains4
Edit `/etc/proxychains4.conf` and add the following line at the end:

```bash
socks5 127.0.0.1 1080
```

### 3. Create an SSH tunnel with dynamic port forwarding on target client.

```bash
ssh -N -D 1080 user@192.168.0.254
```

### 4. Test the setup
Use proxychains4 to run a command through the SOCKS5 proxy:
```bash
proxychains pg_isready -h postgres-aurora.cluster-xxxxxxxxxxxx.us-west-2.rds.amazonaws.com -p 5432 -t 5
proxychains psql "host=postgres-aurora.cluster-xxxxxxxxxxxx.us-west-2.rds.amazonaws.com  port=5432 dbname=postgres user=admin"
```
