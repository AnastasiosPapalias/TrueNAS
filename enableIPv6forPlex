# How-To: Enable IPv6 for Plex with HTTPS on `plex.mywebsite.com`

This guide will walk you through the process of enabling IPv6 access for your Plex server using a custom domain, specifically `plex.mywebsite.com`. By the end of this tutorial, your Plex server will be accessible securely over IPv6 with HTTPS.

**Prerequisites:**
- You have a domain name (e.g., `mywebsite.com`) and control over its DNS settings.
- Your Plex server has a global IPv6 address.
- You have administrative access to your server and router.

---

## Step 1: Confirm Your Server’s Global IPv6 Address

Your Plex server needs a globally routable IPv6 address to be accessible over the internet.

1. Open a terminal on your Plex server.
2. Run this command to check your IPv6 addresses:
   ```bash
   ip -6 addr show
   ```
3. Look for an address labeled `scope global`. It will look like `2001:db8::1`. Avoid addresses starting with `fe80::` (these are local and won’t work for external access).
4. If you don’t see a global IPv6 address:
   - Confirm your ISP supports IPv6.
   - Check that your router has IPv6 enabled and is assigning addresses to devices.
   - Ensure your server’s network settings allow IPv6.
5. Write down your server’s global IPv6 address—you’ll need it for the next step.

---

## Step 2: Configure DNS with an AAAA Record

You’ll need to link your subdomain `plex.mywebsite.com` to your server’s IPv6 address using a DNS record.

1. Log in to your DNS provider’s control panel (e.g., Namecheap, GoDaddy, Cloudflare).
2. Create a new **AAAA record** with these details:
   - **Name**: `plex.mywebsite.com`
   - **Value**: Your server’s global IPv6 address (e.g., `2001:db8::1`).
   - **TTL**: Leave it at the default (usually 3600 seconds).
3. Save the changes.
4. To confirm it’s working, run this command in your terminal:
   ```bash
   dig AAAA plex.mywebsite.com
   ```
   - Check the response to ensure it shows your IPv6 address.

---

## Step 3: Install and Configure Caddy as a Reverse Proxy

Caddy is a simple tool that will manage HTTPS for you and forward traffic to Plex.

1. **Install Caddy** on your Plex server (these commands work for Debian/Ubuntu-based systems):
   ```bash
   sudo apt install -y debian-keyring debian-archive-keyring apt-transport-https
   curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/gpg.key' | sudo tee /etc/apt/trusted.gpg.d/caddy-stable.asc
   curl -1sLf 'https://dl.cloudsmith.io/public/caddy/stable/debian.deb.txt' | sudo tee /etc/apt/sources.list.d/caddy-stable.list
   sudo apt update
   sudo apt install caddy
   ```
2. **Configure Caddy**:
   - Open the Caddy configuration file in a text editor:
     ```bash
     sudo nano /etc/caddy/Caddyfile
     ```
   - Replace its contents with this:
     ```
     plex.mywebsite.com:443 {
         reverse_proxy localhost:32400
     }
     ```
   - This tells Caddy to:
     - Listen for requests to `plex.mywebsite.com` on port 443 (the standard HTTPS port).
     - Send those requests to Plex, which runs on `localhost:32400`.
3. **Restart Caddy** to apply your changes:
   ```bash
   sudo systemctl restart caddy
   ```

Caddy will automatically get a free HTTPS certificate from Let’s Encrypt and keep it updated.

---

## Step 4: Update Plex Settings

Tell Plex to use your new custom URL for remote access.

1. Open Plex Web on your server by going to `http://localhost:32400/web` in a browser.
2. Navigate to **Settings** > **Network**.
3. Find the **Custom server access URLs** field and enter:
   ```
   https://plex.mywebsite.com:443
   ```
4. Click **Save Changes**.

---

## Step 5: Adjust Firewall Settings

You need to allow IPv6 traffic to reach your server on port 443.

1. **Router Firewall**:
   - Log in to your router’s admin page (check your router’s manual for how to do this).
   - Open **TCP port 443** for IPv6 traffic and point it to your Plex server’s IPv6 address.
2. **Server Firewall** (if your server has a firewall like `ufw` enabled):
   ```bash
   sudo ufw allow 443/tcp
   sudo ufw reload
   ```
3. Double-check that port 443 is open and reachable.

---

## Step 6: Test Your Configuration

Make sure everything works as expected.

1. From a device on a different network (not your home Wi-Fi), visit:
   ```
   https://plex.mywebsite.com:443
   ```
2. Check that:
   - The Plex interface loads in your browser.
   - The connection is secure (look for the padlock icon next to the URL).
3. Open the Plex app on another device and test remote access to confirm it connects properly.

---

## Additional Notes

- **Why Use a Reverse Proxy?**
  - It handles HTTPS certificates automatically, saving you time.
  - It renews certificates for you.
  - It’s handy if you run other services on the same server.

- **Alternative Without a Proxy**
  - You could skip Caddy and open Plex’s default port (32400) directly, but:
    - You’d need to manually set up HTTPS certificates in Plex.
    - Renewing certificates would be your responsibility.
    - It’s less secure and not recommended for internet access.

- **Supporting IPv4 Clients**
  - If some of your users are on IPv4-only networks:
    - Enable Plex Relay (requires Plex Pass, limited to 2 Mbps).
    - Set up a reverse proxy on a VPS that supports both IPv4 and IPv6.
    - Use Cloudflare’s free proxy (not great for video streaming).

---

## Conclusion

By following these steps, your Plex server will be available at `https://plex.mywebsite.com:443` over IPv6 with secure HTTPS, thanks to Caddy. This setup is secure and straightforward. If you need to support IPv4 users too, check the options above. Enjoy your Plex server!

---

This Markdown guide is ready to be copied and pasted into a `.md` file on your GitHub page. It’s written to be easy to follow, with clear instructions and formatting to make each step stand out. Let me know if you’d like any tweaks!
