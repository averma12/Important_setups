# Mosquitto Setup & Running Commands (Ubuntu)

This document summarizes the steps to install, configure, and run the Mosquitto MQTT broker on an Ubuntu server, including WebSocket support and basic password authentication.

## 1. Installation

First, update your package list and install the Mosquitto broker and command-line client tools. The `-y` flag automatically confirms the installation.

```bash
sudo apt update
sudo apt install mosquitto mosquitto-clients -y
```

2. ConfigurationWe will place custom configurations in a separate file to avoid conflicts with the main config file during package upgrades.a) Create Custom Configuration FileCreate and edit a new configuration file:sudo nano /etc/mosquitto/conf.d/custom.conf
Paste the following content into this file. Adjust ports or paths if needed.# --- Listeners ---
# Default MQTT listener on standard port 1883
listener 1883

# WebSocket listener on port 9001
listener 9001
protocol websockets

# --- Security (VERY Important on a Server!) ---
# Disallow connections without authentication
allow_anonymous false

# Specify the location of the password file
# (We will create this file next)
password_file /etc/mosquitto/passwd

# --- Persistence (Optional but recommended) ---
# Save message data across restarts
persistence true
persistence_location /var/lib/mosquitto/

# --- Logging (Optional but recommended) ---
log_dest file /var/log/mosquitto/mosquitto.log
# Optional: Add more log types if needed for debugging
# log_type error
# log_type warning
# log_type notice
# log_type information
# log_type all
# log_type debug
Save and close the file (in nano: Ctrl+O, Enter, Ctrl+X).b) Remove Duplicate Settings from Main Config (If Necessary)Mosquitto fails if settings like persistence_location or log_dest are defined in both the main file and a conf.d file. Edit the main file and comment out any duplicates you added to custom.conf.sudo nano /etc/mosquitto/mosquitto.conf

Find lines like persistence_location ... or log_dest file ... and add a # at the beginning to comment them out:# persistence_location /var/lib/mosquitto/
# log_dest file /var/log/mosquitto/mosquitto.log
Save and close the file.c) Create Password FileCreate the password file and add your first user. Replace <your_username> with the actual username you want to use. You will be prompted to enter and confirm a password.# Use '-c' ONLY for the very first user to create the file
sudo mosquitto_passwd -c /etc/mosquitto/passwd <your_username>
To add more users later, omit the -c:# sudo mosquitto_passwd /etc/mosquitto/passwd <another_username>

3. Running and Managing the ServiceMosquitto runs as a systemd service.a) Restart Mosquitto to Apply Changessudo systemctl restart mosquitto
b) Check StatusVerify that the service started correctly and is running.sudo systemctl status mosquitto.service
(Look for Active: active (running). Press q to exit.)c) Enable Service on Boot (Optional but Recommended)Ensures Mosquitto starts automatically when the server boots. It might already be enabled by default.sudo systemctl enable mosquitto
d) Stop ServiceTo stop the broker manually:sudo systemctl stop mosquitto

4. Firewall Configuration (Example using UFW)If you use a firewall like UFW, allow traffic on the ports Mosquitto uses.# Allow standard MQTT port
sudo ufw allow 1883/tcp

# Allow WebSocket port
sudo ufw allow 9001/tcp

# Check status (and enable if needed)
# sudo ufw enable
sudo ufw status
Remember to also configure any cloud provider firewalls (AWS Security Groups, GCP Firewall Rules, etc.) if applicable.5. Testing / Debugginga) Check LogsIf the service fails to start, check the detailed logs:# View recent logs (use arrow keys, Shift+G for end, q to quit)
sudo journalctl -xeu mosquitto.service

# Dump all logs for the service without paging
sudo journalctl -eu mosquitto.service --no-pager
b) Test Connection (using mosquitto_sub/pub)From the server itself or another machine with mosquitto-clients installed:# Subscribe (will ask for password)
mosquitto_sub -h <your_server_ip_or_domain> -p 1883 -t "test/topic" -u "<your_username>" -P "<your_password>" -v

# Publish from another terminal
mosquitto_pub -h <your_server_ip_or_domain> -p 1883 -t "test/topic" -m "Hello from command line" -u "<your_username>" -P "<your_password>"
6. Final NotesRemember to replace placeholders like <your_username>, <your_password>, and <your_server_ip_or_domain> with your actual values.Update your client applications (frontend UI, backend listener) to connect to the correct server address, port (9001 for WS, 1883 for MQTT), and use the username/password you