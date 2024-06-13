# RabbitMQ Cluster Architecture in EKS using EFS 
![rabbitmq-cluster](https://github.com/asttle/eks-rabbitmq-cluster-using-efs/assets/64640283/679fc471-7afc-4d90-bc2a-47a83a1d98de)

This repository contains the necessary files and instructions to set up a RabbitMQ cluster architecture in Amazon EKS (Elastic Kubernetes Service) using Amazon EFS (Elastic File System) for storage.

## Table of Contents
1. [Overview](#overview)
2. [Node Identification](#node-identification)
3. [Authentication](#authentication)
4. [Kubernetes Setup](#kubernetes-setup)
5. [Configuration and Deployment](#configuration-and-deployment)
6. [RabbitMQ Management Portal](#rabbitmq-management-portal)
7. [Sample Application](#sample-application)
8. [Mirroring Queues](#mirroring-queues)
9. [Conclusion](#conclusion)

## Overview
This setup creates a RabbitMQ cluster with nodes backed by EFS. The RabbitMQ cluster uses Kubernetes for orchestration, ensuring that all nodes have consistent configuration and authentication settings.

## Node Identification
- **Node Prefix:** `rabbit@rabbitmq-<node_number>`
- **DNS Resolution:** 
  - For Docker: Use Docker network and container hostname.
  - For Kubernetes: Use StatefulSets (preferred over Deployments due to fixed pod names).
- **Ports:**
  - **4369:** Discovery port used by the daemon.
  - **5671 - 5672:** Used by AMQP with and without TLS for RabbitMQ communication.

## Authentication
- **Erlang Cookie:** Used for authentication between RabbitMQ nodes and `rabbitmqctl` command.
  - Location: `/var/lib/rabbitmq/.erlang.cookie`

## Kubernetes Setup
1. **Namespace:** Create a namespace `rabbits` for RabbitMQ resources.
2. **Storage Class:** Ensure the use of EFS as the storage class.
3. **Service Account:** Create a service account for RabbitMQ to interact with the Kubernetes API for node discovery.

### Authentication
- All RabbitMQ nodes must share the same Erlang cookie.
- **Create Erlang Cookie:**
  ```bash
  ERLANG_COOKIE=$(openssl rand -hex 16)
  ```
- Base64 encode it and create a Kubernetes secret.

## Discovery Mechanism
- Use the Kubernetes plugin for node discovery.
- Pass configurations as a ConfigMap.

### Enabled Plugins:
- `rabbitmq_federation`: Sync messages across instances.
- `rabbitmq_management`: Provides UI and dashboard.
- `rabbitmq_peer_discovery_k8s`: Required for discovery.

## Configuration and Deployment
1. **Define Backend**: Use RabbitMQ discovery plugin.
2. **API Server**: Specify the API server details.
3. **Connection**: Nodes connect using hostnames.
4. **Headless Service**: Use a headless service to resolve DNS directly to pod IPs, useful for stateful applications.

## Deploy RabbitMQ Instances
- Use StatefulSet for unique pod names and persistent storage.
- Init Containers: Utilize a BusyBox init container to move configurations and plugins to appropriate folders.

## EFS Provisioning
- Annotate the service account with the IAM role ARN for dynamic provisioning.
- Ensure Erlang cookies are replicated across nodes.

## RabbitMQ Management Portal
To access the RabbitMQ management portal, use the following command:
```bash
kubectl port-forward rabbitmq-0 -n rabbits 8080:15672
```



