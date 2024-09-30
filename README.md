# FleetMD deployment with Helm

This guide will walk you through the installation of FleetMD using Helm, including the setup of Redis and MySQL. You will prepare your Kubernetes environment, install Redis and MySQL using values.yaml, and finally deploy FleetMD.

## Components
### 1. **Fleet**
Fleet is the main service that will be deployed as part of the FleetMD setup. It handles the core functionalities of FleetMD and interacts with both MySQL and Redis to store and retrieve data.

### 2. MySQL
MySQL is used to store persistent data for FleetMD, including metadata, application data, and other critical records. A PersistentVolume and PersistentVolumeClaim are required for storing MySQL data.

### 3. Redis
Redis is used as a caching layer for FleetMD to improve performance. It stores temporary data like session information and other cached content.

## Prerequisites

Before installing FleetMD, ensure that your Kubernetes cluster is up and running and that Helm is installed.

## Preparation
### 1. Create a Namespace for FleetMD
First, create a dedicated Kubernetes namespace for FleetMD:

```bash
kubectl create namespace fleet
```

### 2. PersistentVolume (PV) and PersistentVolumeClaim (PVC)

Change HostPath path respectively to your environment.

Apply configuration for Redis and MySQL:

```bash
kubectl apply -f mysql/volume.yaml
kubectl apply -f redis/volume.yaml
```
### 3. Create Secrets for MySQL and Redis
FleetMD requires Redis and MySQL secrets for secure access. You will need to create secrets for both services.

##### Redis Secrets
To create the Redis secret with the password, use the following command:

```bash
kubectl create secret generic redis-secrets \
  --from-literal=redis-password=changeme
```
##### MySQL Secrets
Create the MySQL secret for the root, user, and replication passwords:

```bash
kubectl create secret generic mysql-secrets \
  --from-literal=mysql-root-password=changeme \
  --from-literal=mysql-password=changeme \
  --from-literal=mysql-replication-password=changeme
```
You can modify the *changeme* password to a more secure value of your choice.

## Installing Redis and MySQL using Helm
Instead of passing parameters via the command line, you can use values.yaml files to configure the installations of Redis and MySQL.

### 1. Install Redis

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update

helm install redis bitnami/redis --namespace fleet -f redis-values.yaml
```

### 2. Install MySQL

```bash
helm install mysql bitnami/mysql --namespace fleet -f mysql-values.yaml
```
This will deploy MySQL with the specified root, user, and replication passwords, and will use the PersistentVolumeClaim mysql-pvc created earlier for storage.

## Installing FleetMD using Helm
Once the namespace, PV/PVC, secrets, and Redis/MySQL services are up and running, you can proceed with installing FleetMD.

##### Add the Helm Repository
First, add the Helm chart repository for FleetMD:

```bash
helm repo add fleetmd https://charts.fleetmd.io
helm repo update
```
##### Install FleetMD
Now, install the FleetMD Helm chart in the fleet namespace:

```bash
helm install fleetmd fleetmd/fleetmd --namespace fleet
```

##### Verify Installation
Check the status of your deployment to ensure everything is running as expected:

```bash
kubectl get pods -n fleet
```
## Post-Installation
Once the installation is complete, you can access FleetMD using the appropriate service (e.g., via a load balancer, ingress, or NodePort). Refer to the FleetMD documentation for additional configuration steps.

## Conclusion
This guide has provided the steps to install FleetMD, along with Redis and MySQL, on Kubernetes using Helm and values.yaml for configuration. Be sure to customize your passwords and storage paths as needed for your environment.