# K8S bootstrap

Bootstrap a single-node Kubernetes cluster in a Vagrant Ubuntu VM, step-by-step, by manually configuring each [component](https://kubernetes.io/docs/concepts/overview/components).

## Motivation & credits

This repo is part of my "learning Kubernetes" experience and is heavily based on [Joshua Sheppard's Kubernetes by Component](https://github.com/joshuasheppard/k8s-by-component) series. Read more about my motivation and approach on [this post](https://luisfsgoncalves.wordpress.com/2020/05/01/learning-kubernetes-internals-by-configuring-a-cluster/).

Other references:

- [Kelsey Hightower - Kubernetes The Hard Way](https://github.com/kelseyhightower/kubernetes-the-hard-way)
- [Carson Anderson - Kubernetes Deconstructed](https://vimeo.com/245778144/4d1d597c5e) (Kubecon 2017)

## Remarks

The cluster resulting from the different steps in this repo is functional, although incomplete. Some remarks:
  - All components run as Docker containers, except for Kubelet, which runs using systemd.
  - TLS client authentication between components and the API Server is not configured.
  - Authorization mode on the api-server is set to `AlwaysAllow`.
  - RBAC is not configured.
  - CoreDNS is not configured.

## Prerequisites

- The cluster runs in a VM created/configured using [Vagrant](https://www.vagrantup.com/), so it needs to be installed on the host machine.
- Download the Kubernetes server binaries from the Kubernetes [releases](
https://github.com/kubernetes/kubernetes/releases) page into the current directory.
  - This is not part of the Vagrantfile to avoid the long download on each `vagrant up`.
  - Tested using v1.18.2. Adapt the URL accordingly.
  ```
  wget https://dl.k8s.io/v1.18.2/kubernetes-server-linux-amd64.tar.gz
  ```
- Some knowledge of Kubernetes is required. The different [components](https://kubernetes.io/docs/concepts/overview/components) are briefly introduced as they are used.

## Run it!

Each step configures one or more components towards the full cluster configuration. There's a folder for each step, containing a short description and a `Vagrantfile` that configures a VM accordingly. The main learnings are highlighted on each step, but reading the full journey in  [Joshua Sheppard's](https://github.com/joshuasheppard/k8s-by-component) posts may also help.

- [Step 1](/step-1) - Running Pods using `kubelet`
- [Step 2](/step-2) - Add the Kubernetes API Server and access it using `kubectl`
- [Step 3](/step-3) - Add `kube-scheduler` so that Pods are automatically assigned to the node
- [Step 4](/step-4) - Add `kube-controller-manager` and start using Deployments
- [Step 5](/step-5) - Add `kube-proxy` and access the cluster from the host machine