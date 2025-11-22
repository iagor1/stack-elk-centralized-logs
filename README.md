# Stack ELK - Centralized Logging System

A distributed logging solution using Elasticsearch, Kibana, and Fluent Bit for collecting and analyzing logs from remote Docker containers.

## 📋 Overview

This project implements a centralized logging infrastructure where:
- **Elasticsearch + Kibana** run on a central server for log storage and visualization
- **Fluent Bit** runs on application servers to collect and forward logs
- **HAProxy** provides secure HTTPS access and routing to Kibana and Elasticsearch



## 🚀 Features

- **Centralized Log Collection**: Gather logs from multiple remote servers
- **Docker Integration**: Automatically collect logs from all Docker containers
- **Scalable**: Add new log sources by deploying Fluent Bit on additional servers
- **Rate Limiting**: Protection against abusive requests
- **IP Blacklisting**: Block malicious traffic

## 📁 Project Structure

```
.
├── docker-compose.yml          # Elasticsearch, Kibana, Metricbeat
├── metricbeat.yml             # Metricbeat configuration
├── hpx/
│   ├── docker-compose.yml     # HAProxy service
│   ├── haproxy.cfg           # HAProxy configuration
│   ├── blacklist.acl         # Blocked IP addresses
│   └── ssl/                  # SSL certificates
└── fluent-bit/
    ├── docker-compose.yml     # Fluent Bit service (remote servers)
    └── fluent-bit.conf       # Fluent Bit configuration
```

## 🔧 How It Works

### Central Server (Elasticsearch + Kibana)

- **Elasticsearch** stores and indexes all incoming logs
- **Kibana** provides a web interface for searching and visualizing logs
- **Metricbeat** monitors system metrics and Elasticsearch health
- **HAProxy** routes traffic:
   - `kibana.mydomain.com` → Kibana UI
   - DDOS protection

### Application Servers (Fluent Bit)

- **Fluent Bit** tails Docker container logs from `/var/lib/docker/containers/`
- Parses JSON log entries automatically
- Extracts fields like `statusCode`, `method`, `uri`, etc.

## 📦 Installation

### Prerequisites

- Docker and Docker Compose installed on all servers
- Domain names pointing to your central server
- SSL certificate for HTTPS (Let's Encrypt recommended)

### Central Server Setup

1. Clone the repository on your central server:

2. Configure environment variables in `.env` file:
   ```
   - ELASTIC_PASSWORD=your_secure_password
   - KIBANA_PASSWORD=your_kibana_password
   ```

3. Update `hpx/haproxy.cfg` with your domain names:
   ```
   acl is_kibana hdr(host) -i kibana.yourdomain.com
   ```

4. Place your SSL certificate in `hpx/ssl/`:
   ```bash
   cat fullchain.pem privkey.pem > hpx/ssl/certificate.pem
   ```

5. Start Haproxy, Elastic and Kibana

### Application Server Setup (Fluent Bit)

1. Copy the `fluent-bit` directory to your application server

2. Update the OUTPUT in `fluent-bit/fluent-bit.conf`:
   ```ini
   [OUTPUT]
       Name  es
       Host  elasticsearch.yourdomain.com
       Port  9200
       Index my_index
       HTTP_User elastic
       HTTP_Passwd your_secure_password
   ```

3. Start Fluent Bit:

4. Check logs are being sent:
   ```bash
   docker logs -f fluent-bit
   ```

## 🔍 Usage

### Accessing Kibana

1. Open your browser and navigate to `https://kibana.yourdomain.com`
2. Login with credentials (default: `elastic` / your password)
3. Go to **Discover** to view incoming logs
4. Create **Index Pattern**: `my_index-*` or `fluentbit-*`

### Viewing Logs

Logs are automatically parsed with fields:
- `host`: Container hostname
- `time`: Timestamp
- `client_ip`: Client IP address
- `statusCode`: HTTP status code (integer)
- `method`: HTTP method (GET, POST, etc.)
- `uri`: Request URI
- `backend`: Target backend service


## 🛡️ Security

### Features:

- Silently Dropping Requests
- Limiting Request Rates
- Slowloris protection
- Deny requests using http/1.0
- Deny requests without User-Agent header and some headless browsers

## 📊 Monitoring

View HAProxy stats at `http://your-server:8404/stats` (configure credentials in `haproxy.cfg`).

Metricbeat sends system metrics to Elasticsearch for monitoring cluster health.

## Evidence
<img width="1895" height="1020" alt="home kibana" src="https://github.com/user-attachments/assets/5d9c6eab-8c2a-4877-b497-754e10d26a0c" />
<img width="1890" height="1029" alt="Discover Kibana" src="https://github.com/user-attachments/assets/d3c70ec6-f349-4e9f-acc4-226b123b2c9c" />
<img width="1890" height="1029" alt="Discover Kibana 2" src="https://github.com/user-attachments/assets/9314d1ff-d6ba-4601-b086-72833195419d" />
