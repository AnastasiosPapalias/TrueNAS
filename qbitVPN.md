# How to Set Up qBittorrent with Wireguard VPN on TrueNAS Scale

This guide provides a comprehensive walkthrough for setting up qBittorrent with a VPN connection using Wireguard on TrueNAS Scale. By following these steps, you’ll ensure your torrent downloads are private and secure, routed through a VPN to protect your identity and prevent ISP monitoring. The setup uses Docker and a `docker-compose` file, which we’ll deploy on TrueNAS Scale using tools like Dockge.

---

## Prerequisites

- **TrueNAS Scale Server**: A running instance with Docker support (e.g., via Dockge or Portainer).
- **VPN Account**: An account with a VPN provider supporting Wireguard (e.g., AirVPN).
- **Basic Knowledge**: Familiarity with Docker, `docker-compose`, and basic command-line operations.
- **File Access**: Ability to access and modify files on your TrueNAS server (e.g., via shell or file explorer).

---

## Step 1: Set Up Your VPN Account and Generate Wireguard Configuration

1. **Create an AirVPN Account**:
   - Visit [AirVPN](https://airvpn.org/) and sign up for an account.
   - Log in to the client area.

2. **Generate a Wireguard Configuration File**:
   - In the client area, go to **Config Generator**.
   - Select **Linux** as the platform and **Wireguard** as the protocol.
   - Enable the **Advanced** option.
   - Choose **IPv4 only** (this avoids IPv6 issues with the Docker container).
   - Pick a server (e.g., one with 20,000 Mbit bandwidth for high performance).
   - Click **Generate** and download the configuration file (e.g., `AirVPN_<location>.conf`).

   **Example Wireguard AirVPN_<location>.conf that we will use for wg0.conf**:
   ```ini
   [Interface]
   Address = 10.169.199.59/32
   PrivateKey = 2P1e/T0RM4mY/D+tNOABbp2DVEh65ry4KzOYy8O4Flk=
   MTU = 1320
   DNS = 10.128.0.1

   [Peer]
   PublicKey = PyLCXAQT8KkM4P-dUsOQfn+Ub3pGxfGlxkIApuig+hk=
   PresharedKey = QtdeTp/kbUYQCR11DZZetZjuyE6kgZ1vH5UM2Bw7y5E=
   Endpoint = 37.46.117.92:1637
   AllowedIPs = 0.0.0.0/0
   PersistentKeepalive = 15
   ```

   **Note**: You will need this info for the file `wg0.conf` later.

---

## Step 2: Prepare Your TrueNAS Scale Environment

1. **Log in to TrueNAS Scale**:
   - Access your TrueNAS Scale web interface.

2. **Install Dockge or Portainer**:
   - This guide uses Dockge for managing Docker containers. Install it via the TrueNAS apps catalog if not already set up.
   - Alternatively, use the TrueNAS apps catalog with a custom app via YAML.

   **Important**: The TrueNAS apps catalog doesn’t natively support qBittorrent with a VPN, so we’ll use a custom `docker-compose` setup.

3. **Create Storage Directories**:
   - Create directories on your TrueNAS server for configuration and downloads (e.g., `/mnt/Pool1/data/qbitvpn/` and `/mnt/Pool1/data/qbitvpn/torrents`).

---

## Step 3: Create the `docker-compose` File

1. **Prepare the `docker-compose` File**:
   - Use the following `docker-compose` configuration, adjusting it to your environment:

   ```yaml
   services:
     qbittorrent:
       container_name: qbittorrent
       image: ghcr.io/hotio/qbittorrent
       restart: unless-stopped
       ports:
         - 8080:8080
       environment:
         - PUID=1000
         - PGID=1000
         - UMASK=002
         - TZ=Europe/Athens
         - WEBUI_PORTS=8080/tcp,8080/udp
         - VPN_ENABLED=true
         - VPN_CONF=wg0
         - VPN_PROVIDER=generic
         - VPN_LAN_NETWORK=192.168.1.0/24
         - VPN_EXPOSE_PORTS_ON_LAN=
         - VPN_AUTO_PORT_FORWARD=true
         - VPN_AUTO_PORT_FORWARD_TO_PORTS=5687
         - VPN_KEEP_LOCAL_DNS=false
         - VPN_FIREWALL_TYPE=auto
         - PRIVOXY_ENABLED=false
         - UNBOUND_ENABLED=false
       cap_add:
         - NET_ADMIN
       sysctls:
         - net.ipv4.conf.all.src_valid_mark=1
         - net.ipv6.conf.all.disable_ipv6=1
       volumes:
         - /mnt/Pool1/data/qbitvpn/:/config
         - /mnt/Pool1/data/qbitvpn/torrents:/downloads
   ```

2. **Customize Environment Variables**:
   - **`TZ`**: Set your timezone (e.g., `Europe/Athens`).
   - **`VPN_LAN_NETWORK`**: Match your local network subnet (e.g., `192.168.1.0/24`). Check your router’s IP range (e.g., `192.168.1.x`).
   - **`VPN_AUTO_PORT_FORWARD_TO_PORTS`**: Set the port for qBittorrent (e.g., `5687`).

3. **Customize Volumes**:
   - **`/mnt/Pool1/data/qbitvpn/:/config`**: Configuration storage. Adjust the host path to a valid TrueNAS location.
   - **`/mnt/Pool1/data/qbitvpn/torrents:/downloads`**: Downloads storage. Adjust accordingly.

   **Note**: Ensure these directories exist (e.g., via `mkdir -p /mnt/Pool1/data/qbitvpn/torrents` in the TrueNAS shell).

---
## Step 4: Deploy the Container

1. **Deploy via Dockge**:
   - Open Dockge, create a new stack (e.g., `qbitvpn`), paste the `docker-compose` file, and click **Deploy**.
   - The container will start normaly.

---

## Step 5: Add the Wireguard Configuration File

1. **Create the file wg0.conf**:
   - Create the file with: nano wg0.conf inside the /wireguard folder.
     ```bash
     nano wg0.conf
     ```

2. **Edit the File**:
   - Copy the contents of your previously dowloaded file from airvpn inside the wg0.conf, and save.

   **Note**: The container expects `wg0.conf` in this exact location.

3. **Stop and Deply**:
   - Stop the container and deploy again.
     
---

## Step 6: Verify the VPN Connection

1. **Access qBittorrent Web UI**:
   - Open a browser and go to `http://<your_truenas_ip>:8080`.
   - Default credentials: Username: `admin`, Password: Check the container logs for the initial password.

2. **Verify VPN IP**:
   - Exec into the container via Dockge or Portainer shell:
     ```bash
     curl ip.me
     ```
   - The returned IP should match your VPN’s IP, something like 37.46.117.90, not your public ISP IP.

---

## Step 7: Configure qBittorrent Settings

1. **Set Default Save Path**:
   - In the web UI, go to **Settings > Downloads**.
   - Set the default save path to `/downloads`.

2. **Configure Listening Port**:
   - Go to **Settings > Connection**.
   - See that the incoming connections port is set to something like `5687` (or your chosen port)? We are going to change that.

3. **Bind to Wireguard Interface**:
   - Go to **Settings > Advanced**.
   - Set **Network interface** to `wg0`.

4. **Save Changes**:
   - Apply settings and restart qBittorrent if prompted.

---

## Step 8: Set Up Port Forwarding

1. **Configure Port Forwarding in AirVPN**:
   - Log in to AirVPN’s client area.
   - Go to "**Client Area** and then to **Your VPN devices** and "Add a new device" with a name like "VPN QB".
   - Go to **Client Area** then to **Ports** and add a new port with the + symbol. It should be something like "27404".
   - Assign it to your device "VPN QB".
   - For Protocol use "TCP+UDP"
   - For IP Layer use "IPv4 only"
   - Save and note the forwarded port (i.e 27404).

2. **Update qBittorrent**:
   - Ensure the port in **Settings > Connection** matches the forwarded port (i.e 27404).

3. **Verify Port**:
   - Use the test port feature from Airvpn to test it.
   - Go to [air](https://airvpn.org/ports/) and click "Test open" 

---

## Step 9: Test the Setup

1. **Add a Test Torrent**:
   - Upload a torrent (e.g., Ubuntu ISO) via the web UI.

2. **Check Download**:
   - Verify it downloads, even if slowly. This confirms the VPN and qBittorrent are working.

3. **Monitor Logs**:
   - Ensure no errors appear in the container logs.

---

## Troubleshooting

- **Container Restarts**: Check `wg0.conf` placement and contents.
- **No VPN Connection**: Verify `curl ip.me` returns the VPN IP.
- **Port Not Open**: Confirm port forwarding settings in AirVPN and qBittorrent match.
- **Path Errors**: Ensure volume paths exist and are accessible.

---

## Conclusion

You now have qBittorrent running securely behind a Wireguard VPN on TrueNAS Scale. Your torrent traffic is encrypted and routed through AirVPN, with a kill switch ensuring privacy if the VPN fails. 

---

