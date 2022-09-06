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

We can collect the performance data from each of the components operating in the Aqua deployment, based on the following:
- Host / Worker Node CPU, RAM, Disk and Network I/O
- Database resource usage - RAM, CPU, queries and IOPs
- Kubernetes pod usage
- Aqua Console, Gateway and Enforcer statistics
- Being able to identify issues and service/host outages at a glance.

This data can be used and graphed to provide insight of current and historical data metrics though the use of Prometheus, the available exporters and Grafana to provide the dashboard and visual metrics for DevOps and SRE teams managing several Aquasec deployments.

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
You should deploy this on the VM/hosts in your K8s cluster (if possible). 

4. The PostgreSQL DB exporter & access to the Aqua PostgreSQL DB - obtained from [Prometheus's GitHub](https://github.com/prometheus-community/postgres_exporter). Again, deployed into the namespace to connect to the K8s Service name or K8s ClusterIP/LoadBalancer IP's which expose the AquaDB (Scalock) and Aqua Audit DB(SLK_Audit)
5. The Aquasec Prometheus endpoint token and FQDN for the Aquasec Console. i.e. https://aquasec-console-dev.mydomain.com
6. Patience and coffee :) 

# Deployment
## What kubernetes platforms have I tested this against
This stack has been developed and deployed it onto a physical server which runs [microk8s](https://microk8s.io/). 

| Kubernetes Platform |                 | AKS | EKS | GKE | Native K8s | MicroK8s | Minikube |   |   |   |
|:-------------------:|:---------------:|:---:|:---:|:---:|:----------:|:--------:|:--------:|:---:|:---:|:---:|
| Checked against     |  | -   | -   | -   | -          | Yes      | -        |   |   |   |
|                     |                 |     |     |     |            |          |          |   |   |   |

## Prometheus Exporters

Firstly we will setup the exporters and check that the data is available for Prometheus to 'scrape' from.

Deploy a PostgreSQL exporter Prometheus instance for each instance of Aqua databases (Scalock and SLK Audit)

Where you are using a AWS RDS Postgres or Microsoft Azure PostgreSQL server for both Aqua DB's, you only need to deploy one Posgres SQL exporter.  

If you're using the Aqua Advanced Deployment Architecture and are using the containerised deployment of Databases, you can deploy two of the Postgres exporters as illustrated in the diagram.
### Docker

You can deploy the exporter using Docker via the command below.

- Modify the password string to the literal password used to connect to the Postgres DB used for Aquasec when Aqua was deployed. 
- Where the DB is not hosted locally, modify that host to the FQDN/reachable IP of the DB instance.

```$ docker run \```
```--net=host \```
```-e DATA_SOURCE_NAME="postgresql://postgres:<YourPG_DB_password>@<AquaDbFQDN>:5432/postgres?sslmode=disable" \ ```
```quay.io/prometheuscommunity/postgres-exporter```

SSL mode can be modified if you're db uses Mutual TLS.  The exporter provides [parameters](https://github.com/prometheus-community/postgres_exporter#flags) that support this.  
## Kubernetes
###  Deployment
Find out the Service name of your Aqua DB's which are exposing port 5432 from Postgres so that the connection from the PG exporters will connect.

_If your Aqua namespace is not called aqua, you can change as required._

`$ kubectl get svc -n aqua`

Insert output from `kubectl get svc -n aqua`

The postgres-exporter deployment files `aqua_db_prometheus_exporter.yaml` & `audit_db-prometheus-exporter.yaml` for Aqua-db(scalock) and Audit-db (slk_audit) contains the connection string which will initiate polling for the DB instances as shown in line 48 in both aqua-db / audit-db yamls :

`          value: postgresql://$(AQUA_DB_USERNAME):$(AQUA_DB_PASSWORD)@192.168.1.226:5432/postgres?sslmode=disable`
<br>

Substitute `192.168.1.222` with the FQDN for each Aqua DB respectively.

Deploying the exporters :

Run the command ` $ kubectl create -f postgresql-exporter/ `


**Note** : Once the exporter is deployed you can test the connection to it by requesting the  `/metrics` uri from each exporter using the command:

`$ curl --location --request GET 'http://192.168.1.231:9187/metrics' `

This can be seen in the logs of each pod

**Start up**

``ts=2022-09-05T10:07:27.709Z caller=main.go:123 level=info msg="Listening on address" address=:9187``<br>
`ts=2022-09-05T10:07:27.709Z caller=tls_config.go:195 level=info msg="TLS is disabled." http2=false`

**When the wget hits the exporter**

`ts=2022-09-05T10:09:00.122Z caller=server.go:74 level=info msg="Established new database connection" fingerprint=192.168.1.226:5432`<br>
`ts=2022-09-05T10:09:00.150Z caller=postgres_exporter.go:662 level=info msg="Semantic version changed" server=192.168.1.226:5432 from=0.0.0 to=11.16.0`

As you can see the last two lines shows the connection to the DB as a consequence of the wget. This mimics what prometheus will do when it polls each exporter to collect stats and confirms connectivity.


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