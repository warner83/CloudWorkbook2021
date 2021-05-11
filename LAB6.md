# Minikube download and installation
    curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \ && chmod +x minikube

    sudo mkdir -p /usr/local/bin/
    sudo install minikube /usr/local/bin/
  
  
# Start Minikube 
    sudo apt-get install -y conntrack
    sudo minikube start --driver=none

    sudo minikube status
  
# Interact with the cluster
    snap install kubectl –classic
    sudo kubectl cluster-info
  
  
# Get nodes
    kubectl get nodes
  
# Install load balancer module
    kubectl apply -f https://raw.githubusercontent.com/google/metallb/v0.8.3/manifests/metallb.yaml

# Check installation
    kubectl get all -n metallb-system
  
# Configure the module
  # Create a conf file config-map.yaml
    apiVersion: v1
      kind: ConfigMap
      metadata:
        namespace: metallb-system
        name: config
      data:
        config: |
          address-pools:
          - name: default
            protocol: layer2
            addresses:
            - 192.168.1.10-192.168.1.50

    kubectl create -f  config-map.yaml
    kubectl describe configmap config -n metallb-system

# Activate metrics plugin
    minikube addons enable metrics-server
