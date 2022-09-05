# Aqua Observability

A guide to monitoring your Aquasec deployment using observability tools such as Prometheus and Grafana 

## Introduction

You are a maintainer (DevSecOps/DevOps/SRE) of an Aquasec deployment on premises and you would like to better understand the resource usage of your deployment from the following components.

- Underlying Host/VM utilization  - separate to any Cloud specific monitoring tools such as AWS Cloudwatch, GCP Stack Driver or Azure Monitor etc.

- Aqua Console & container resource usage
- Aqua Gateway and Enforcers deployed.
- Aqua Databases hosted on PostgresDB (containerized or Hosted) 
- Image Scan statistics
- The number of different policies which are deployed
- User login & session details
## How does it work ?

The Aquasec console can expose a Prometheus metrics endpoint which provides a number of statistics which Prometheus can scrape. This in turn can be visualized by using a Grafana deployment and importing a number of graphs.

Here's a schematic of the components and how they integrate with each other.

<img src="./images/aqua-obs-2022-02-24-1540.png" alt_text="Aqua Prometheus & Grafana Observability">
<br>

If you are not familiar with Prometheus and Grafana, in short, it's basically a collection of monitoring tools that pulls real-time performance data from end-points (aka exporters) or hosts/clusters/applications, and stores this data over time, into a database. 

This data is made available for graphing tools to use - such as Grafana so that it can be visualized and the performance of these applications can be understood.

Think of this visually as a collection a dashboards which contain dials, graphs, stat & bar charts with data mapped to it for everything. 
 
 *i* Disclaimer: by using the deployment files and configuration you do so at your own risk and no support is implied by the contributor. *i*
## What does this all look like in real terms? 

Here are some screen shots of the kinds of data which are being collected & visualized.
<br>

Linux OS Node Exporter
<br>
<img src="./images/node-exporter-dashboard.png" width="30%" height="30%">
<br>
Aqua Console
<br>
<img src="./images/aqua-perf-dashboard.png" width="30%" height="30%">
<br>
PostgreSQL DB
<br>
<img src="./images/postgresql-perf-dashboard.png" width="30%" height="30%">
<br>
## What do I need to set this up?

1. Access to a Kubernetes cluster (or Docker) and a namespace for  'monitoring' or 'observability'. Using your own namespace name is possible also. Be sure to modify the deployment yaml where relevant.

2. A deployment of Prometheus and Grafana into the namespace mentioned above. 

To make the data persistent for Prometheus and Grafana I have create Persistent Volume Claims within the deployments for Prometheus and Grafana.
In the deployment section, that has been included the Kubernetes yaml files for everything.

3. Prometheus node exporter which can be found [here](https://github.com/prometheus/node_exporter). 
You should deploy this on the VM/hosts in your K8s cluster (if possible.) I deployed it onto my physical server which runs [microk8s](https://microk8s.io/). 

4. The PostgreSQL DB exporter & access to the Aqua PostgreSQL DB - obtained from [Prometheus's GitHub](https://github.com/prometheus-community/postgres_exporter). Again, deployed into the namespace to connect to the K8s Service name or K8s ClusterIP/LoadBalancer IP's which expose the AquaDB (Scalock) and Aqua Audit DB(SLK_Audit)
5. The Aquasec Prometheus endpoint token and FQDN for the Aquasec Console. i.e. https://aquasec-console-dev.mydomain.com
6. Patience and coffee :) 

# Deployment


## Prometheus Exporters

Firstly we will setup the exporters and check that the data is available for Prometheus to 'scrape'.

Deploy the Node Exporter (where possible) on your VM/Worker nodes, and the PostgreSQL exporter for each instance of AquaDB.

Where you are using a AWS RDS Postgres or Microsoft Azure PostgreSQL server for both Aqua DB's, you only need to deploy one PosgreSQL exporter.  


### Docker

You can deploy the exporter using Docker via the command below.

- Modify the password string to the literal password used to connect to the Postgres DB used for Aquasec when Aqua was deployed. 
- Where the DB is not hosted locally, modify that host to the FQDN/reachable IP of the DB instance.

```$ docker run \```
```--net=host \```
```-e DATA_SOURCE_NAME="postgresql://postgres:<YourPG_DB_password>@<AquaDbFQDN>:5432/postgres?sslmode=disable" \ ```
```quay.io/prometheuscommunity/postgres-exporter```

SSL mode can be modified if you're db uses Mutual TLS.  The exporter provides parameters that support this 
### Kubernetes

Find out the IP's of your Aqua DB's which are exposing port 5432

If your Aqua namespace is not called aqua, change as required.

```$ kubectl get svc -n aqua```

` # insert output from kubectl get svc -n aqua 


``` $ kubectl create -f postgresql-exporter/ ```


### Exposing the Aqua Prometheus Endpoint

Aqua Prom Endpoint : Console > integrations > Monitoring > Prometheus

You can query the data exposed via this endpoint using a standard http query, using your favorite browser, API client or curl/wget.

``` $ curl --location --request GET 'http://<AquaWeb-FQDN>:8080/metrics' --header 'Authorization: Bearer <AquaPrometheusToken>'```

This should expose the data from the Aqua console

<output>
### Using Postman

<img src="./images/Aqua-prometheus-endpoint-token.png">

### Checking Postgres SQL exporter data 

To ensure that Prometheus will be able to collect the data from our exporters, you can use curl/wget/a browser to view the data being exposed from the exporter via http. 

Example:
```
$ sh-3.2$ curl http://192.168.1.12:9187
<html>
	<head><title>Postgres exporter</title></head>
	<body>
	<h1>Postgres exporter</h1>
	<p><a href='/metrics'>Metrics</a></p>
	</body>
</html>
``` 

to pull the stats from the node exporter.
<br>
## Deploying Prometheus

### Data Persistence
### Prometheus Configuration

Prometheus stores its config file in /etc/prometheus as prometheus.yml. We're going to be storing this file as a configMap to represent which will be mounted inside the Prometheus container at run time. 

Our data exporters are defined in the config file as "Jobs" with target endpoints that get scraped.

```
      - job_name: 'Aqua'
        bearer_token: '7acc84a52a4ee2754c450a8d9723ba71255d4456'

        static_configs:
          - targets: ['192.168.1.222:8080']

      - job_name: 'postgres-aqua-db'
        static_configs:
          - targets: ['192.168.1.12:9187']

      - job_name: 'postgres-aqua-audit-db'
        static_configs:
          - targets: ['192.168.1.12:9188']
```
## Deploying Grafana

### Using the Aqua supplied Dashboard

Within the grafana directory, there is an Aquasec dashboard configuration file which we will use to deploy the Aqua Dashboard.

The default dashboard is a yaml file, it's been modified since as I preferred the aesthetics of some of the tiled views for the data that's displayed from Prometheus.

### Further sections are work in progress