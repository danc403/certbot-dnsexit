# Certbot DNSExit 

The `new_domain.py` script automates the entire process of obtaining Let's Encrypt certificates using Certbot with the DNS-01 challenge for a DNSExit domain.  It handles certificate requests, *including automatic DNS record manipulation through built-in hook scripts*, and includes a deploy hook for updating symlinks and reloading Apache.  This script makes setting up SSL certificates for your DNSExit domain incredibly easy.

## Overview

This script drastically simplifies the process of getting and renewing SSL certificates for your DNSExit domain.  It uses Certbot in manual mode with the DNS-01 challenge, *completely automating the DNS record updates through included and pre-configured hook scripts*.  This script focuses on the *initial setup* of a new domain. Once configured, renewals should be fully automatic (see "Automated Renewals" section).

## Prerequisites

*   A DNSExit account.
*   Certbot installed on your system. (e.g., `dnf install certbot` on Fedora/RHEL-based systems, `apt install certbot` on Debian/Ubuntu-based systems). It is recommended to use the Certbot packages provided by your distribution.
*   Python 3 installed.
*   requests
*   dnspython
(python -m pip install requests dnspython


## Installation

1.  Clone this repository: `git clone https://github.com/danc403/certbot-dnsexit.git`
2.  Copy the contents of the cloned repository into `/etc/letsencrypt/web-hooks`.  This includes `new_domain.py`, `certbot_dnsexit.json`, and the `pre`, `post`, and `deploy` directories containing the *already configured* hook scripts.
3.  Ensure the `new_domain.py` script is executable: `chmod +x /etc/letsencrypt/web-hooks/new_domain.py`

## Configuration (`certbot_dnsexit.json`)

Edit the included `certbot_dnsexit.json` file located in `/etc/letsencrypt/web-hooks`.  You only need to modify the following values:

```json
{
  "DNSEXIT_domain": "your_domain.com",  // Your DNSExit domain (e.g., example.com)
  "domain_email": "[email address removed]", // Your email address
  "API_KEY": "YOUR_API_KEY" // Your DNSExit API Key
}

Do not modify the DNS_SERVER value unless you are using a secondary DNS server provided by DNSExit.  The default value (ns1.dnsexit.com) is usually correct.
Usage
/etc/letsencrypt/web-hooks/new_domain.py [-w | --wildcard]
• -w or --wildcard: Use this option to request a wildcard certificate (e.g., *.yourdomain.com). If this option is omitted, a certificate for the base domain (e.g., yourdomain.com) will be requested.
How it works (Fully Automated!)
1. You run the new_domain.py script.
2. The script reads the configuration from the included certbot_dnsexit.json file.
3. It constructs the certbot command, automatically including the paths to the already configured and included hook scripts.
4. It executes the certbot command.
5. Certbot automatically calls the included and configured pre-hook script to update the DNS records.
6. Certbot validates the domain ownership via DNS.
7. Certbot automatically calls the included and configured post-hook script to clean up the DNS records.
8. Certbot obtains the certificate.
9. Certbot automatically calls the included and configured deploy hook to update symlinks and reload Apache.
Logging
The script (and the hook scripts) log their activity.
On Linux: /var/log/dns_challenge.log
On Win and Mac: in your home directory. ~/dns_challenge.log

Automated Renewals
You'll need a mechanism for automated certificate renewals.  This is typically done using a cron job or systemd timer.  An example cron job entry would be:
0 0 1,15 * * /etc/letsencrypt/web-hooks/new_domain.py  # Run at midnight on the 1st and 15th of each month. 
Remember to adjust the timing as needed.  You can also include the -w flag if you are using a wildcard certificate.
* Newer certbot installations usually setup a timer automatically.

Troubleshooting
• Check the log files for any errors.
• Verify the accuracy of the configuration file, especially the API key.
