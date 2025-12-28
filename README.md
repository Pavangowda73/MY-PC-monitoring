# MY-PC-monitoring

You can monitor your Windows laptop (CPU, RAM, disk, network, etc.) using Prometheus + Grafana by installing Windows Exporter (formerly WMI Exporter).

Below is a complete, step-by-step guide from scratch, written assuming Windows 10/11.

# Local setup

Architecture (Simple)

Windows Laptop <br>
 ├── windows_exporter  → exposes metrics on :9182 <br>
 ├── Prometheus        → scrapes metrics on :9090 <br>
 └── Grafana           → visualizes metrics on :3000 <br>


# STEP 1: Install Windows Exporter (Metrics Agent)

Windows Exporter exposes CPU, memory, disk, etc. in Prometheus format.

1. Download Installer

👉 https://github.com/prometheus-community/windows_exporter/releases

Download the .msi file
Example:

windows_exporter-0.xx.x-amd64.msi

2. Install Windows Exporter

Double-click the .msi

During install:

Collectors: select

cpu
memory
logical_disk
os
system
net


Finish installation

3. Verify Windows Exporter

Open browser and visit:

http://localhost:9182/metrics


✅ You should see a long list of metrics like:

windows_cpu_time_total
windows_memory_available_bytes

# STEP 2: Install Prometheus on Windows
1. Download Prometheus

👉 https://prometheus.io/download/

Download:

prometheus-2.xx.x.windows-amd64.zip

2. Extract Prometheus

Extract to:

C:\prometheus


Folder should contain:

prometheus.exe
promtool.exe
prometheus.yml

3. Configure Prometheus

Open:

C:\prometheus\prometheus.yml


Replace content with this 👇

global:
  scrape_interval: 5s

scrape_configs:
  - job_name: "windows"
    static_configs:
      - targets: ["localhost:9182"]


Save the file.

4. Start Prometheus

Open Command Prompt as Administrator:

cd C:\prometheus
prometheus.exe


Open browser:

http://localhost:9090


Check:

Status → Targets

Windows target should be UP ✅

# STEP 3: Install Grafana on Windows
1. Download Grafana

👉 https://grafana.com/grafana/download

Download:

Windows Installer (64-bit)

2. Install Grafana

Keep default options

Grafana will install as a Windows service

3. Start Grafana

Open browser:

http://localhost:3000


Default login:

Username: admin
Password: admin


(Change password when prompted)

# STEP 4: Add Prometheus as Grafana Data Source

Go to ⚙️ Settings → Data Sources

Click Add data source

Select Prometheus

URL:

http://localhost:9090


Click Save & Test

✅ Data source should be working

# STEP 5: Import Windows Monitoring Dashboard
Recommended Pre-built Dashboard

Go to + → Import

Enter Dashboard ID:

14694


(Official Windows Exporter dashboard)

Select Prometheus as data source

Click Import

🎉 You will now see:

CPU usage

Memory usage

Disk usage

Network traffic

System uptime



# Cloud setup

Architecture
Windows Laptop
 └── windows_exporter :9182 <br>
       ▲ <br>
       │ Cloudflare Tunnel (outbound only) <br>
AWS EC2 <br>
 └── Prometheus → Grafana <br>


✔ No public IP
✔ No router changes
✔ No domain
✔ Secure

🚀 COMPLETE SETUP (STEP-BY-STEP)
# STEP 1: Download Cloudflared (Windows)

Download:

 # Cloud fare setup.
https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/install-and-setup/installation/

Download:

cloudflared-windows-amd64.exe


Rename it to:

cloudflared.exe


Move it to:

C:\Program Files\Cloudflare\

✅ STEP 2: Test cloudflared using FULL PATH (IMPORTANT)

Open PowerShell and run this exact command:

"C:\Program Files\Cloudflare\cloudflared.exe" --version

🔹 Result?

✅ If this works → PATH issue (we fix next)

❌ If this fails → file is missing or blocked

👉 Tell me what output you get

