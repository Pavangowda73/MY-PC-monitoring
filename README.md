# MY-PC-monitoring

You can monitor your Windows laptop (CPU, RAM, disk, network, etc.) using Prometheus + Grafana by installing Windows Exporter (formerly WMI Exporter).

Below is a complete, step-by-step guide from scratch, written assuming Windows 10/11.


Architecture (Simple)

Windows Laptop <br>
 ├── windows_exporter  → exposes metrics on :9182 <br>
 ├── Prometheus        → scrapes metrics on :9090 <br>
 └── Grafana           → visualizes metrics on :3000 <br>


 STEP 1: Install Windows Exporter (Metrics Agent)

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

🔹 STEP 2: Install Prometheus on Windows
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

🔹 STEP 3: Install Grafana on Windows
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

🔹 STEP 4: Add Prometheus as Grafana Data Source

Go to ⚙️ Settings → Data Sources

Click Add data source

Select Prometheus

URL:

http://localhost:9090


Click Save & Test

✅ Data source should be working

🔹 STEP 5: Import Windows Monitoring Dashboard
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

