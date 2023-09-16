# Random_Codes

# Steps for Prometheus :
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
---
    node_cpu_seconds_total
    node_cpu_seconds_total{instance="node-exporter2:9100"}
    node_ipvs_connections_total
---
    docker stop node-exporter1
    nano Prometheus_Flask.py
---
* you need to deploy this code on the same docker network as Prometheus. 
* To do this, create a file named Dockerfile in the /home/project
---
    docker build -t Prometheus_Flask .
    docker run -d --name Prometheus_Flask -p 8081:8080 --network monitor Prometheus_Flask
---
* You can check web on that port
---
    docker restart prometheus
    curl localhost:8081
    curl localhost:8081/home
    curl localhost:8081/contact
---
* Use these queries to check python app:
---
    flask_http_request_duration_seconds_bucket
    flask_http_request_total
    process_virtual_memory_bytes
---
# Steps for Grafana :

    Use https://github.com/FiredPhoenixIR/ondaw-prometheus-grafana-lab
    Deploy Prometheus to OpenShift
    Deploy Grafana to OpenShift
    Connect Prometheus as a datasource for Grafana
    Create a dashboard with Grafana

* Use the oc create deployment command to deploy 3 node exporters
---
    oc create deployment node-exporter1 --port=9100 --image=bitnami/node-exporter:latest
    oc create deployment node-exporter2 --port=9100 --image=bitnami/node-exporter:latest
    oc create deployment node-exporter3 --port=9100 --image=bitnami/node-exporter:latest
---
* use the oc expose command to create services that expose the 3 node exporters so that Prometheus can communicate with them
---
    oc expose deploy node-exporter1 --port=9100 --type=ClusterIP
    oc expose deploy node-exporter2 --port=9100 --type=ClusterIP
    oc expose deploy node-exporter3 --port=9100 --type=ClusterIP
---
* Check that the pods are up and running
---
    oc get pods
---
* Step 2: Deploy Prometheus
* While normally you would modify configuration files to configure Prometheus, this is not the case for Kubernetes.
* For a Kubernetes environment the proper appoach is to use a ConfigMap
* This makes it easy to change the configuraton later. You will find the configuration files from which to make the ConfigMap in a folder named ./config.
* You will also need Kubernetes manifests to describe the Prometheus deployment and to link the ConfigMap with the Prometheus.
* These manifests can be found in the ./deploy folder.
---
    oc create configmap prometheus-config \
       --from-file=prometheus=./config/prometheus.yml \
       --from-file=prometheus-alerts=./config/alerts.yml
    oc apply -f deploy/prometheus-deployment.yaml
    oc get pods -l app=prometheus
---
* Step 3: Deploy Grafana
* use the oc expose command to expose the grafana service with an OpenShift route. Routes are a special feature of OpenShift that makes it easier to use for developers.
---
    oc apply -f deploy/grafana-deployment.yaml
    oc expose svc grafana
---
* Use the following oc patch command to enable TLS and the https:// protocol for the route.
---
    oc patch route grafana -p '{"spec":{"tls":{"termination":"edge","insecureEdgeTerminationPolicy":"Redirect"}}}'
---
* Step 4: Log in to Grafana
* Use the oc describe command along with grep to extract the URL of the Requested Host for the grafana route:
---
    oc describe route grafana | grep "Requested Host:"
---
* Check URL and define source in Grafana UI adding URL : http://prometheus:9090
* Step 6: Create a Dashboard
* You will use a precreated template provided by Grafana Dashboard. This template is identified by the id 1860.
