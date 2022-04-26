# Aqua Observability

A guide to monitoring an Aquasec deployment using observability tools such as Prometheus and Grafana 

## Introduction

Use Case.

You are a maintainer (DevSecOps/DevOps/SRE) of an Aquasec deployment on premises and you would like to better understand the resource usage of your deployment from the following components.

- Underlying Host/VM utilization  - separate to any Cloud specific monitoring tools such as AWS Cloudwatch, GCP Stack Driver or Azure Monitor etc.

- Aqua Console & container resource usage
- Aqua Gateway and Enforcers deployed.
- Aqua Databases hosted on PostgresDB (containerized or Hosted) 
- Image Scan statistics
- Policies which are deployed
- User session details
## How does it work ?

The Aquasec console can expose a Prometheus metrics endpoint which provides a number of statistics which Prometheus can scrape. This in turn can be visualized by using a Grafana deployment and importing a number of graphs.

Here's a schematic of the components and how they integrate with each other.

<img src="./aqua-obs-2022-02-24-1540.png" alt_text="Aqua Prometheus & Grafana Observability">
<br>

If you are not familiar with Prometheus and Grafana, it's basically a collection of monitoring tools that pulls real-time performance data from end-points (aka exporters) or hosts/clusters/applications, and stores this data over time, into a database. This data is made available for graphing tools to use - such as Grafana so that it can be visualized and the performance of these applications can be understood.

Think of this visually as a collection a dashboards which contain dials, graphs, stat & bar charts with data mapped to it for everything. 

## What does this all look like in real terms? 

Here are some screen shots of the kinds of data which are being collected & visualized.


<br>

Linux OS Node Exporter
<img src="./node-exporter-dashboard.png" width="20%" height="20%">
<br>
Aqua Console
<img src="./aqua-perf-dashboard.png" width="20%" height="20%">
<br>
PostgreSQL DB
<img src="./postgresql-perf-dashboard.png" width="20%" height="20%">
<br>
## What do I need to set this up?

1. Access to a Kubernetes cluster (or Docker) and a namespace for  'monitoring' or 'observability' ( you can decide what to call this )
2. A deployment of Prometheus and Grafana into the namespace above - I used a basic configuration from Bibin Wilson's tutorials [here](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/) & [Grafana](https://github.com/bibinwilson/kubernetes-grafana). This is the basis of my configuration - if you are familiar with these components you can modify them at as you need.

To make the data persistent from Prometheus and Grafana you will need to create Persistent Volume Claims within your cluster.
In the deployment section, i have included the Kubernetes yaml files for everything.

3. Prometheus node exporter which can be found [here](https://github.com/prometheus/node_exporter). 
You should deploy this on the VM/hosts in your K8s cluster (if possible.) I deployed it onto my physical server which runs [microk8s](https://microk8s.io/). 

4. The PostgreSQL DB exporter & access to the Aqua PostgreSQL - obtained from [Prometheus's GitHub](https://github.com/prometheus-community/postgres_exporter). Again, deployed into the namespace to connect to the K8s servicename or K8s ClusterIP/LoadBalancer IP's which expose the AquaDB (Scalock) and Aqua Audit DB(SLK_Audit)
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

```docker run \```
```--net=host \```
```-e DATA_SOURCE_NAME="postgresql://postgres:password@localhost:5432/postgres?sslmode=disable" \ ```
```quay.io/prometheuscommunity/postgres-exporter```


### Kubernetes

Find out our Aqua DB's which are exposing port 5432

If your Aqua namespace is not called aqua, change as required.

```$ kubectl get svc -n aqua```

` # insert output from kubectl get svc -n aqua 




``` $ kubectl create -f postgresql-exporter/ ```
Modify the 

### Exposing the Aqua Prometheus Endpoint


You can query the data exposed via this endpoint using a standard http query, using yoru favourite browser, API client or wget.

``` $ curl --location --request GET 'http://192.168.1.222:8080/metrics' --header 'Authorization: Bearer <AquaPrometheusToken>'```

### Using Postman

<img src="Aqua-prometheus-endpoint-token.png">