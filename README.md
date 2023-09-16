# Random_Codes

# Steps for premethues :
    Configure the targets for Prometheus to monitor
    Create queries to get the metrics about the target
    Determine the status of the targets
    Identify information about the targets and visualize it with graphs
    Instrument a Python Flask application to be monitored by Prometheus
---
    docker pull bitnami/node-exporter:latest
    docker pull bitnami/prometheus:latest
    docker network create monitor
    docker run -d --name node-exporter1 -p 9101:9100 --network monitor bitnami/node-exporter:latest
    docker run -d --name node-exporter2 -p 9102:9100 --network monitor bitnami/node-exporter:latest
    docker run -d --name node-exporter3 -p 9103:9100 --network monitor bitnami/node-exporter:latest
    docker ps | grep node-exporter
    touch /home/project/prometheus.yml
---
    docker run -d --name prometheus -p 9090:9090 --network monitor \
    -v $(pwd)/prometheus.yml:/opt/bitnami/prometheus/conf/prometheus.yml \
    bitnami/prometheus:latest
---
* You can check web on that port
* some premethues queries:
    node_cpu_seconds_total
    node_cpu_seconds_total{instance="node-exporter2:9100"}
    node_ipvs_connections_total
---
* docker stop node-exporter1
* nano Prometheus_Flask.py
* you need to deploy this code on the same docker network as Prometheus. 
* To do this, create a file named Dockerfile in the /home/project
* docker build -t Prometheus_Flask .
* docker run -d --name Prometheus_Flask -p 8081:8080 --network monitor Prometheus_Flask
* You can check web on that port
* docker restart prometheus
* curl localhost:8081
* curl localhost:8081/home
* curl localhost:8081/contact
* Use these queries to check python app:
* flask_http_request_duration_seconds_bucket
* flask_http_request_total
* process_virtual_memory_bytes
---
# Steps for Grafana :

    Deploy Prometheus to OpenShift
    Deploy Grafana to OpenShift
    Connect Prometheus as a datasource for Grafana
    Create a dashboard with Grafana
