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

For the benefit of illustration, here are some screen shots of the kinds of data which are being collected.

[Linux OS Node Exporter](<img src="./node-exporter-dashboard.png">)
Aqua Console
PostgreSQL DB

## What do I need to set this up?

1. Access to a Kubernetes cluster (or Docker) and a namespace for  'monitoring' or 'observability' ( you can decide what to call this )
2. A deployment of Prometheus and Grafana into the named namespace above - I found a basic configuration that I got from Bibin Wilson's tutorials [here](https://devopscube.com/setup-prometheus-monitoring-on-kubernetes/) 
3. Prometheus node exporter which can be pulled from [here](https://github.com/prometheus/node_exporter) I run this on the physical server which runs [microk8s](https://microk8s.io/).
4. The PostgreSQL DB exporter & access to the Aqua PostgreSQL - obtained from [Prometheus's GitHub]()
5. The Aquasec Prometheus endpoint token and FQDN for the Aquasec Console. i.e. https://aquasec-console-dev.mydomain.com
6. Patience and coffee :) 

# Deployment

