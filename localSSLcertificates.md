### Introduction
This guide will help you set up local SSL certificates for your homelab on TrueNAS Scale 24.10, focusing on using docker images, Portainer, and containers. We’ll adapt the process to ensure your applications (like Plex or Home Assistant) are accessible locally with secure, valid SSL certificates without public exposure.

### Setting Up Your Domain
First, register a domain with a DNS provider like DuckDNS for a free option or Namecheap for paid. Point this domain to your public IP address, e.g., `yourdomain.duckdns.org`. This is crucial for Let’s Encrypt to verify ownership via DNS challenge.

### Configuring Local DNS Resolution
To access your domain internally, set up a local DNS server like Pi-Hole on your network. Configure it to map your domain (e.g., `yourdomain.duckdns.org`) to TrueNAS Scale’s local IP (e.g., `192.168.1.100`). This ensures local users can resolve the domain to access applications privately.

### Installing and Configuring NGINX Proxy Manager
Install NGINX Proxy Manager through TrueNAS Scale’s app catalog under **Apps**, or manually via Docker if needed. Configure it to use Let’s Encrypt’s DNS challenge for SSL certificates, ensuring it listens only on the local network to prevent public access. Set up proxy hosts for each application, linking them to their local ports.

### Verifying the Setup
Test by accessing your domain locally (e.g., `https://yourdomain.duckdns.org`) to ensure applications load with valid SSL. Confirm no public access is possible by checking from an external network.

---

### Survey Note: Detailed Guide for Setting Up Local SSL Certificates on TrueNAS Scale 24.10

This comprehensive guide expands on the steps to set up local SSL certificates for your homelab using TrueNAS Scale 24.10, based on the provided attachment and adapted for current capabilities. It addresses the use of docker images, Portainer, and containers, ensuring a secure and user-friendly setup without public exposure.

#### Background and Context
The attachment provided a tutorial on setting up a reverse proxy and obtaining valid SSL certificates for home lab applications (e.g., Plex, Sonarr, Paperless, Home Assistant) using NGINX Proxy Manager and Let’s Encrypt’s DNS challenge. This guide adapts that process for TrueNAS Scale 24.10, which has shifted from Kubernetes to Docker for app management, enhancing flexibility for homelab setups.

#### Step-by-Step Instructions

