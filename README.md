# Linux-Assignment-3-part-1

# Static System Information Generator

This project sets up a Bash script to generate a static `index.html` file containing system information. The file is served by an Nginx web server and secured with a firewall using UFW. The system is automated using a systemd service and timer to run daily at 5:00 AM.

## Features

- Generates an HTML page with system information such as:
  - Kernel release
  - Operating system name
  - Current date
  - Number of installed packages
- Automates the script execution using systemd service and timer.
- Serves the HTML page using an Nginx web server.
- Secures the server with UFW by:
  - Allowing SSH and HTTP traffic
  - Enabling SSH rate limiting
- Provides setup instructions for a new server.

## Prerequisites

- An Arch Linux server.
- Root access or `sudo` privileges.
- Installed packages:
  - `nginx`
  - `ufw`
  - `systemd` (pre-installed on most systems)
  - `git`

### Step 1: Create a System User

Create a non-login system user `webgen` with a home directory:

`sudo useradd -r -m -d /var/lib/webgen -s /usr/sbin/nologin webgen`

Ensure the user owns its directory:
`sudo chown -R webgen:webgen /var/lib/webgen`

### Step 2: Clone the Repository

Clone this repository into `/var/lib/webgen/bin/`:
```bash
sudo mkdir -p /var/lib/webgen/bin/
sudo git clone <repository-link> /var/lib/webgen/bin/
```

### Step 3: Configure Systemd Service and Timer

Move the service and timer files to the appropriate locations:

Move the service and timer files to the appropriate locations:
```bash
sudo cp generate-index.service /etc/systemd/system/
sudo cp generate-index.timer /etc/systemd/system/
```
Enable and start the timer:
```bash
sudo systemctl enable generate-index.timer
sudo systemctl start generate-index.timer
```

### Step 4: Configure Nginx

1. Modify `nginx.conf` to run Nginx as the `webgen` user:
    - Locate `user` directive in `/etc/nginx/nginx.conf` and set it to:
      ```nginx
      user webgen;
      ```

2. Create a new server block configuration file:
    - Copy `server-block.conf` to `/etc/nginx/sites-available/` and create a symlink:
```bash
    sudo ln -s /etc/nginx/sites-available/server-block.conf /etc/nginx/sites-enabled/
```

3. Test and reload Nginx:

```bash
sudo nginx -t
sudo systemctl reload nginx
```

### Step 5: Configure UFW

Install and configure UFW to allow SSH and HTTP traffic:

```bash
sudo pacman -S ufw
sudo ufw allow ssh
sudo ufw allow http
sudo ufw limit ssh
sudo ufw enable
```
Check the status:
`sudo ufw status`



