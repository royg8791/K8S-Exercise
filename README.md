# K8S-Exercise
MiniKube (single Node/VM) and KubeADM (Multi Node/VM)

# Kubernetes

Instalation:

    minikube - https://minikube.sigs.k8s.io/docs/start/
    
    kubeadm, kubectl, kubelet - https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl


/minikube/:

    minikube cluster: One Node/VM Cluster

    animals:
        Pods                - 3 (bear, hare, moose):
            with an image that runs a webpage and shows a picture of the suggested animal
        Services            - 3 (NodePort):
            exposes the Pods on port 80
        Ingress             - 1 (animals):
            exposes every service on a specific domain name
    volumes:
        StorageClass        - puts Persistant-Volume and PV-Claim in it
        Persistant Volume   - that is mounted localy on "/tmp/capture"
        PV-Claim            - for future pod to use
        Pods                - 1 (capture):
            connects PV-Claim to it and mounts it on /usr/share/nginx/html (for image nginx:alpine webpage use)
        Service             - 1 (NodePort)
        Ingress             - 1 (capture):
            exposes every service on a specific domain name
    webtest:
        Deployments         - 3 replicas of a pod:
            image: webpage that shows internal IP of the Pod
        Service             - 1 (LoadBalancer):
            when hiting REFRESH in the webpage - it will change IP to the next Pod
        Ingress             - 1 (webtest):
            exposes every service on a specific domain name

/kubeadm/:

    kubernetes cluster: Multi Node/VM Cluster

    nginx instalation: 'apt install nginx'

    Had to add NGINX reverse-proxy for it to work and redirect from specific Domain-name to Service:Port
        look at /nginx_site-available_default file
        actual location - /etc/nginx/sites-available/default (needs to be a linked to "/etc/nginx/sites-enabled")
    Doing this - we won't need to use "Ingress" any longer

    animals:
        Pods                - 3 (bear, hare, moose):
            with an image that runs a webpage and shows a picture of the suggested animal
        Services            - 3 (NodePort):
            exposes the Pods on port 80
    volumes:
        StorageClass        - puts Persistant-Volume and PV-Claim in it
        Persistant Volume   - that is mounted localy on "/tmp/capture"
            changed the "accessModes" to "ReadWriteMany"
            path: /mnt/common/tmp/capture # have to create the directory on NFS directory mounted on all nodes
        PV-Claim            - for future pod to use
        Pods                - 1 (capture):
            connects PV-Claim to it and mounts it on /usr/share/nginx/html (for image nginx:alpine webpage use)
        Service             - 1 (NodePort)
    webtest:
        Deployments         - 8 replicas of a pod:
            image: webpage that shows internal IP of the Pod
        Service             - 1 (LoadBalancer):
            when hiting REFRESH in the webpage - it will change IP to the next Pod
        