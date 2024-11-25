# Linux-Assignment-3-part-1

# Static System Information Generator

This project sets up a Bash script to generate a static `index.html` file containing system information. The file is served by an Nginx web server and secured with a firewall using UFW. The system is automated using a systemd service and timer to run daily at 5:00 AM.

## Features

- Generates an HTML page with system information such as:
  - Kernel release
  - Operating system name
  - Current date
  - Number of installed packages
  - System uptime
  - Disk usage (specific to `webgen`)
- Automates the script execution using systemd service and timer.
- Serves the HTML page using an Nginx on port 80.
- Secures the server with UFW by:
  - Allowing SSH and HTTP traffic
  - Enabling SSH rate limiting
- Provides setup instructions for a new server.

## Prerequisites

- Basic bash scripting.
- An Arch Linux server.
- Root access or `sudo` privileges.
- Installed packages:
  - `nginx`
  - `ufw`
  - `git`
  Can be installed with `sudo pacman -S nginx ufw git`.

### Step 1: Create a System User

1. Create a non-login system user `webgen` with a home directory:

`sudo useradd -r -m -d /var/lib/webgen -s /usr/sbin/nologin webgen`

2. Ensure the user owns its directory:
`sudo chown -R webgen:webgen /var/lib/webgen`

3. Create the required directories:
```bash
mkdir -p /var/lib/webgen/bin
mkdir -p /var/lib/webgen/HTML
```

### Step 2: Clone the Repository

1. Clone this repository into `/var/lib/webgen/bin/`:
```bash
sudo mkdir -p /var/lib/webgen/bin/
sudo -u webgen git clone  https://github.com/ChelsieSalome/Linux-Assignment-3-part-1.git /var/lib/webgen/bin/
```

2. Ensure the following files from the repository are in the correct locations on the server:
    * **`generate_index`** : Script that generates the index.html file. 
        *Already located in `/var/lib/webgen/bin/` after cloning the repository. This script generates the `index.html` file.*
    * **`generate-index.service`**: Copy this file to `/etc/systemd/system/`:  
```bash
sudo cp /var/lib/webgen/bin/generate-index.service /etc/systemd/system/
```
    * **`generate-index.timer`**: Copy this file to **/etc/systemd/system/**
        `sudo cp /var/lib/webgen/bin/generate-index.timer /etc/systemd/system/`
    - **`nginx.conf`**: This file configures the Nginx server. If there is already an `/etc/nginx/nginx.conf` file on your system, follow these steps:

    1. **Back up the Existing Configuration**:
     ```sudo cp /etc/nginx/nginx.conf /etc/nginx/nginx.conf.bak```

    2. **Compare and Merge Changes**:
        Open the existing `/etc/nginx/nginx.conf` and the repository's `nginx.conf` to identify differences. Ensure the following changes are made:
        
        - Replace the `user` directive (usually near the top of the file) with:
        ```user webgen;```
        - Ensure the `http` block includes the following at the end:
        ```include /etc/nginx/sites-enabled/*;```

    3. **Test the Configuration**:
        After merging the changes, test the Nginx configuration:
        ```sudo nginx -t```

    4. **Restart Nginx**:
        Restart the Nginx service to apply changes:
        ```sudo systemctl restart nginx ```

    5. **If the Backup is Needed**:
        If something goes wrong, restore the original configuration:
        ```sudo cp /etc/nginx/nginx.conf.bak /etc/nginx/nginx.conf```
        ```sudo systemctl restart nginx```

    * **`generate-index.conf`**: Copy this file to **/etc/nginx/sites-available/**:
        `sudo cp /var/lib/webgen/bin/generate-index.conf /etc/nginx/sites-available/`
    

### Step 3: Configure Nginx

1. Enable the `generate-index.conf` server block:
`sudo ln -s /etc/nginx/sites-available/generate-index.conf /etc/nginx/sites-enabled/`

3. Test and reload Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### Step 4: Enable and Start Systemd Service and Timer

1. Reload systemd:
`sudo systemctl daemon-reload`

2. Enable the service and timer:
```bash
sudo systemctl enable --now generate-index.service
sudo systemctl enable --now generate-index.timer
```


### Step 5: Configure UFW

Allow SSH and HTTP traffic and enable ufw:

```bash
sudo pacman -S ufw
sudo ufw allow ssh
sudo ufw allow http
sudo ufw limit ssh
sudo ufw enable
```
Check the status:
`sudo ufw status verbose`



