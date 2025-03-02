This project documents the process of installing and managing the Jellyfin Media Server within WSL (Ubuntu) on a Windows machine. It includes instructions for setting up a dedicated movies folder, configuring Windows port forwarding using `netsh`, testing mobile connectivity.

## Overview

This project allows you to run Jellyfin inside WSL Ubuntu while exposing it to your local network via Windows port proxy. It lets you stream your movies using a mobile app or browser and also provides clear instructions to disable or remove the setup when needed.

## Prerequisites

- Windows 10/11 with WSL enabled
- Ubuntu installed via WSL
- Administrator privileges on Windows (to set up port forwarding)
- Basic familiarity with Linux command-line

## Installation & Configuration

### 1. Install Jellyfin in WSL Ubuntu

Open your WSL Ubuntu terminal and run the following commands:

```bash
sudo apt update
sudo apt install apt-transport-https ca-certificates curl gnupg -y

# Add Jellyfin GPG key and repository
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://repo.jellyfin.org/jellyfin_team.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/jellyfin.gpg
echo "deb [signed-by=/etc/apt/keyrings/jellyfin.gpg arch=$(dpkg --print-architecture)] https://repo.jellyfin.org/ubuntu $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/jellyfin.list

sudo apt update
sudo apt install jellyfin -y

# Enable and start Jellyfin service
sudo systemctl enable --now jellyfin
```

### 2. Create and Configure the Movies Folder

Decide on a dedicated location for your media files. For example, to create a movies folder:

```bash
sudo mkdir -p /media/movies
```

If you use ACLs for permissions (make sure the `acl` package is installed):

```bash
sudo apt install acl -y
sudo setfacl -m u:jellyfin:rx /media/movies
```

Place or copy your movie files into the `/media/movies` folder.

### 3. Configure Windows Port Proxy

Since Jellyfin runs in WSL, it’s on a virtual network. To allow mobile devices to connect, forward the port from your Windows host to WSL:

1. **Find your Windows LAN IP**  
    Run `ipconfig` in a Command Prompt (look for your Wi‑Fi adapter IPv4, e.g., `192.168.1.9`).
    
2. **Set up the port proxy (run as Administrator):**
    
    First, remove any existing rule (if applicable):

```bash
netsh interface portproxy delete v4tov4 listenport=8096 listenaddress=0.0.0.0
```

Then add a new rule (replace `192.168.1.9` with your actual Windows LAN IP and confirm your WSL IP is correct, e.g., `192.168.36.122`):

    
3.  **Firewall Considerations:**  
    Ensure that Windows Defender Firewall allows inbound connections on TCP port 8096. You may need to create an inbound rule for this port.

## Usage

### Accessing Jellyfin on Mobile

On your mobile device (connected to the same network), open the Jellyfin mobile app or a web browser and use the following URL:

```bash
http://192.168.1.9:8096
```

![Screenshot 2025-03-02 182052](https://github.com/user-attachments/assets/c0ae555a-ba97-46da-9d5b-1e5fa836f502)


This URL uses your Windows LAN IP (as configured above) and forwards traffic to the Jellyfin server running in WSL.

## Troubleshooting

- **WSL IP Changes:**  
    If the WSL IP (`192.168.36.122` in our example) changes after a restart, update the port proxy rule accordingly.
    
- **Firewall Issues:**  
    Confirm Windows Firewall isn’t blocking port 8096. Temporarily disable it to test connectivity if needed.
    
- **Service Status:**  
    Check Jellyfin’s status in WSL with:

```bash
sudo systemctl status jellyfin
```

![Screenshot 2025-03-02 182124](https://github.com/user-attachments/assets/cb145578-19e7-45ba-81b8-e3804a2fe92d)
    
- **Network Connectivity:**  
    Make sure your mobile device is on the same network as your Windows machine.
