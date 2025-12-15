# Centralized Security Logging with ELK Stack

## Overview
This module demonstrates the deployment of a centralized logging architecture using the **ELK Stack** (Elasticsearch, Logstash, Kibana). The objective is to ingest raw system logs, parse them into structured JSON, and visualize threat data for real-time monitoring.



## Environment & Prerequisites
- **Host System:** Ubuntu 20.04 LTS (4GB+ RAM recommended)
- **Stack Components:** Elasticsearch 7.x, Logstash 7.x, Kibana 7.x
- **Runtime:** OpenJDK 11 / Java 8+

## Deployment Architecture

### Phase 1: Infrastructure Provisioning (Elasticsearch)
Deploy the search engine core to index and store security telemetry.

1.  **Install & Configure:**
    ```bash
    wget -qO - [https://artifacts.elastic.co/GPG-KEY-elasticsearch](https://artifacts.elastic.co/GPG-KEY-elasticsearch) | sudo apt-key add -
    echo "deb [https://artifacts.elastic.co/packages/7.x/apt](https://artifacts.elastic.co/packages/7.x/apt) stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list
    sudo apt-get update && sudo apt-get install elasticsearch
    ```

2.  **Service Initialization:**
    ```bash
    sudo systemctl enable --now elasticsearch
    # Verify cluster health
    curl -X GET "localhost:9200/_cluster/health?pretty"
    ```

### Phase 2: Pipeline Configuration (Logstash)
Configure the ingestion pipeline to normalize log data before indexing.

1.  **Install Logstash:**
    ```bash
    sudo apt-get install logstash
    ```

2.  **Pipeline Definition (`/etc/logstash/conf.d/security-pipeline.conf`):**
    This configuration ingests system logs and outputs them to the local Elasticsearch cluster.
    ```ruby
    input {
      file {
        path => "/var/log/syslog"
        type => "syslog"
        start_position => "beginning"
      }
    }
    filter {
      if [type] == "syslog" {
        grok {
          match => { "message" => "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
        }
      }
    }
    output {
      elasticsearch {
        hosts => ["localhost:9200"]
        index => "security-logs-%{+YYYY.MM.dd}"
      }
    }
    ```

3.  **Pipeline Execution:**
    ```bash
    sudo systemctl restart logstash
    ```

### Phase 3: Visualization & Dashboards (Kibana)
Deploy the frontend interface for threat hunting and data visualization.



1.  **Install Kibana:**
    ```bash
    sudo apt-get install kibana
    sudo systemctl enable --now kibana
    ```

2.  **Dashboard Configuration:**
    * **Access:** Navigate to `http://localhost:5601`.
    * **Index Pattern:** Create a pattern for `security-logs-*` to link the Elasticsearch indices.
    * **Visualizations:**
        * *Vertical Bar:* Top 10 Failed SSH Logins (Source IP).
        * *Pie Chart:* Sudo Usage Distribution by User.
        * *Line Graph:* Log Volume over Time (Anomaly Detection).

## Outcome
This deployment establishes a functional SIEM-like capability, allowing for the correlation of disparate log events and the visual identification of security anomalies.
