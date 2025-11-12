FortiClient VPN-Only Sucks for Both Windows and Linux Users.

Fortinet is dropping SSL VPN support in newer firmware versions. For Linux users who need VPN access for remote work, OpenConnect works significantly better than FortiClient VPN-only on Linux and Windows.

As a cybersecurity engineer who uses Linux every day, everything works well in Linux, and it’s possible to integrate the machine with a domain controller and other security products, such as EDR, while still retaining the full Linux user experience.

We were experiencing issues with backing up our cloud Linux server at HQ, but StrongSwan resolved this. Now, we can create an IPsec Linux-to-site tunnel without any problems. The setup supports auto-connect, maintains an always-on connection, and automatically retries if the tunnel goes down.

### The Pain Points with FortiClient
From my hands-on work, FortiClient has some notable limitations:
- **Business Model Trade-offs**: Features like auto-connect and always-on are locked behind paid tiers, which makes sense for Fortinet but frustrates individual users or small teams.
- **Windows 11 Compatibility Issues**: Many report random disconnections, "host is down" errors, SAML authentication crashes, and unhelpful logs for debugging.
- **Linux Gaps**: No native IPsec client-to-site support. It's baffling why this is missing, forcing users to seek workarounds.


<img width="1006" height="821" alt="image" src="https://github.com/user-attachments/assets/fee11f46-342b-46f3-903c-2f5cc19d80b1" />


These reminders pop up for free version users—it's just a simple VPN, right?
<img width="1011" height="634" alt="image" src="https://github.com/user-attachments/assets/742e36c6-0a03-449b-893f-83b890a2b48a" />

In cybersecurity, these issues aren't just annoyances; they can expose data to risks like man-in-the-middle attacks if connections drop unexpectedly. Encrypting traffic end-to-end is crucial for compliance and protection.

### Discovering StrongSwan: A Faster, More Flexible Alternative
Enter StrongSwan—an open-source powerhouse for IPsec VPNs. It might seem intimidating at first, but it's faster, easier to debug, and highly customizable. Key perks:
- Supports IPsec IKEv1 and IKEv2.
- Enables auto-connect and always-on features.
- Provides detailed logs and straightforward CLI commands.
- Integrates with Network Manager (though CLI via swanctl is more robust).
- Ideal for Linux-to-site tunnels: Connect your cloud Linux server to local HQ for secure backups and resource access. No floating data over the internet—just an encrypted IPsec tunnel.

The documentation can be a bit scattered, but once you get past that, it's a game-changer. I've used it to create stable, monitored connections that outperform proprietary options.

### A Ready-to-Use StrongSwan Configuration Example
Here's a swanctl.conf setup I've tested for an IPsec IKEv1 tunnel using a pre-shared key (PSK) and username/password (XAuth). I've added comments for clarity. **Important Security Note**: Always replace placeholders (e.g., IPs, PSKs, passwords) with your own secure values. Use strong, unique keys and follow best practices—this is for educational purposes only.

### Crafting Proposals: The Tricky Part Made Simple
Proposals define the encryption and integrity algorithms. This can be the hardest step after installing StrongSwan on Debian 12.

<img width="1002" height="359" alt="image" src="https://github.com/user-attachments/assets/59d820ef-1596-41cc-8e00-41724d8c068b" />

Check out the official docs for more: https://docs.strongswan.org/docs/latest/config/proposals.html. Use Ctrl+F to find IANA group names and map them to StrongSwan keywords.

You can chain multiple proposals with commas, like this:

<img width="1011" height="471" alt="image" src="https://github.com/user-attachments/assets/c1dc5d09-5940-4feb-986c-78e859a45468" />


### Installation and Setup Tips
- **Debian 12**: Avoid `apt` installs due to common issues. Download the source from strongswan.org and compile:
```bash
sudo apt update
sudo apt upgrade
sudo apt install build-essential wget bzip2 bison flex gperf libtool pkg-config libgmp-dev libssl-dev libgcrypt20-dev libcurl4-openssl-dev libldap2-dev libsqlite3-dev libxml2-dev libsystemd-dev libpam0g-dev libcap-dev libkrb5-dev libtss2-dev libnm-dev libiptc-dev
```
```bash
tar xjf strongswan-6.0.2.tar.bz2
cd strongswan-6.0.2
```

```bash
  ./configure --prefix=/usr --sysconfdir=/etc --with-ipseclibdir=/usr/lib/$(dpkg-architecture -qDEB_BUILD_MULTIARCH) --enable-swanctl --enable-systemd
  make
  sudo make install
```

Place your config in `/etc/swanctl/conf.d/swanctl.conf`.

- **Fedora**: Simpler—just use `dnf install strongswan`. Config goes in `/etc/strongswan/swanctl/swanctl.conf`.
if you have some missing plugins ChatGPT will helps you find that i did install that long time ago and i didn't rememeber which package i did install


The same config works across distributions.
To get started:
1. Load the config:
   ```
   sudo swanctl --load-all
   ```
2. Initiate the tunnel:
   ```
   sudo swanctl --initiate --ike tunnel
   ```
3. Check status:
   ```
   sudo swanctl --list-sas
   ```
4. Terminate:
   ```
   sudo swanctl --terminate --ike tunnel
   ```

### Looking Ahead
I'm exploring ways to improve Network Manager integration for GNOME users—something as user-friendly as OpenConnect. Plus, it's high time for open-source SAML authentication tools; we've relied on community projects long enough—let's contribute back!

If you're a Linux user in cybersecurity, give StrongSwan a shot for your IPsec needs. Have you faced similar VPN woes or tried this setup? Share your thoughts in the comments—I'd love to discuss and collaborate!

  
