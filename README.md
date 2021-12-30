# K8S-Exercise
MiniKube (single Node/VM) and KubeADM (Multi Node/VM)

# Kubernetes

Dir-minikube:

    components to run in a minikube cluster.
    minikube cluster: One Node/VM Cluster

    animals:
        3 Pods (bear, hare, moose) - with an image that runs a webpage and shows a picture of the suggested animal
        3 Services (NodePort) - exposes the Pods on port 80
        Ingress - exposes every service on a specific domain name
    volumes:
        StorageClass to put Persistant-Volume and PV-Claim in it
        Persistant Volume that is mounted localy on "/tmp/capture"
        PV-Claim for future pod to use
        Pod, connects PV-Claim to it and mounts it on /usr/share/nginx/html (for image nginx:alpine webpage use)
        Service
        Ingress - exposes every service on a specific domain name
    webtest:
        Deployment with 3 replicas of a pod - image: webpage that shows internal IP of the Pod
        Service - LoadBalancer (when hiting REFRESH in the webpage - it will change IP to the next Pod)
        Ingress - exposes every service on a specific domain name

Dir-kubeadm:

    components to run in a multy Node/VM cluster.

    had to add NGINX reverse-proxy for it to work and redirect from specific Domain-name to Service-Port
    add a server per every domain name - file - nginx_sites-available_default
    actual location (after installing nginx) - /etc/nginx/sites-available/default (needs to be a link to "/etc/nginx/sites-enabled")
    Doing this - we won't need to use "Ingress" any longer

    volumes:
        changed the "accessModes" to "ReadWriteMany"
        path: /mnt/common/tmp/capture # have to create the directory on NFS directory mounted on all nodes
        