# Minecraft Bedrock Server Deployment on Google Cloud with Splunk OpenTelemetry

![GCP](https://img.shields.io/badge/GCP-Deployment-green)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Setup-blue)
![Helm](https://img.shields.io/badge/Helm-Integration-orange)
![License](https://img.shields.io/github/license/yourusername/your-repo-name)

## Table of Contents

- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Project Overview](#project-overview)
- [Setup Guide](#setup-guide)
  - [1. Create a New GCP Project](#1-create-a-new-gcp-project)
  - [2. Enable GCP Services](#2-enable-gcp-services)
  - [3. Set Up GKE Autopilot Cluster](#3-set-up-gke-autopilot-cluster)
  - [4. Deploy Minecraft Bedrock Server](#4-deploy-minecraft-bedrock-server)
  - [5. Deploy Splunk OpenTelemetry Collector](#5-deploy-splunk-opentelemetry-collector)
  - [6. Secure Connectivity to Homelab Splunk (Optional)](#6-secure-connectivity-to-homelab-splunk-optional)
- [Usage](#usage)
- [Troubleshooting](#troubleshooting)
- [References](#references)
- [License](#license)
- [Contact](#contact)

## Introduction

Welcome to the **Minecraft Bedrock Server Deployment** project! This repository is part of my journey towards obtaining the **Certified Kubernetes Administrator (CKA)** certification. The project demonstrates how to deploy a Minecraft Bedrock Server on **Google Cloud Platform (GCP)** using **Google Kubernetes Engine (GKE) Autopilot**. Additionally, it integrates **Splunk OpenTelemetry (Otel) Collector** to collect and send logs to **Splunk Enterprise** hosted in my homelab.

## Prerequisites

Before you begin, ensure you have the following:

- **Google Cloud Platform (GCP) Account:** [Sign up](https://cloud.google.com/) if you don't have one.
- **Google Cloud SDK:** Installed and configured. [Installation Guide](https://cloud.google.com/sdk/docs/install)
- **Kubectl:** Installed for interacting with Kubernetes clusters. [Installation Guide](https://kubernetes.io/docs/tasks/tools/)
- **Helm:** Installed for managing Kubernetes applications. [Installation Guide](https://helm.sh/docs/intro/install/)
- **Basic Knowledge:** Familiarity with Kubernetes and Helm is beneficial.
- **Splunk Enterprise:** Deployed in your homelab with an available HTTP Event Collector (HEC).

## Project Overview

This project involves the following key components:

1. **GKE Autopilot Cluster:** Utilizes GKE Autopilot for managing the Kubernetes infrastructure.
2. **Minecraft Bedrock Server:** Deployed as a Kubernetes application using a YAML configuration.
3. **Splunk OpenTelemetry Collector:** Deployed via Helm to collect logs from Kubernetes pods and send them to Splunk Enterprise using HEC.

## Setup Guide

Follow the steps below to set up the entire environment.

### 1. Create a New GCP Project

1. **Navigate to the GCP Console:**

   Visit the [GCP Console](https://console.cloud.google.com/).

2. **Create a New Project:**

   - Click on the project dropdown in the top navigation bar.
   - Select **"New Project"**.
   - Enter your **Project Name** and **Billing Account**.
   - Click **"Create"**.

3. **Set the Project in gcloud:**

   ```bash
   gcloud config set project YOUR_PROJECT_ID
   ```

### 2. Enable GCP Services

Enable the necessary GCP services using the Cloud SDK CLI.

1. **Initialize gcloud (if not already done):**

   ```bash
   gcloud init
   ```

2. **Enable Required Services:**

   ```bash
   gcloud services enable container.googleapis.com
   gcloud services enable compute.googleapis.com
   ```

### 3. Set Up GKE Autopilot Cluster

Create a Kubernetes cluster using GKE Autopilot.

```bash
gcloud container clusters create-auto minecraft-cluster \
  --region us-central1 \
  --project YOUR_PROJECT_ID
```

Verify the cluster is up and running:

```bash
gcloud container clusters list
```

### 4. Deploy Minecraft Bedrock Server

Deploy the Minecraft Bedrock Server using the provided YAML configuration.

1. **Clone the Repository:**

   ```bash
   git clone https://github.com/wedge22/k8s-mc-splunk-otel.git
   cd k8s-mc-splunk-otel
   ```

2. **Navigate to the Deployment Guide:**

   Follow the detailed steps in the [GCP Guide](https://github.com/wedge22/k8s-mc-splunk-otel/blob/master/gcp-guide.md).

3. **Update Security Settings (Important):**

   Before deploying, update the `loadBalancerSourceRanges` in `minecraft-bedrock-server.yaml` to restrict access to your IP addresses:

   ```yaml
   # In minecraft-bedrock-server.yaml, update the Service section:
   loadBalancerSourceRanges:
     - "YOUR_HOME_IP/32"        # Replace with your actual IP
     - "FRIEND_IP_1/32"         # Add additional IPs as needed
   ```

   To find your current public IP:
   ```bash
   curl ifconfig.me
   ```

   > **Security Note:** By default, the LoadBalancer exposes the Minecraft server to the public internet. Using `loadBalancerSourceRanges` restricts access to only the IP addresses you specify, significantly improving security.

4. **Apply the YAML Configuration:**

   ```bash
   kubectl apply -f minecraft-bedrock-server.yaml
   ```

   *This will create the necessary pods and services for the Minecraft server.*

### 5. Deploy Splunk OpenTelemetry Collector

Integrate Splunk Otel Collector to collect and forward logs.

1. **Add Helm Repository for Splunk Otel Collector:**

   ```bash
   helm repo add splunk-otel-collector-chart https://signalfx.github.io/splunk-otel-collector-chart
   helm repo update
   ```

2. **Create `values.yaml`:**

   Customize the Helm chart with your Splunk credentials.

   ```bash
   nano values.yaml
   ```

   **Example `values.yaml`:**

   ```yaml
   splunk:
     accessToken: YOUR_SPLUNK_ACCESS_TOKEN
     endpoint: https://ingest.YOUR_SPLUNK_HEC_ENDPOINT:443
   ```

   *Replace `YOUR_SPLUNK_ACCESS_TOKEN` and `YOUR_SPLUNK_HEC_ENDPOINT` with your actual Splunk HEC details.*

3. **Install the Splunk Otel Collector:**

   ```bash
   helm install my-splunk-otel-collector --values values.yaml splunk-otel-collector-chart/splunk-otel-collector
   ```

4. **Verify the Installation:**

   ```bash
   kubectl get pods
   ```

5. **Access the Pod (if needed):**

   ```bash
   kubectl exec --stdin --tty my-splunk-otel-collector-pod -- /bin/sh
   ```

   *Replace `my-splunk-otel-collector-pod` with the actual pod name.*

### 6. Secure Connectivity to Homelab Splunk (Optional)

If your Splunk Enterprise instance is running in a homelab behind a firewall, you'll need to securely expose the HEC endpoint to receive data from GKE.

**Recommended Approach: Cloudflare Tunnel + Reverse Proxy**

This project uses Cloudflare Tunnel with Traefik to securely route data from GKE to a homelab Splunk instance without exposing ports:

```
GKE OTel Collector → HTTPS → Cloudflare Tunnel → Traefik → Splunk HEC
```

**Benefits:**
- ✅ End-to-end HTTPS encryption
- ✅ No port forwarding required on home network
- ✅ Valid SSL certificates from Cloudflare
- ✅ Free tier available

Once configured, update your `values.yaml` endpoint to use your Cloudflare domain:

```yaml
splunkPlatform:
  endpoint: "https://splunk-hec.yourdomain.com/services/collector/event"
  token: "YOUR_HEC_TOKEN"
  insecureSkipVerify: false  # Cloudflare provides valid SSL certificates
```

**Alternative Options:**
- **Cloud VPN:** Create an IPsec VPN tunnel between GCP and your homelab
- **Dynamic DNS + Port Forwarding:** Expose HEC via HTTPS (ensure proper security measures)

> **Note:** Detailed setup of Cloudflare Tunnel and Traefik is beyond the scope of this guide. Refer to the [Cloudflare Tunnel documentation](https://developers.cloudflare.com/cloudflare-one/connections/connect-apps/) for setup instructions.

## Usage

Once all components are deployed:

1. **Access the Minecraft Server:**

   - Retrieve the external IP of the Minecraft service:

     ```bash
     kubectl get services
     ```

   - Use the external IP to connect via your Minecraft Bedrock client.

2. **Monitor Logs in Splunk:**

   - Navigate to your Splunk Enterprise dashboard.
   - Use the HEC endpoint to view and analyze the logs collected from the Kubernetes pods.

## Troubleshooting

- **Check Pod Status:**

  ```bash
  kubectl get pods
  ```

- **View Pod Logs:**

  ```bash
  kubectl logs pod-name
  ```

- **Describe Pod for Detailed Information:**

  ```bash
  kubectl describe pod pod-name
  ```

- **Helm Release Status:**

  ```bash
  helm status my-splunk-otel-collector
  ```

- **Common Issues:**
  - **Service Not Accessible:** Ensure the Kubernetes service has an external IP and that firewall rules allow traffic on the necessary ports.
  - **Splunk Logs Not Appearing:** Verify the `values.yaml` configurations and ensure that the Splunk HEC endpoint and access token are correct.

## References

- **GCP Guide:** [k8s-mc-splunk-otel/gcp-guide.md](https://github.com/wedge22/k8s-mc-splunk-otel/blob/master/gcp-guide.md)
- **Minecraft Bedrock Server YAML:** [minecraft-bedrock-server.yaml](https://github.com/wedge22/k8s-mc-splunk-otel/blob/master/minecraft-bedrock-server.yaml)
- **Casey West's Guide:** [Casey West - Minecraft Bedrock Server Setup](http://caseywest.com/)

## License

This project is licensed under the [MIT License](LICENSE).

