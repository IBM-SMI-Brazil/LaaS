# LaaS - Log Analytics as a Service

 When consolidation through Syslog Server is not enough!

![SysLog_Diagram](https://github.com/IBM-SMI-Brazil/LaaS/blob/master/images/syslog.png)

Syslog, is a standardized way (or Protocol) of producing and sending Log and Event information from Unix/Linux and Windows systems (which produces Event Logs) and Devices (Routers, Firewalls, Switches, Servers, etc) over UDP Port 514 to a centralized Log/Event Message collector which is known as a Syslog Server.

One of the main reasons Syslog was so widely accepted throughout the industry was because of its simplicity – There is little to no uniformity or standardization when it comes to the content that a Device, Server or Operating system is written and sends log information.

However the diversity of information that can be consolidate through a SysLog server is also one of its biggest problems, since as the amount ingested data increases, the complexity also increases, and support teams start to have considerable problems to extract relevant information from the central stash, also, there's no way of finding information except "grepping" through the data manually or using scripts.

## Why provide Log Analytics as a Service?

Our goal is to help supporting teams on finding patterns and trends from their logs, connecting the dots from multiple sources through a consolidated view. **If your application/system writes a log, the solution will read it!**.

![LaaS Representation](https://github.com/IBM-SMI-Brazil/LaaS/blob/master/images/laas.png)

This repository is inteended to assist with the implementation of a set of tools to assist with Log Consolidation to assist troubleshooting, auditing and investigation by using open-source tools according to the requirements of each different environment.

The solution we're proposing is based on Open-Source softwares, designed for the Cloud Computing era to be offered as a service. The idea is to provide easy deployment through dockerimages with minimum customization To forward syslogs to a central stash, where we'll be able to keep a history to assist administrators on troubleshooting, analytics and creating relevant KPI’s.

We'll be integrating the following tools:

- [ELK stack:](https://www.elastic.co/webinars/introduction-elk-stack) The ELK stack consists of Elasticsearch, Logstash, and Kibana. Although they've all been built to work exceptionally well together, each one is a separate project that is driven by the open-source vendor Elastic —- which itself began as an enterprise search platform vendor.

  - [Elasticsearch:](https://www.elastic.co/products/elasticsearch) is a search engine based on Lucene. It provides a distributed, multitenant-capable full-text search engine with an HTTP web interface and schema-free JSON documents. Elasticsearch is developed in Java and is released as open source under the terms of the Apache License.
  
  - [Logstash:](https://www.elastic.co/products/logstash) Is an open source tool for collecting, parsing, and storing logs for future use. Kibana 3 is a web interface that can be used to search and view the logs that Logstash has indexed. Both of these tools are based on Elasticsearch. Elasticsearch, Logstash, and Kibana, when used together is known as an ELK stack.
  
  - [Kibana:](https://www.elastic.co/products/kibana) Is an open source data visualization plugin for Elasticsearch. It provides visualization capabilities on top of the content indexed on an Elasticsearch cluster. Users can create bar, line and scatter plots, or pie charts and maps on top of large volumes of data.

- [GrayLog:](https://www.graylog.org/) is a powerful log management and analysis tool that has many use cases, from monitoring SSH logins and unusual activity to debugging applications. It is based on Elasticsearch, Java, MongoDB, and Scala.

- [Grafana:](https://github.com/grafana/grafana) is an open source metric analytics & visualization suite. It is most commonly used for visualizing time series data for infrastructure and application analytics but many use it in other domains including industrial sensors, home automation, weather, and process control.

Additionally we're also proposing that depending on the architecture where the solution is being implemented we should complement the log data whenever possible with system information from the servers so that we can assing the troubleshooting by drilling-down to the exact server's state when a given events is perceived through the log analysis.

- [Beats](https://www.elastic.co/products/beats): Is a lightweight agent from Elastic, which is great for gathering data. They sit on your servers and centralize data in Elasticsearch. Idea is that we use it to increase the the processing muscle, Beats can also ship to Logstash for transformation and parsing.

- [CAdvisor:](https://github.com/google/cadvisor) (Container Advisor) provides container users an understanding of the resource usage and performance characteristics of their running containers. It is a running daemon that collects, aggregates, processes, and exports information about running containers. Specifically, for each container it keeps resource isolation parameters, historical resource usage, histograms of complete historical resource usage and network statistics. This data is exported by container and machine-wide.

- [Prometheus](https://github.com/prometheus): Prometheus, a Cloud Native Computing Foundation project, is a systems and service monitoring system. It collects metrics from configured targets at given intervals, evaluates rule expressions, displays the results, and can trigger alerts if some condition is observed to be true.

- [Node_Exporter:](https://github.com/prometheus/node_exporter) Prometheus exporter for hardware and OS metrics exposed by \*NIX kernels, written in Go with pluggable metric collectors.


We're proposing the following advantages with the bundle of solutions that we've defined :

- A solution based on Open-Source softwares, designed for the Cloud Computing era to be offered as a service
- Easy deployment through docker images
- Minimum customizations required to forward the logs to a central stash
- Historic view to assist troubleshoot, analytics and creation of relevant KPI’s
- A solution portable to Cloud, On-Premisses or Hybrid environments
- Correlation of metrics and logs so that one can find trends about their devices and their applications
- Can perform cross-server queries, giving a big picture of their overall performance
- Up/down monitoring (heartbeat) can be used to visualize ping errors by region


## Architecture

The following is a visual representation of how the different tools will be integrated:
![LaaS Proposed Architecture](https://github.com/IBM-SMI-Brazil/LaaS/tree/master/images/laas_architecture.png)

## Deployment Steps

For the full deployment steps please follow instructions provided at the following page:

- [LaaS Deployment Steps](deployment/DEPLOYMENT_STEPS.md)

## Presentation Deck

We're currently using the following presentation as a quick reference to present the solution:

![2017-11-01_LaaS_(LogAnalytics_As_A_Service)_en-us_v6.0.pdf](https://github.com/IBM-SMI-Brazil/LaaS/tree/master/ppt-deck/2017-11-01_LaaS_(LogAnalytics_As_A_Service)_en-us_v6.0.pdf)

![2017-11-04_LaaS_(LogAnalytics_As_A_Service)_pt-br_v6.0.pdf](https://github.com/IBM-SMI-Brazil/LaaS/tree/master/ppt-deck/2017-11-04_LaaS_(LogAnalytics_As_A_Service)_pt-br_v6.0.pdf)

## Contact

In case you need any further information or assistance, the following folks can be contacted for assistance.

Team            |        Name   |    Email    | Role  
---------------------|------------------------------------|---------|------|
<img src="https://github.com/IBM-SMI-Brazil/LaaS/tree/master/images/hprado.png" width="120"> | [Hugo Prado](https://www.linkedin.com/in/hugodoprado/) | hprado@br.ibm.com | IT Specialist
<img src="https://https://github.com/IBM-SMI-Brazil/LaaS/tree/master/images/fsilveir.jpg" width="120">  | [Felipe Silveira](https://www.linkedin.com/in/fsilveira/) | fsilveir@br.ibm.com | IT Specialist
