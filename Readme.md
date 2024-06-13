# RabbitMQ Cluster Architecture in EKS using EFS 
![rabbitmq-cluster](https://github.com/asttle/eks-rabbitmq-cluster-using-efs/assets/64640283/679fc471-7afc-4d90-bc2a-47a83a1d98de)

This repository contains the necessary files and instructions to set up a RabbitMQ cluster architecture in Amazon EKS (Elastic Kubernetes Service) using Amazon EFS (Elastic File System) for storage.

Table of Contents
Overview
Node Identification
Authentication
Kubernetes Setup
Configuration and Deployment
RabbitMQ Management Portal
Sample Application
Mirroring Queues
Overview
This setup creates a RabbitMQ cluster with nodes backed by EFS. The RabbitMQ cluster uses Kubernetes for orchestration, ensuring that all nodes have consistent configuration and authentication settings.

Node Identification
Node Prefix: rabbit@rabbitmq-<node_number>
DNS Resolution:
For Docker: Use Docker network and container hostname.
For Kubernetes: Use StatefulSets (preferred over Deployments due to fixed pod names).
Ports:
4369: Discovery port used by the daemon.
5671 - 5672: Used by AMQP with and without TLS for RabbitMQ communication.
Authentication
Erlang Cookie: Used for authentication between RabbitMQ nodes and rabbitmqctl command.
Location: /var/lib/rabbitmq/.erlang.cookie
Kubernetes Setup
Namespace: Create a namespace rabbits for RabbitMQ resources.
Storage Class: Ensure the use of EFS as the storage class.
Service Account: Create a service account for RabbitMQ to interact with the Kubernetes API for node discovery.
Authentication
All RabbitMQ nodes must share the same Erlang cookie.
Create Erlang Cookie:
bash
Copy code
ERLANG_COOKIE=$(openssl rand -hex 16)
Base64 encode it and create a Kubernetes secret.
Discovery Mechanism
Use the Kubernetes plugin for node discovery.
Pass configurations as a ConfigMap.
Enabled Plugins:
rabbitmq_federation: Sync messages across instances.
rabbitmq_management: Provides UI and dashboard.
rabbitmq_peer_discovery_k8s: Required for discovery.
Configuration and Deployment
Define Backend: Use RabbitMQ discovery plugin.
API Server: Specify the API server details.
Connection: Nodes connect using hostnames.
Headless Service: Use a headless service to resolve DNS directly to pod IPs, useful for stateful applications.
Deploy RabbitMQ Instances
Use StatefulSet for unique pod names and persistent storage.
Init Containers: Utilize a BusyBox init container to move configurations and plugins to appropriate folders.
EFS Provisioning
Annotate the service account with the IAM role ARN for dynamic provisioning.
Ensure Erlang cookies are replicated across nodes.
RabbitMQ Management Portal
To access the RabbitMQ management portal, use the following command:

bash
Copy code
kubectl port-forward rabbitmq-0 -n rabbits 8080:15672
Sample Application
To verify the RabbitMQ cluster setup, deploy the sample publisher and consumer applications.

Deploy Publisher:
bash
Copy code
kubectl port-forward pod/rabbitmq-publisher-<pod_id> 5672:80 -n default
Test with Postman:
url
Copy code
http://localhost:5672/publish/hello
Mirroring Queues
For high availability, mirror queues across all RabbitMQ nodes to ensure message persistence.

Set Policy for Mirroring:
bash
Copy code
rabbitmqctl set_policy ha-fed \
  ".*" '{"federation-upstream-set":"all", "ha-sync-mode":"automatic", "ha-mode":"nodes", "ha-params":["rabbit@rabbitmq-0.rabbitmq.rabbits.svc.cluster.local","rabbit@rabbitmq-1.rabbitmq.rabbits.svc.cluster.local","rabbit@rabbitmq-2.rabbitmq.rabbits.svc.cluster.local"]}' \
  --priority 1 \
  --apply-to queues
Conclusion
This setup provides a resilient RabbitMQ cluster with EFS, ensuring data consistency and high availability across Kubernetes-managed nodes. For any issues or contributions, feel free to open an issue or pull request.


