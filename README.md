# Simple single-node Kubernetes cluster

Bootstrap a single-node Kubernetes cluster in a Vagrant Ubuntu VM, by manually configuring each [component](https://kubernetes.io/docs/concepts/overview/components).

This exercise was part of my "learning Kubernetes" experience. The resulting cluster is functional, although incomplete. Some remarks:
  - All components run as docker containers, except for Kubelet, which runs using systemd.
  - TLS client authentication between components and api-server is not configured.
  - CoreDNS is not configured.
  - Authorization mode on the api-server is set to `AlwaysAllow` (no RBAC).

## Running the cluster

1. Download the Kubernetes server binaries from the [releases](
https://github.com/kubernetes/kubernetes/releases) page into the current directory. Tested using v1.18.2.
    ```
    wget https://dl.k8s.io/v1.18.2/kubernetes-server-linux-amd64.tar.gz
    ```
    - This is not part of the Vagrantfile to avoid the long download on each `vagrant up` (in case changes are made to the Vagrantfile).
    - The VM setup extracts the binaries using the `/vagrant` folder (mounts the current directory into the VM).
1. Start the VM and wait for the configuration. It will take a while.
    ```
    vagrant up
    ```
1. Deploy the test pod to the cluster.
    ```
    kubectl apply -f manifests/nginx-truncator.yaml --kubeconfig=.out/kubeconfig
    ```

    - The setup script generates a `kubeconfig` file to connect to the cluster. It uses a [static token](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#static-token-file) that is added as part of the api-server configuration.
    - If you don't have `kubectl` in your host you can `vagrant ssh` into the VM and run the command there.

    ```
    kubectl apply -f /vagrant/manifests/nginx-truncator.yaml
    ```
1. Wait until the pods are running
    ```
    kubectl get pods --kubeconfig=.out/kubeconfig
    ```
1. Check that the test pod is responding. Use the actual IP for your VM.
    ```
    curl http://{vm-ip}:30006
    ```
1. Take a look at the docker containers running in the VM. You'll see the containers for the test pod as well as the containers running the Kubernetes components.
    ```
    vagrant ssh
    $ docker ps
    $ logout
    ```
1. Remove the test pod
    ```
    kubectl delete -f manifests/nginx-truncator.yaml --kubeconfig=.out/kubeconfig
    kubectl get pods --kubeconfig=.out/kubeconfig
    ```
1. Remove the VM
    ```
    vagrant destroy
    ```

## Credits

Heavily inspired by:

- https://github.com/joshuasheppard/k8s-by-component
- https://github.com/kelseyhightower/kubernetes-the-hard-way