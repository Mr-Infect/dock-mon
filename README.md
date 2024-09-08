# Docker Monitoring with Python, Prometheus, and Grafana

This project sets up a monitoring system for Docker containers using Python, Prometheus, and Grafana. The solution is designed to be lightweight and suitable for low-config systems.

## Project Overview

- **Python**: Collects custom metrics from Docker containers.
- **Prometheus**: Scrapes and stores metrics data from Python and cAdvisor.
- **Grafana**: Visualizes the metrics data with interactive dashboards.

## Requirements

- Ubuntu-based system
- Docker
- Python 3
- Prometheus
- Grafana

## Setup Steps

### 1. Setup Directory Structure

Create a project directory and subdirectories:

```bash
mkdir ~/docker-monitoring
cd ~/docker-monitoring
mkdir prometheus grafana cadvisor python_monitoring

2. Install Docker

Ensure Docker is installed on your system:

bash

sudo apt update
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker

3. Setup cAdvisor

Run cAdvisor to collect Docker container metrics:

bash

docker run -d \
  --name=cadvisor \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:ro \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --volume=/cgroup:/cgroup:ro \
  -p 8080:8080 \
  google/cadvisor:latest

4. Setup Prometheus

    Create the Prometheus Configuration File

    Create the configuration file:

    bash

nano ~/docker-monitoring/prometheus/prometheus.yml

Add the following content:

yaml

global:
  scrape_interval: 5s

scrape_configs:
  - job_name: 'cadvisor'
    static_configs:
      - targets: ['localhost:8080']

  - job_name: 'python-monitoring'
    static_configs:
      - targets: ['localhost:5000']

Run Prometheus Container

Replace /absolute/path/to/prometheus.yml with the full path:

bash

    sudo docker run -d \
      -p 9090:9090 \
      --name prometheus \
      -v /home/your_username/docker-monitoring/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml \
      prom/prometheus

5. Create Python Docker Monitoring Script

    Install Required Python Libraries

    bash

sudo apt install python3-pip
pip3 install docker Flask

Create the Python Script

Create the script:

bash

nano ~/docker-monitoring/python_monitoring/docker_monitor.py

Add the following content:

python

from flask import Flask
import docker

app = Flask(__name__)
client = docker.from_env()

@app.route('/metrics')
def docker_metrics():
    metrics = []
    for container in client.containers.list():
        stats = container.stats(stream=False)
        cpu_usage = calculate_cpu_percentage(stats)
        mem_usage = calculate_memory_percentage(stats)
        metrics.append(f'container_cpu_usage{{name="{container.name}"}} {cpu_usage}')
        metrics.append(f'container_mem_usage{{name="{container.name}"}} {mem_usage}')
    return '\n'.join(metrics)

def calculate_cpu_percentage(stats):
    cpu_delta = stats['cpu_stats']['cpu_usage']['total_usage'] - stats['precpu_stats']['cpu_usage']['total_usage']
    system_cpu_delta = stats['cpu_stats']['system_cpu_usage'] - stats['precpu_stats']['system_cpu_usage']
    cpu_percentage = (cpu_delta / system_cpu_delta) * len(stats['cpu_stats']['cpu_usage']['percpu_usage']) * 100
    return cpu_percentage

def calculate_memory_percentage(stats):
    mem_usage = stats['memory_stats']['usage']
    mem_limit = stats['memory_stats']['limit']
    mem_percentage = (mem_usage / mem_limit) * 100
    return mem_percentage

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5000)

Run the Python Script

bash

    python3 ~/docker-monitoring/python_monitoring/docker_monitor.py

6. Setup Grafana

    Run Grafana Container

    bash

    docker run -d \
      -p 3000:3000 \
      --name=grafana \
      grafana/grafana

    Configure Grafana
        Access Grafana at http://localhost:3000 (default credentials: admin/admin).
        Add Prometheus as a Data Source:
            Go to Configuration > Data Sources.
            Select Prometheus and set the URL to http://localhost:9090.
        Import a Docker Monitoring Dashboard:
            Go to Dashboard > Import.
            Enter Dashboard ID 893 and click Load.

7. Automate Python Script and Docker Containers

    Create a systemd Service for the Python Script

    bash

sudo nano /etc/systemd/system/docker_monitor.service

Add the following content:

ini

[Unit]
Description=Docker Monitor
After=network.target

[Service]
ExecStart=/usr/bin/python3 /home/your_username/docker-monitoring/python_monitoring/docker_monitor.py
Restart=always
User=your_username
Environment=PYTHONUNBUFFERED=1

[Install]
WantedBy=multi-user.target

Enable and Start the Service

bash

    sudo systemctl daemon-reload
    sudo systemctl enable docker_monitor.service
    sudo systemctl start docker_monitor.service

Final Checks

    Prometheus: http://localhost:9090
    Grafana: http://localhost:3000
    cAdvisor: http://localhost:8080
    Python Monitoring: http://localhost:5000/metrics

This setup provides a comprehensive monitoring solution for Docker containers, combining Python, Prometheus, and Grafana for real-time metrics collection and visualization.
