# ELK-Stack
ELK Stack demonstration
# Background & Purpose #

The purpose of ELK Stack demo is created to showcase SIEM servers, monitoring and alerting.
# Scope #

This project was deployed in an Azure cloud environment. This ELK stack monitored Filebeats & Metricbeats for a DVWA application.
# Implementation #

## VNet Creation ##

1. Create the VNet with network IP range 10.0.0.0/16
2. Create default subnet with IP range 10.0.0.0/24

## Establish Network Security Group ##

Immediately created a network security group that by default blocked ALL traffic to protect the environment as it was built. The NSG is subsequently modified to allow specific traffic as the infrastructure is built.

![Cloud Diagram](images/CloudNetwork.png)

