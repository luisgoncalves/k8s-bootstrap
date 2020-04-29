Vagrant.configure("2") do |config|
    config.vm.box = "hashicorp/bionic64"
    
    # Install docker and download common images
    config.vm.provision "docker" do |docker|
    end

    # Extract k8s binaries
    $k8s_binaries = <<-SCRIPT
    tar xvzf /vagrant/kubernetes-server-linux-amd64.tar.gz
    SCRIPT
    config.vm.provision "shell", inline: $k8s_binaries, name: "extract k8s binaries"

    # Run etcd
    $etcd = <<-SCRIPT
    sudo mkdir /var/lib/etcd
    docker run \
        -d \
        --name etcd-server \
        --volume=/var/lib/etcd:/etcd-data \
        --net=host \
        --restart unless-stopped \
        quay.io/coreos/etcd \
        /usr/local/bin/etcd \
        --data-dir=/etcd-data \
        --name etcd-server
    SCRIPT
    config.vm.provision "shell", inline: $etcd, name: "etcd"

    # Config file for the different components that access the api-server
    config.vm.provision "shell", inline: "sudo mkdir /etc/kubernetes && sudo cp /vagrant/kubeconfig /etc/kubernetes/kubeconfig"

    # Key and certificate for service accounts
    $service_accounts_key = <<-SCRIPT
    openssl req \
       -newkey rsa:2048 -nodes -keyout service-accounts.key \
       -x509 -days 365 -out service-accounts.crt \
       -subj "/C=PT/O=Kubernetes/CN=service-accounts"
    sudo mv service-accounts.crt /etc/kubernetes/service-accounts.crt
    sudo mv service-accounts.key /etc/kubernetes/service-accounts.key
    SCRIPT
    config.vm.provision "shell", inline: $service_accounts_key, name: "service-accounts key"

    # Run kube-apiserver
    $api_server = <<-SCRIPT
    sudo cp /vagrant/static-tokens.csv /etc/kubernetes/static-tokens.csv
    sudo docker load -i kubernetes/server/bin/kube-apiserver.tar
    docker run \
        -d \
        --name=kube-apiserver \
        --volume=/var/run/kubernetes:/var/run/kubernetes \
        --volume=/etc/kubernetes:/etc/kubernetes:ro \
        --net=host \
        --restart unless-stopped \
        k8s.gcr.io/kube-apiserver-amd64:`cat kubernetes/server/bin/kube-apiserver.docker_tag` \
        kube-apiserver \
        --service-cluster-ip-range=10.0.0.0/16 \
        --etcd-servers=http://127.0.0.1:2379 \
        --token-auth-file=/etc/kubernetes/static-tokens.csv \
        --service-account-key-file=/etc/kubernetes/service-accounts.key
    while [[ "$(curl -s -o /dev/null -w ''%{http_code}'' http://localhost:8080/healthz)" != "200" ]]; do sleep 5; done
    SCRIPT
    config.vm.provision "shell", inline: $api_server, name: "api-server"

    # Run kube-scheduler
    $scheduler = <<-SCRIPT
    sudo docker load -i kubernetes/server/bin/kube-scheduler.tar
    docker run \
        -d \
        --name=kube-scheduler \
        --volume=/etc/kubernetes:/etc/kubernetes:ro \
        --net=host \
        --restart unless-stopped \
        k8s.gcr.io/kube-scheduler-amd64:`cat kubernetes/server/bin/kube-scheduler.docker_tag` \
        kube-scheduler \
        --kubeconfig=/etc/kubernetes/kubeconfig
    SCRIPT
    config.vm.provision "shell", inline: $scheduler, name: "scheduler"

    # Run kube-controller-manager
    $controller_manager = <<-SCRIPT
    sudo docker load -i kubernetes/server/bin/kube-controller-manager.tar
    docker run \
        -d \
        --name=kube-controller-manager \
        --volume=/etc/kubernetes:/etc/kubernetes:ro \
        --net=host \
        --restart unless-stopped \
        k8s.gcr.io/kube-controller-manager-amd64:`cat kubernetes/server/bin/kube-controller-manager.docker_tag` \
        kube-controller-manager \
        --kubeconfig=/etc/kubernetes/kubeconfig \
        --service-account-private-key-file=/etc/kubernetes/service-accounts.key
    SCRIPT
    config.vm.provision "shell", inline: $controller_manager, name: "controller-manager"

    # Run kube-proxy
    $proxy = <<-SCRIPT
    sudo docker load -i kubernetes/server/bin/kube-proxy.tar
    docker run \
        -d \
        --name=kube-proxy \
        --volume=/etc/kubernetes:/etc/kubernetes:ro \
        --net=host \
        --restart unless-stopped \
        --privileged \
        k8s.gcr.io/kube-proxy-amd64:`cat kubernetes/server/bin/kube-proxy.docker_tag` \
        kube-proxy \
        --kubeconfig=/etc/kubernetes/kubeconfig
    SCRIPT
    config.vm.provision "shell", inline: $proxy, name: "proxy"

    # Run kubelet
    config.vm.provision "shell", inline: "sudo cp /vagrant/kubelet-config.yaml /etc/kubernetes/kubelet-config.yaml"
    config.vm.provision "shell", inline: "sudo cp /vagrant/kubelet.service /etc/systemd/system/kubelet.service"

    $kubelet = <<-SCRIPT
    chmod +x kubernetes/server/bin/kubelet
    mv kubernetes/server/bin/kubelet /usr/bin
    sudo systemctl daemon-reload
    sudo systemctl enable kubelet
    sudo systemctl start kubelet
    SCRIPT
    config.vm.provision "shell", inline: $kubelet, name: "kubelet"
    # sudo kubelet --kubeconfig /etc/kubernetes/kubeconfig --config /etc/kubernetes/kubelet-config.yaml

    # Install kubectl
    $kubectl = <<-SCRIPT
    chmod +x kubernetes/server/bin/kubectl
    mv kubernetes/server/bin/kubectl /usr/bin
    SCRIPT
    config.vm.provision "shell", inline: $kubectl, name: "kubectl"
    config.vm.provision "file", source: "kubeconfig", destination: ".kube/config"

    # Generate a kubeconfig file to be used on the host
    $host_kubeconfig = <<-SCRIPT
    mkdir /vagrant/.out
    CLUSTER_IP=$(hostname -I | cut -d' ' -f1)
    kubectl config set-cluster vagrant-$HOSTNAME \
        --server=https://$CLUSTER_IP:6443 \
        --certificate-authority=/var/run/kubernetes/apiserver.crt \
        --embed-certs=true \
        --kubeconfig=/vagrant/.out/kubeconfig
    kubectl config set-credentials admin \
        --token=admin-token \
        --kubeconfig=/vagrant/.out/kubeconfig
    kubectl config set-context default \
        --cluster=vagrant-$HOSTNAME \
        --user=admin \
        --kubeconfig=/vagrant/.out/kubeconfig
    kubectl config use-context default --kubeconfig=/vagrant/.out/kubeconfig
    SCRIPT
    config.vm.provision "shell", inline: $host_kubeconfig, name: "host kubeconfig"

    # Cleanup
    config.vm.provision "shell", inline: "rm -rf kubernetes", name: "cleanup"
end