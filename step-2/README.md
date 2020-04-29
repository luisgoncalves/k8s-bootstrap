# Add the Kubernetes API Server

## Remarks 

- Configure [`kube-apiserver`](https://kubernetes.io/docs/concepts/overview/components/#kube-apiserver)
  - It is the frontend for the Kubernetes control plane and is used to manage the different [Kubernetes objects](https://kubernetes.io/docs/concepts/#kubernetes-objects).
  - Also configure `etcd` - the data store used by the API Server.
- The *ServiceAccount* [admission controller](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/) is disabled in the API Server.
  - This admission controller ensures that the Service Account configured for the Pod exists (or the default, if none is specified).
  - Default service accounts are created by another component which is not yet in place. This means that Pod creation would fail.
  - For this step, it's simpler to disable the admission controller.
- `kubelet` is now configured to connect to the API Server.
  - Check the supplied arguments in the `kubelet.service` file.
- The Pod [definition](pod.yaml) for this step explicitly sets the node name to which the Pod should be deployed.
  - Assigning Pods to nodes is the responsibility of another component which is not yet in place.
- Configure `kubectl`, which uses the API Server under the covers.

## Run it!

1. Start the VM and wait for the configuration. Then SSH into it.
    ```
    vagrant up
    vagrant ssh
    ```
1. Check that `etcd` and `kube-apiserver` are running as Docker containers.
    ```
    docker ps
    ```
1. Create the test pod using `kubectl`. It may take a while for containers to be created due to image downloads.
    ```
    kubectl create -f pod.yaml
    kubectl get pods
    docker ps
    ```
    - One can also get the pod directly via the API Server.
    ```
    curl http://localhost:8080/api/v1/namespaces/default/pods/nginx
    ```
    - If the pod is created without `nodeName`, it will remain in a pending state (not scheduled).

1. Get the IP of the Pod and check that it is responding.
    ```
    POD_IP=$(kubectl get pod nginx -o=jsonpath="{.status.podIP}")
    echo $POD_IP
    curl $POD_IP
    ```
1. Remove the test pod and check that containers are removed. It may take a bit.
    ```
    kubectl delete -f pod.yaml
    kubectl get pods
    docker ps
    ```
1. Close the SSH connection and destroy the VM
    ```
    logout
    vagrant destroy -f
    ```