✅ STEP 3: Fix PATH Permanently (Manual & Guaranteed)

Press Win + R

Type:

sysdm.cpl


Open Advanced tab

Click Environment Variables

Under System Variables

Select Path

Click Edit

Click New

Paste:

C:\Program Files\Cloudflare\


Click OK → OK → OK

✅ STEP 4: Restart EVERYTHING

⚠️ This is mandatory

Close PowerShell

Close Command Prompt

(Optional but safest) Restart your PC

✅ STEP 5: Verify Again

Open NEW PowerShell and run:

cloudflared --version


Expected:

cloudflared version 2025.x.x

# ubuntu 22.04 (t2.medium)

launch the insatnce open the ports 9090, 3000

Install Prometheus and Grafana:

Set up Prometheus and Grafana to monitor your application.

Installing Prometheus:

First, create a dedicated Linux user for Prometheus and download Prometheus:

sudo useradd --system --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
Extract Prometheus files, move them, and create directories:

tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
cd prometheus-2.47.1.linux-amd64/
sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
Set ownership for directories:

sudo chown -R prometheus:prometheus /etc/prometheus/ /data/
Create a systemd unit configuration file for Prometheus:

sudo nano /etc/systemd/system/prometheus.service
Add the following content to the prometheus.service file:

[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
Here's a brief explanation of the key parts in this prometheus.service file:

User and Group specify the Linux user and group under which Prometheus will run.

ExecStart is where you specify the Prometheus binary path, the location of the configuration file (prometheus.yml), the storage directory, and other settings.

web.listen-address configures Prometheus to listen on all network interfaces on port 9090.

web.enable-lifecycle allows for management of Prometheus through API calls.

Enable and start Prometheus:

sudo systemctl enable prometheus
sudo systemctl start prometheus
Verify Prometheus's status:

sudo systemctl status prometheus
You can access Prometheus in a web browser using your server's IP and port 9090:

http://<your-server-ip>:9090

Configure Prometheus Plugin Integration:

Integrate Jenkins with Prometheus to monitor the CI/CD pipeline.

Prometheus Configuration:

To configure Prometheus to scrape metrics from Node Exporter and Jenkins, you need to modify the prometheus.yml file. Here is an example prometheus.yml configuration for your setup:

```
scrape_configs:
  - job_name: 'windows-laptop'
    scheme: https
    metrics_path: /metrics
    static_configs:
      - targets:
          - calm-mouse-9182.trycloudflare.com
```

Make sure to replace <your-jenkins-ip> and <your-jenkins-port> with the appropriate values for your Jenkins setup.

Check the validity of the configuration file:

promtool check config /etc/prometheus/prometheus.yml
Reload the Prometheus configuration without restarting:

curl -X POST http://localhost:9090/-/reload
You can access Prometheus targets at:

http://<your-prometheus-ip>:9090/targets


####Grafana

Install Grafana on Ubuntu 22.04 and Set it up to Work with Prometheus

Step 1: Install Dependencies:

First, ensure that all necessary dependencies are installed:

sudo apt-get update
sudo apt-get install -y apt-transport-https software-properties-common
Step 2: Add the GPG Key:

Add the GPG key for Grafana:

wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
Step 3: Add Grafana Repository:

Add the repository for Grafana stable releases:

echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
Step 4: Update and Install Grafana:

Update the package list and install Grafana:

sudo apt-get update
sudo apt-get -y install grafana
Step 5: Enable and Start Grafana Service:

To automatically start Grafana after a reboot, enable the service:

sudo systemctl enable grafana-server
Then, start Grafana:

sudo systemctl start grafana-server
Step 6: Check Grafana Status:

Verify the status of the Grafana service to ensure it's running correctly:

sudo systemctl status grafana-server
Step 7: Access Grafana Web Interface:

Open a web browser and navigate to Grafana using your server's IP address. The default port for Grafana is 3000. For example:

http://<your-server-ip>:3000

You'll be prompted to log in to Grafana. The default username is "admin," and the default password is also "admin."


