# Add `kube-controller-manager`

## Remarks 

- Configure [`kube-controller-manager`](https://kubernetes.io/docs/concepts/overview/components/#kube-controller-manager), which runs the different [controller](https://kubernetes.io/docs/concepts/architecture/controller/) processes.
- The *ServiceAccount* admission controller is now enabled.
  - Default service accounts and their tokens are created by the *Service Account* controller process.
  - `kube-controller-manager` is configured with a key to sign Service Account tokens.
- A Deployment [definition](deployment.yaml) is now used instead of a Pod.
  - The control plane components will create the corresponding Replica Set and Pods.
  - No need to tolerate taints anymore, since the control plane is complete.

## Run it!

1. Start the VM and wait for the configuration. Then SSH into it.
    ```
    vagrant up
    vagrant ssh
    ```
1. Check that `etcd`, `kube-apiserver`, `kube-scheduler` and `kube-controller-manager` are running as Docker containers.
    ```
    docker ps
    ```
1. Check that there is a `default` service account and that it has one secret (its token).
    - This means that `kube-controller-manager` detected that there was no `default` service account and created it.
    - Actually, the `serviceaccount` controller created the service account and the `serviceaccount-token` controller created the token/secret.
    ```
    kubectl get sa
    ```
1. Create the test Deployment using `kubectl` and check that the 2 Pods are running. It may take a while for containers to be created due to image downloads.
    - This means that `kube-controller-manager` picked up the new Deployment and created the Replica Set and the Pods (this is actually done by the `deployment`, `replicaset` and `replication` controllers).
    - Then `kube-scheduler` picked up the pods and assigned a node to them.
    ```
    kubectl apply -f deployment.yaml
    kubectl get deployments
    kubectl get replicasets
    kubectl get pods
    docker ps
    ```
1. Change the number of replicas to 1 in the Deployment and update it.
    ```
    sed 's/^  replicas:.*/  replicas: 1/' deployment.yaml > deployment-1-replica.yaml
    kubectl apply -f deployment-1-replica.yaml
    kubectl get deployments
    kubectl get pods
    ```
1. Close the SSH connection and destroy the VM
    ```
    logout
    vagrant destroy -f
    ```