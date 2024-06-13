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