##### 1. Registering a Domain with a DNS Provider
To begin, you need a domain for your applications. Options include:
- **DuckDNS**: Offers free dynamic DNS services, ideal for homelabs. Register at [DuckDNS](https://www.duckdns.org/).
- **Paid Providers**: Consider Namecheap or Cloudflare for more features.

After registration, update the DNS settings to point your domain (e.g., `yourdomain.duckdns.org`) to your public IP address. This is necessary for Let’s Encrypt’s DNS-01 challenge, which verifies domain ownership via DNS records rather than HTTP access.

##### 2. Setting Up Local DNS Resolution
For local access without public exposure, your domain must resolve to TrueNAS Scale’s local IP internally. The attachment’s summary suggests no need for custom DNS servers, but practical implementation often requires one:
- **Install Pi-Hole**: Pi-Hole can run as a docker container on TrueNAS Scale. Install it via the app catalog or manually, and configure it to resolve `yourdomain.duckdns.org` to your TrueNAS Scale’s local IP (e.g., `192.168.1.100`).
- **Alternative**: Some routers support local DNS overrides. Check your router’s settings for domain-to-IP mapping, though this is less common.

This step ensures local users can access `https://yourdomain.duckdns.org` and reach your applications, while external users cannot, as the reverse proxy will not be exposed publicly.

##### 3. Installing NGINX Proxy Manager on TrueNAS Scale 24.10
TrueNAS Scale 24.10 supports Docker natively, making it straightforward to deploy NGINX Proxy Manager:
- **Via App Catalog**: Navigate to **Apps** in the TrueNAS Scale UI, search for “Nginx Proxy Manager,” and install. Note: Some users report deployment issues, so ensure your storage pool (preferably SSD) supports fast I/O, as HDDs may cause delays ([TrueNAS Community](https://www.truenas.com/community/threads/nginx-proxy-manager-wont-deploy.113904/)).
- **Manual Docker Installation**: If issues arise, pull the image and run:
  ```
  docker pull jc21/nginx-proxy-manager
  docker run -d --name=nginx_proxy_manager -p 80:80 -p 443:443 -v /path/to/data:/data jc21/nginx-proxy-manager
  ```
  Adjust paths and ports as needed, ensuring the container listens on local interfaces only.

##### 4. Configuring NGINX Proxy Manager for SSL and Proxying
Once installed, access NGINX Proxy Manager’s UI (typically at `http://localhost:81` or as configured):
- **Set Up SSL Certificates**: Go to **SSL Certificates**, click **Add SSL Certificate**, and select **Let’s Encrypt**. Enable **Use DNS Challenge** and configure your DNS provider (e.g., DuckDNS). You’ll need API credentials for automation, which DuckDNS provides via token.
- **Create Proxy Hosts**: For each application (e.g., Nextcloud on port 8080), add a proxy host:
  - Domain Name: `nextcloud.yourdomain.duckdns.org`
  - Forward Hostname / IP: `127.0.0.1` or local IP, Forward Port: Application port (e.g., 8080).
  - Enable SSL and select the certificate created earlier.

Ensure NGINX Proxy Manager is configured to listen only on the local network interface to prevent public access. This can be adjusted in the Docker settings or by editing the configuration files, though the UI typically handles this.

##### 5. Setting Up and Configuring Local Applications
TrueNAS Scale 24.10 allows easy deployment of applications via its apps feature or Docker:
- **Using Apps Feature**: Install applications like Plex or Home Assistant from the catalog, noting their ports (e.g., Plex on 32400).
- **Manual Docker**: Use Portainer for container management. Pull images and run, mapping ports as needed:
  ```
  docker run -d --name=plex -p 32400:32400 plexinc/pms-docker
  ```
  Ensure applications are accessible locally on their ports before proxying.

##### 6. Verifying the Setup
- **Local Access Test**: From a device on your local network, access `https://yourdomain.duckdns.org` or subdomains. Verify SSL is valid (green lock in browser) and applications load correctly.
- **Public Access Check**: Attempt access from outside your network (e.g., via mobile data). It should fail, confirming no public exposure. If accessible, review router port forwarding and NGINX Proxy Manager settings.

#### Additional Considerations and Troubleshooting
- **Certificate Renewal**: NGINX Proxy Manager automates Let’s Encrypt renewals, but ensure your DNS provider credentials remain valid.
- **Performance**: If deployment hangs, consider using an SSD pool for app storage, as HDDs may cause delays ([TrueNAS Community](https://www.truenas.com/community/threads/cant-install-nginx-proxy-manager.111184/)).
- **Security**: Configure access lists in NGINX Proxy Manager to restrict access to local IPs only, enhancing privacy.

#### Tables for Reference

| Step                 | Action                                      | Notes                                                                 |
|----------------------|---------------------------------------------|----------------------------------------------------------------------|
| Domain Registration  | Register with DuckDNS or Namecheap          | Point to public IP for Let’s Encrypt verification.                   |
| Local DNS Setup      | Install Pi-Hole, map domain to local IP     | Essential for internal resolution; router may support, but rare.      |
| Install NPM          | Use app catalog or Docker manually          | Prefer SSD pool for performance; watch for deployment issues.         |
| Configure SSL        | Use DNS challenge in NPM, set up provider   | Requires API credentials; automates certificate management.           |
| Set Up Applications  | Install via apps or Docker, note ports      | Use Portainer for container management if preferred.                  |
| Verify               | Test locally, check no public access        | Ensure SSL valid, applications inaccessible externally.               |

#### Conclusion
This guide provides a detailed pathway to secure your homelab applications on TrueNAS Scale 24.10 with local SSL certificates, leveraging NGINX Proxy Manager and Let’s Encrypt’s DNS challenge. By setting up local DNS resolution, you ensure private, user-friendly access without public exposure, aligning with homelab security best practices.

### Key Citations
- [TrueNAS SCALE 24.10 Shifts from Kubernetes to Docker Linuxiac](https://linuxiac.com/truenas-scale-24-10-shifts-from-kubernetes-to-docker/)
- [24.10 (Electric Eel) Version Notes TrueNAS Documentation Hub](https://www.truenas.com/docs/scale/24.10/gettingstarted/scalereleasenotes/)
- [How to Use Docker on TrueNAS Scale WunderTech](https://www.wundertech.net/how-to-use-docker-on-truenas-scale/)
- [Nginx proxy manager on TrueNas Scale without True Charts LNA-DEV](https://lna-dev.net/en/posts/home-server/nginx-proxy-manager/)
- [Nginx Proxy Manager Setup TrueNAS Community](https://www.truenas.com/community/threads/nginx-proxy-manager-setup.116682/)
- [TrueNas Scale config reverse proxy with NGINX proxy manager TrueNAS Community Forums](https://forums.truenas.com/t/truenas-scale-config-reverse-proxy-with-nginx-proxy-manager/22696)
- [Nginx Proxy Manager App and internal DNS TrueNAS Community](https://www.truenas.com/community/threads/nginx-proxy-manager-app-and-internal-dns.110300/)
- [Access Truenas scale UI with nginx TrueNAS Community](https://www.truenas.com/community/threads/access-truenas-scale-ui-with-nginx.115642/)
- [Having trouble using Nginx Proxy Manager for TrueNAS apps TrueNAS Community Forums](https://forums.truenas.com/t/having-trouble-using-nginx-proxy-manager-for-truenas-apps/9062)
- [Nginx Proxy Manager won't deploy TrueNAS Community](https://www.truenas.com/community/threads/nginx-proxy-manager-wont-deploy.113904/)
- [Can't install Nginx Proxy Manager TrueNAS Community](https://www.truenas.com/community/threads/cant-install-nginx-proxy-manager.111184/)
- [Guide Nginx Proxy Manager](https://nginxproxymanager.com/guide/)
- [HomeLab: Nginx-Proxy-Manager: Setup SSL Certificate with Domain Name in Cloudflare DNS Medium](https://medium.com/@life-is-short-so-enjoy-it/homelab-nginx-proxy-manager-setup-ssl-certificate-with-domain-name-in-cloudflare-dns-732af64ddc0b)
- [Wildcard Let's Encrypt certificates with Nginx Proxy Manager and Cloudflare jverkamp.com](https://blog.jverkamp.com/2023/03/27/wildcard-lets-encrypt-certificates-with-nginx-proxy-manager-and-cloudflare/)
