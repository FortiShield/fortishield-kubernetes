# Fortishield Kubernetes

[![Slack](https://img.shields.io/badge/slack-join-blue.svg)](https://fortishield.github.io/community/join-us-on-slack/)
[![Email](https://img.shields.io/badge/email-join-blue.svg)](https://groups.google.com/forum/#!forum/fortishield)
[![Documentation](https://img.shields.io/badge/docs-view-green.svg)](https://fortishield.github.io/documentation)
[![Documentation](https://img.shields.io/badge/web-view-green.svg)](https://fortishield.github.io)

Deploy a Fortishield cluster with a basic indexer and dashboard stack on Kubernetes.

## Branches

* `master` branch contains the latest code, be aware of possible bugs on this branch.
* `stable` branch on correspond to the last Fortishield stable version.


## Documentation

## Amazon EKS development

To deploy a cluster on Amazon EKS cluster read the instructions on [instructions.md](instructions.md).
Note: For Kubernetes version 1.23 or higher, the assignment of an IAM Role is necessary for the CSI driver to function correctly. Within the AWS documentation you can find the instructions for the assignment: https://docs.aws.amazon.com/eks/latest/userguide/ebs-csi.html
The installation of the CSI driver is mandatory for new and old deployments if you are going to use Kubernetes 1.23 for the first time or you need to upgrade the cluster.

## Local development

To deploy a cluster on your local environment (like Minikube, Kind or Microk8s) read the instructions on [local-environment.md](local-environment.md).

## Directory structure

    ├── CHANGELOG.md
    ├── cleanup.md
    ├── envs
    │   ├── eks
    │   │   ├── dashboard-resources.yaml
    │   │   ├── indexer-resources.yaml
    │   │   ├── kustomization.yml
    │   │   ├── storage-class.yaml
    │   │   ├── fortishield-master-resources.yaml
    │   │   └── fortishield-worker-resources.yaml
    │   └── local-env
    │       ├── indexer-resources.yaml
    │       ├── kustomization.yml
    │       ├── storage-class.yaml
    │       └── fortishield-resources.yaml
    ├── instructions.md
    ├── LICENSE
    ├── local-environment.md
    ├── README.md
    ├── upgrade.md
    ├── VERSION
    └── fortishield
        ├── base
        │   ├── storage-class.yaml
        │   └── fortishield-ns.yaml
        ├── certs
        │   ├── dashboard_http
        │   │   └── generate_certs.sh
        │   └── indexer_cluster
        │       └── generate_certs.sh
        ├── indexer_stack
        │   ├── fortishield-dashboard
        │   │   ├── dashboard_conf
        │   │   │   └── opensearch_dashboards.yml
        │   │   ├── dashboard-deploy.yaml
        │   │   └── dashboard-svc.yaml
        │   └── fortishield-indexer
        │       ├── cluster
        │       │   ├── indexer-api-svc.yaml
        │       │   └── indexer-sts.yaml
        │       ├── indexer_conf
        │       │   ├── internal_users.yml
        │       │   └── opensearch.yml
        │       └── indexer-svc.yaml
        ├── kustomization.yml
        ├── secrets
        │   ├── dashboard-cred-secret.yaml
        │   ├── indexer-cred-secret.yaml
        │   ├── fortishield-api-cred-secret.yaml
        │   ├── fortishield-authd-pass-secret.yaml
        │   └── fortishield-cluster-key-secret.yaml
        └── fortishield_managers
            ├── fortishield-cluster-svc.yaml
            ├── fortishield_conf
            │   ├── master.conf
            │   └── worker.conf
            ├── fortishield-master-sts.yaml
            ├── fortishield-master-svc.yaml
            ├── fortishield-workers-svc.yaml
            └── fortishield-worker-sts.yaml

## Contribute

If you want to contribute to our project please don't hesitate to send a pull request. You can also join our users [mailing list](https://groups.google.com/d/forum/fortishield) or the [Fortishield Slack community channel](https://fortishield.github.io/community/join-us-on-slack/) to ask questions and participate in discussions.

## Credits and Thank you

Based on the previous work from JPLachance [coveo/fortishield-kubernetes](https://github.com/coveo/fortishield-kubernetes) (2018/11/22).

## License and copyright

FORTISHIELD
Copyright (C) 2016, Fortishield Inc.  (License GPLv2)

## References

* [Fortishield website](http://fortishield.github.io)
