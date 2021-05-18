# Create a POD
# Create a pod.yaml file
    apiVersion: v1
    kind: Pod
    metadata:
        name: helloworld
        labels:
            app: helloworld
    spec:
        containers:
        - name: helloworld
          image: luksa/kubia
          ports:
          - containerPort: 8080

# Run, check and delete
    kubectl apply -f pod.yaml
    kubectl get pods
    kubectl delete pod helloworld
    kubectl get pods

# Instantiate a deployment with replicas
# Create a file deployment.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: helloworld
    spec:
      selector:
        matchLabels:
          run: helloworld
      replicas: 2
      template:
        metadata:
          labels:
            run: helloworld
        spec:
          containers:
          - name: helloworld
            image: luksa/kubia
            ports:
            - containerPort: 8080

# Run, check and delete
    kubectl apply -f deployment.yaml
    kubectl get deployments
    kubectl get pods
    kubectl delete pod helloworld-958544d5c-lrjww

# Expose a service
    kubectl expose deployment/helloworld
    kubectl get services
    kubectl describe service helloworld

# DNS service 
    kubectl get services kube-dns --namespace=kube-system
    
    kubectl run curl --image=radial/busyboxplus:curl -i --tty

    nslookup helloworld

# NodePort
    kubectl delete service helloworld
    kubectl expose deployment/helloworld --type=NodePort
    kubectl get services

# LoadBalancer
    kubectl delete service helloworld
    kubectl expose deploy helloworld --port 8080 --type LoadBalancer

# Autoscale
# Create a file autoscale.yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: php-apache
    spec:
      selector:
        matchLabels:
          run: php-apache
      replicas: 2
      template:
        metadata:
          labels:
            run: php-apache
        spec:
              containers:
              - name: php-apache
                image: k8s.gcr.io/hpa-example
                ports:
                - containerPort: 80
                resources:
                  limits:
                    cpu: "500m"
                  requests:
                    cpu: "200m"
                    
    kubectl apply -f autoscale.yaml

# Instantiate the autoscale
    kubectl autoscale deployment php-apache --cpu-percent=50 --min=1 --max=10

# Put some load 
    kubectl run --generator=run-pod/v1 -it --rm load-generator --image=busybox /bin/sh

    while true; do wget -q -O- http://php-apache.default.svc.cluster.local; done

# Check autoscale
    kubectl  get hpa


