## Pipe Network Incentivized Testnet Node Installation Guide

![image](https://github.com/user-attachments/assets/202c15fb-8431-4351-8f86-a42577e05d06)

**Get an Invite code! — Register for the node and provide your contact information and geographic information where the PoP node will be located.**

Airtable link: https://airtable.com/apph9N7T0WlrPqnyc/pagSLmmUFNFbnKVZh/form

## Testnet Info's [CLICK HERE](https://pipe.network/blog/announcing-pipequest)

| **Resource**   | **Requirement**  |
|----------------|------------------|
| CPU            | 4 Cores          |
| RAM            | 16 GB            |
| Storage        | 100 GB SSD       |
| Internet       | 1 Gbps           |
| Last Update    | 10-05-2025       |


**Note**: Only users who received an email invitation are eligible to run this node.

### ***You need to choose only one method to run your Pipe Testnet Node, either using Docker or using a Systemd service.***!

## INSTALLATION ***Using Docker***:

### 1. Stop Devnet Node (if previously installed):
```
sudo systemctl stop pop.service
sudo systemctl disable pop.service
```

### 2. Update Packages & Install Dependencies:
```
sudo apt update && sudo apt install libssl-dev ca-certificates docker.io -y
```

### 3. Optimize Network Settings (Recommended):
```
sudo bash -c 'cat > /etc/sysctl.d/99-popcache.conf << EOL
net.ipv4.ip_local_port_range = 1024 65535
net.core.somaxconn = 65535
net.ipv4.tcp_low_latency = 1
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_wmem = 4096 65536 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.core.wmem_max = 16777216
net.core.rmem_max = 16777216
EOL'

sudo sysctl -p /etc/sysctl.d/99-popcache.conf
```
- This will help improve the performance and network reliability of your node.

### 4.  Increase File Descriptor Limits:
```
sudo bash -c 'cat > /etc/security/limits.d/popcache.conf << EOL
*    hard nofile 65535
*    soft nofile 65535
EOL'
```

### 5. Create Installation Directory:
```
sudo mkdir -p /opt/popcache
cd /opt/popcache
```

### 6. Download and Extract Node Binary:
```
wget https://download.pipe.network/static/pop-v0.3.0-linux-x64.tar.gz
sudo tar -xzf pop-v0.3.0-linux-*.tar.gz
chmod 755 /opt/popcache/pop
```

### 7. Create and Edit `config.json`:
```
nano config.json
```
- Paste the following content and replace with your own details:
```
{
  "pop_name": "your-pop-name",
  "pop_location": "Your-City, Your-Country",
  "invite_code": "your-invite-code",
  "server": {
    "host": "0.0.0.0",
    "port": 443,
    "http_port": 80,
    "workers": 40
  },
  "cache_config": {
    "memory_cache_size_mb": 4096,
    "disk_cache_path": "./cache",
    "disk_cache_size_gb": 100,
    "default_ttl_seconds": 86400,
    "respect_origin_headers": true,
    "max_cacheable_size_mb": 1024
  },
  "api_endpoints": {
    "base_url": "https://dataplane.pipenetwork.com"
  },
  "identity_config": {
    "node_name": "Your_Node_Name",
    "name": "Your_Name",
    "email": "your.email@example.com",
    "website": "https://your-website.com",
    "discord": "your_discord_username",
    "telegram": "your_telegram_handle",
    "solana_pubkey": "YOUR_SOLANA_WALLET_ADDRESS"
  }
}
```
- Replace `your-pop-name` with your own name
- Replace `Your-City, Your-Country`
- Replace `your-invite-code`
- Replace `Your_Node_Name`
- Replace `Your_Name`
- Replace `your.email@example.com` Code reciever
- Replace `https://your-website.com` If you don’t have one, just skip it.
- Replace `your_discord_username`
- Replace `your_telegram_handle`
- Replace `YOUR_SOLANA_WALLET_ADDRESS` with your **DEVNET** address or use other wallet addess
- Save with: `CTRL + X`, then `Y`, then `Enter`.

### 8. Create Dockerfile:
```
cat > Dockerfile << EOL
FROM ubuntu:24.04

# Install base dependencies
RUN apt update && apt install -y \\
    ca-certificates \\
    curl \\
    libssl-dev \\
    && rm -rf /var/lib/apt/lists/*

# Set working directory
WORKDIR /opt/popcache

# Copy files from host to container
COPY pop .
COPY config.json .

# Set permissions
RUN chmod +x ./pop

# Run the node
CMD ["./pop", "--config", "config.json"]
EOL
```

### 9.Build the Docker Image:
```
docker build -t popnode .
```

### 10. Run the Node in a Container:
```
docker run -d \
  --name popnode \
  -p 80:80 \
  -p 443:443 \
  --restart unless-stopped \
  popnode
```

### 11. Check Node Status:
- Check running container:
```
docker ps
```
- Check logs:
```
docker logs -f popnode
```

![image](https://github.com/user-attachments/assets/6085f4ec-6a72-4fc7-b73d-27bc59ca1114)

### 12. Check Node Health:
- From terminal (replace `your-vps-ip` with actual `IP`):
```
curl http://your-vps-ip/health
```
- Or from browser:
```
https://your-vps-ip/state
```

## INSTALLATION ***Using Systemd Service***:

### 1. Update Packages & Install Dependencies
```
sudo apt update && sudo apt install libssl-dev ca-certificates -y
```

### 2. Optimize Network Settings:
```
sudo bash -c 'cat > /etc/sysctl.d/99-popcache.conf << EOL
net.ipv4.ip_local_port_range = 1024 65535
net.core.somaxconn = 65535
net.ipv4.tcp_low_latency = 1
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_slow_start_after_idle = 0
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_wmem = 4096 65536 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.core.wmem_max = 16777216
net.core.rmem_max = 16777216
EOL'

sudo sysctl -p /etc/sysctl.d/99-popcache.conf
```

### 3. Increase File Limits:
```
sudo bash -c 'cat > /etc/security/limits.d/popcache.conf << EOL
*    hard nofile 65535
*    soft nofile 65535
EOL'
```
- Logout and log back in to apply changes before proceeding.

### 4. Create Installation Directory:
```
sudo mkdir -p /opt/popcache/logs
cd /opt/popcache
```

### 5. Download & Extract Binary:
```
wget https://download.pipe.network/static/pop-v0.3.0-linux-x64.tar.gz
sudo tar -xzf pop-v0.3.0-linux-*.tar.gz
chmod 755 /opt/popcache/pop
```

### 6. Create & Edit `Config` File:
```
nano config.json
```
- Paste the following config and edit the required fields:
```
{
  "pop_name": "your-pop-name",
  "pop_location": "Your-City, Country",
  "invite_code": "your-invite-code",
  "server": {
    "host": "0.0.0.0",
    "port": 443,
    "http_port": 80,
    "workers": 40
  },
  "cache_config": {
    "memory_cache_size_mb": 4096,
    "disk_cache_path": "./cache",
    "disk_cache_size_gb": 100,
    "default_ttl_seconds": 86400,
    "respect_origin_headers": true,
    "max_cacheable_size_mb": 1024
  },
  "api_endpoints": {
    "base_url": "https://dataplane.pipenetwork.com"
  },
  "identity_config": {
    "node_name": "Your_Node_Name",
    "name": "Your_Name",
    "email": "your.email@example.com",
    "website": "https://your-website.com",
    "discord": "your_discord_username",
    "telegram": "your_telegram",
    "solana_pubkey": "YOUR_SOLANA_ADDRESS"
  }
}
```
- Replace `your-pop-name` with your own name
- Replace `Your-City, Your-Country`
- Replace `your-invite-code`
- Replace `Your_Node_Name`
- Replace `Your_Name`
- Replace `your.email@example.com` Code reciever
- Replace `https://your-website.com` If you don’t have one, just skip it.
- Replace `your_discord_username`
- Replace `your_telegram_handle`
- Replace `YOUR_SOLANA_WALLET_ADDRESS` with your **DEVNET** address or use other wallet addess
- Save with: `CTRL + X`, then `Y`, then `Enter`.

### 7. Create a Systemd Service:
```
sudo bash -c 'cat > /etc/systemd/system/popcache.service << EOL
[Unit]
Description=POP Cache Node
After=network.target

[Service]
Type=simple
User=root
Group=root
WorkingDirectory=/opt/popcache
ExecStart=/opt/popcache/pop
Restart=always
RestartSec=5
LimitNOFILE=65535
StandardOutput=append:/opt/popcache/logs/stdout.log
StandardError=append:/opt/popcache/logs/stderr.log
Environment=POP_CONFIG_PATH=/opt/popcache/config.json

[Install]
WantedBy=multi-user.target
EOL'
```

### 8. Enable and Start the Node:
```
sudo systemctl enable popcache
sudo systemctl daemon-reload
sudo systemctl start popcache
```

- Check Node status:
```
sudo systemctl status popcache
```

- Monitor application logs:
```
tail -f /opt/popcache/logs/stderr.log
```
```
tail -f /opt/popcache/logs/stdout.log
```

![image](https://github.com/user-attachments/assets/36db08b7-4fdb-443a-844c-a4dfd1e618e1)


- Monitor system service logs:
```
sudo journalctl -u popcache -f
```

![image](https://github.com/user-attachments/assets/2209d322-d5b4-4c55-82a7-0dd8254a54ff)

- Health check:
```
curl http://localhost/health
```

- Or check in browser:
```
https://YOUR-VPS-IP/state
```

**📚 Reference**:
**Official Docs**: https://docs.pipe.network/nodes/testnet
