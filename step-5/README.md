# Add `kube-proxy` and access the cluster from the host

## Remarks 

- Configure [`kube-proxy`](https://kubernetes.io/docs/concepts/overview/components/#kube-proxy), which runs in work nodes and implements part of the Service concept, allowing Pods to be accessible.
- Expose the Deployment [using a Service](deployment.yaml). 
- Generate a *kubeconfig* file to connect to the cluster from the host machine.
  - To keep authentication simple, [static tokens](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#static-token-file) are used. The API Server is configured accordingly.

## Run it!

1. Start the VM and wait for the configuration. This time don't SSH into it.
    ```
    vagrant up
    ```
1. Verify cluster access
    ```
    kubectl get nodes --kubeconfig=.out/kubeconfig
    kubectl get pods --kubeconfig=.out/kubeconfig
    ```
1. Create the test deployment and wait until the pods are running.
    ```
    kubectl apply -f deployment.yaml --kubeconfig=.out/kubeconfig
    kubectl get pods --kubeconfig=.out/kubeconfig
    ```
1. Check that the test pod is responding. Use the actual IP for your VM.
    - This means that `kube-proxy` picked up the new Service/Endpoint and reflected its configuration on the node, so that requests to the Service are forwarded to the Pod.
    ```
    curl http://{vm-ip}:30006
    ```
1. Remove the test pod
    ```
    kubectl delete -f deployment.yaml --kubeconfig=.out/kubeconfig
    kubectl get pods --kubeconfig=.out/kubeconfig
    ```
1. Remove the VM
    ```
    vagrant destroy
    ```