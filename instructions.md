# Usage

This guide describes the necessary steps to deploy Fortishield on Kubernetes.

## Pre-requisites

- Kubernetes cluster already deployed.
- Kubernetes can run on a wide range of Cloud providers and bare-metal environments, this repository focuses on [AWS](https://aws.amazon.com/). It was tested using [Amazon EKS](https://docs.aws.amazon.com/eks). You should be able to:
    - Create Persistent Volumes on top of AWS EBS when using a volumeClaimTemplates
    - Create a record set in AWS Route 53 from a Kubernetes LoadBalancer.
- Having at least two Kubernetes nodes in order to meet the *podAntiAffinity* policy.
- For Kubernetes version 1.23 or higher, the assignment of an IAM Role is necessary for the CSI driver to function correctly. Within the AWS documentation you can find the instructions for the assignment: https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html
- The installation of the CSI driver is necessary for new and old deployments, since it is a Kubernetes feature.


## Overview

### StateFulSet and Deployments Controllers

Like a Deployment, a StatefulSet manages Pods that are based on an identical container specification, but it maintains an identity attached to each of its pods. These pods are created from the same specification, but they are not interchangeable: each one has a persistent identifier maintained across any rescheduling.

It is useful for stateful applications like databases that save the data to a persistent storage. The states of each Fortishield manager as well as Fortishield indexer are desirable to maintain, so we declare them using StatefulSet to ensure that they maintain their states in every startup.

Deployments are intended for stateless use and are quite lightweight and seem to be appropriate for Fortishield dashboard and Nginx, where it is not necessary to maintain the states.

### Pods

#### Fortishield master

This pod contains the master node of the Fortishield cluster. The master node centralizes and coordinates worker nodes, making sure the critical and required data is consistent across all nodes.
The management is performed only in this node, so the agent registration service (authd) and the API are placed here.

Details:
- Image: Docker Hub 'fortishield/fortishield-manager'
- Controller: StatefulSet

#### Fortishield worker 0 / 1

These pods contain a worker node of the Fortishield cluster. They will receive the agent events.

Details:
- Image: Docker Hub 'fortishield/fortishield-manager'
- Controller: StatefulSet


#### Fortishield indexer

Fortishield indexer pod. Used to build an Fortishield indexer cluster.

Details:
- Image: fortishield/fortishield-indexer
- Controller: StatefulSet

#### Fortishield dashboard

Fortishield dashboard pod. It lets you visualize your Fortishield indexer data, along with other features as the Fortishield app.

Details:
- image: Docker Hub 'fortishield/fortishield-dashboard'
- Controller: Deployment

### Services

#### Indexer stack

- fortishield-indexer:
  - Communication for Fortishield indexer nodes.
- indexer:
  - Fortishield indexer API. Used by Fortishield dashboard to write/read alerts.
- dashboard:
  - Fortishield dashboard service. https://fortishield.your-domain.com:443

#### Fortishield

- fortishield:
  - Fortishield API: fortishield-master.your-domain.com:55000
  - Agent registration service (authd): fortishield-master.your-domain.com:1515
- fortishield-workers:
  - Reporting service: fortishield-manager.your-domain.com:1514
- fortishield-cluster:
  - Communication for Fortishield manager nodes.


## Deploy


### Step 1: Deploy Kubernetes

Deploying the Kubernetes cluster is out of the scope of this guide.

This repository focuses on [AWS](https://aws.amazon.com/) but it should be easy to adapt it to another Cloud provider. In case you are using AWS, we recommend [EKS](https://docs.aws.amazon.com/en_us/eks/latest/userguide/getting-started.html).


### Step 2: Create domains to access the services

We recommend creating domains and certificates to access the services. Examples:

- fortishield-master.your-domain.com: Fortishield API and authd registration service.
- fortishield-manager.your-domain.com: Reporting service.
- fortishield.your-domain.com: Fortishield dashboard app.

Note: You can skip this step and the services will be accessible using the Load balancer DNS from the VPC.

### Step 3: Deployment

Clone this repository to deploy the necessary services and pods.

```BASH
$ git clone https://github.com/fortishield/fortishield-kubernetes.git
$ cd fortishield-kubernetes
```

### Step 3.1: Setup SSL certificates

You can generate self-signed certificates for the Fortishield indexer cluster using the script at `fortishield/certs/indexer_cluster/generate_certs.sh` or provide your own.

Since Fortishield dashboard has HTTPS enabled it will require its own certificates, these may be generated with: `openssl req -x509 -batch -nodes -days 365 -newkey rsa:2048 -keyout key.pem -out cert.pem`, there is an utility script at `fortishield/certs/dashboard_http/generate_certs.sh` to help with this.

The required certificates are imported via secretGenerator on the `kustomization.yml` file:

    secretGenerator:
    - name: indexer-certs
        files:
        - certs/indexer_cluster/root-ca.pem
        - certs/indexer_cluster/node.pem
        - certs/indexer_cluster/node-key.pem
        - certs/indexer_cluster/dashboard.pem
        - certs/indexer_cluster/dashboard-key.pem
        - certs/indexer_cluster/admin.pem
        - certs/indexer_cluster/admin-key.pem
        - certs/indexer_cluster/filebeat.pem
        - certs/indexer_cluster/filebeat-key.pem
    - name: dashboard-certs
        files:
        - certs/dashboard_http/cert.pem
        - certs/dashboard_http/key.pem

### Step 3.2: Apply all manifests using kustomize

We are using the overlay feature of kustomize to create two variants: `eks` and `local-env`, in this guide we're using `eks`. (For a deployment on a local environment check the guide on [local-environment.md](local-environment.md))

You can adjust resources for the cluster on `envs/eks/`, you can tune cpu, memory as well as storage for persistent volumes of each of the cluster objects.


By using the kustomization file on the `eks` variant we can now deploy the whole cluster with a single command:

```BASH
$ kubectl apply -k envs/eks/
```

### Verifying the deployment

#### Namespace

```BASH
$ kubectl get namespaces | grep fortishield
fortishield         Active    12m
```

#### Services

```BASH
$ kubectl get services -n fortishield
NAME            TYPE           CLUSTER-IP       EXTERNAL-IP             PORT(S)                          AGE
dashboard       LoadBalancer   10.100.55.244    <entrypoint_assigned>   443:31670/TCP                    4h13m
indexer         LoadBalancer   10.100.199.148   <entrypoint_assigned>   9200:32270/TCP                   4h13m
fortishield           LoadBalancer   10.100.176.82    <entrypoint_assigned>   1515:32602/TCP,55000:32116/TCP   4h13m
fortishield-cluster   ClusterIP      None             <none>                  1516/TCP                         4h13m
fortishield-indexer   ClusterIP      None             <none>                  9300/TCP                         4h13m
fortishield-workers   LoadBalancer   10.100.165.20    <entrypoint_assigned>   1514:30128/TCP                   4h13m
```

#### Deployments

```BASH
$ kubectl get deployments -n fortishield
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
fortishield-dashboard   1/1     1            1           4h16m
```

#### Statefulsets

```BASH
$ kubectl get statefulsets -n fortishield
NAME                   READY   AGE
fortishield-indexer          3/3     4h17m
fortishield-manager-master   1/1     4h17m
fortishield-manager-worker   2/2     4h17m
```

#### Pods

```BASH
$ kubectl get pods -n fortishield
NAME                               READY   STATUS    RESTARTS   AGE
fortishield-dashboard-57d455f894-ffwsk   1/1     Running   0          4h17m
fortishield-indexer-0                    1/1     Running   0          4h17m
fortishield-indexer-1                    1/1     Running   0          4h17m
fortishield-indexer-2                    1/1     Running   0          4h17m
fortishield-manager-master-0             1/1     Running   0          4h17m
fortishield-manager-worker-0             1/1     Running   0          4h17m
fortishield-manager-worker-1             1/1     Running   0          4h17m
```

#### Accessing Fortishield dashboard

In case you created domain names for the services, you should be able to access Fortishield dashboard using the proposed domain name: https://fortishield.your-domain.com.

Also, you can access using the External-IP (from the VPC): https://internal-xxx-yyy.us-east-1.elb.amazonaws.com:443

```BASH
$ kubectl get services -o wide -n fortishield
NAME        TYPE           CLUSTER-IP       EXTERNAL-IP                                                              PORT(S)        AGE     SELECTOR
dashboard   LoadBalancer   10.100.55.244    a91dadfdf2d33493dad0a267eb85b352-1129724810.us-west-1.elb.amazonaws.com  443:31670/TCP  4h19m   app=fortishield-dashboard
```
