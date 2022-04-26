# Aqua-observability

A guide to monitoring an Aquasec deployment using observability tools such as Prometheus and Grafana 

## Introduction

You are a user of an Aquasec deployment on premises and you would like to better understand the resource usage of your deployment from the following components.
- Aqua Console & container resource usage
- Aqua Gateway and Enforcers deployed.
- Aqua Databases hosted on PostgresDB (containerized or Hosted) 
- Image Scan statistics
- Underlying Host/VM utilization  - separate to any Cloud specific monitoring tools such as AWS Cloudwatch, GCP Stack Driver or Azure Monitor etc.

## How does it work

The Aquasec console can expose a Prometheus metrics endpoint which provides a number of statistics which Prometheus can scrape. This in turn can be visualized by using a Grafana deployment and importing a number of graphs.

Here's a schematic of the components and how they integrate with each other.

<img src="./aqua-obs-2022-02-24-1540.png" alt_text="Aqua Prometheus & Grafana Observability">
<br>

## What do I need to set this up?

1. Access to a Kubernetes cluster and a namespace for  'monitoring' or 'observability' ( you can decide what to call this )
2. A deployment of Prometheus and Grafana into the named namespace above.
3. Prometheus node exporter : https://github.com/prometheus/node_exporter
4. The PostgreSQL DB exporter & access to the Aqua PostgreSQL
5. The Aquasec Prometheus endpoint token and FQDN for the Aquasec Console. i.e. https://aquasec-console-dev.mydomain.com
6. Patience and coffee :) 

# Deployment

