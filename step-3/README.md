# Add `kube-scheduler`

## Remarks 

- Configure [`kube-scheduler`](https://kubernetes.io/docs/concepts/overview/components/#kube-scheduler) so that Pods are automatically assigned to a node
- The Pod [definition](pod.yaml) for this step:
  1. Doesn't set the node name to which the Pod should be deployed.
  1. Tolerates the `node.kubernetes.io/not-ready:NoSchedule` [taint](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/). This is needed because Kubernetes seems to taint the nodes while the control plane is not fully setup.
- The *ServiceAccount* [admission controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/) is still disabled, as in the previous step.

## Run it!

1. Start the VM and wait for the configuration. Then SSH into it.
    ```
    vagrant up
    vagrant ssh
    ```
1. Check that `etcd`, `kube-apiserver` and `kube-scheduler` are running as Docker containers.
    ```
    docker ps
    ```
1. Create the test pod using `kubectl` and check that it is running. It may take a while for containers to be created due to image downloads.
    - This means that `kube-scheduler` picked up the new pod and assigned a node to it.
    ```
    kubectl create -f pod.yaml
    kubectl get pods
    docker ps
    ```
1. Check that the pod's recent events include its scheduling.
    ```
    kubectl describe pod nginx | grep -A5 Events
    ```
1. Close the SSH connection and destroy the VM
    ```
    logout
    vagrant destroy -f
    ```