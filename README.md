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

## INSTALLATION:

### Stop Devnet Node (if previously installed):
```
sudo systemctl stop pipe-pop.service
sudo systemctl disable pipe-pop.service
```

### 1. Update Packages & Install Dependencies
```
sudo apt update && sudo apt install libssl-dev ca-certificates -y
```

- Ensure the required ports (443 and 80) are open:
```
sudo ufw allow 443/tcp
sudo ufw allow 80/tcp
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
![image](https://github.com/user-attachments/assets/7eac5cb2-2652-4520-b0b6-72f9d4d02e25)

- Monitor application logs:
```
tail -f /opt/popcache/logs/stderr.log
```
```
tail -f /opt/popcache/logs/stdout.log
```

![image](https://github.com/user-attachments/assets/48d83660-a100-41ac-bc04-2e4715370912)


- Monitor system service logs:
```
sudo journalctl -u popcache -f
```

![image](https://github.com/user-attachments/assets/ac630f73-c927-49af-9d71-2d588bdbb6c5)


- Health check:
```
curl http://localhost/health
```
![image](https://github.com/user-attachments/assets/c7f5e964-80cc-439d-bfc4-23822589fcb9)

- Or check in browser:
```
https://YOUR-VPS-IP/state
```
![image](https://github.com/user-attachments/assets/306793d6-6064-4076-93d7-1a798f60f291)

**📚 Reference**:
**Official Docs**: https://docs.pipe.network/nodes/testnet
