# Running Pods using `kubelet`

## Remarks

- Configure [`kubelet`](https://kubernetes.io/docs/concepts/overview/components/#kubelet), which is the main Kubernetes agent running in a worker node.
  - It takes Pod definitions and ensures that the containers are running and healthy.
- `kubelet` is confgured in a *stand-alone* mode, which monitors a given folder for Pod definitions.
  - Check the supplied arguments in the `kubelet.service` file.

## Run it!

1. Start the VM and wait for the configuration. It will take a while.
    ```
    vagrant up
    ```
1. SSH into the machine.
    ```
    vagrant ssh
    ```
1. Check that no containers are running on Docker.
    ```
    docker ps -a
    ```
1. Copy the test pod definition into the `manifests` folder - watched by Kubelet - and check that the containers are created. It may take a while due to image downloads.
    - In addition to the two containers in the Pod definition, a third container is created by `kubelet`. This is an infrastructure container that allows that all containers in the Pod can be accessed using the same IP address.
    ```
    cp pod.yaml manifests
    docker ps
    ```
1. Get the IP of the Pod - actually the IP of the infrastructure container created by Kubelet - and check that it is responding.
    ```
    POD_IP=$(docker inspect --format '{{ .NetworkSettings.IPAddress  }}' `docker ps -q -f name=POD`)
    echo $POD_IP
    curl $POD_IP
    ```
1. Remove the test pod definition and check that containers are removed. It may take a bit.
    ```
    rm manifests/pod.yaml
    docker ps
    ```
1. Close the SSH connection and destroy the VM
    ```
    logout
    vagrant destroy -f
    ```