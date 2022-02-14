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

    used calico as CNI (Container Network Interface) - see /calico.yaml

    installed ingress-nginx via "HELM" https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx
        ```##### helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx```
        ##### helm repo update
        ##### helm install ingress-nginx ingress-nginx/ingress-nginx
        had to change "externalIPs" of the service to master node IP
        then had to add "ingressClassName: nginx" to ingress in order to use load balancer

    animals:
        Pods                - 3 (bear, hare, moose):
            with an image that runs a webpage and shows a picture of the suggested animal
        Services            - 3 (NodePort):
            exposes the Pods on port 80
        Ingress             - 1 (animals):
            exposes every service on a specific domain name (+ "ingressClassName: nginx")
    volumes:
        had to remove "taint" from master node
        StorageClass        - puts Persistant-Volume and PV-Claim in it
        Persistant Volume   - that is mounted localy on "/tmp/capture"
            changed the "accessModes" to "ReadWriteMany"
            path: /mnt/common/tmp/capture # have to create the directory on NFS directory mounted on all nodes
        PV-Claim            - for future pod to use
        Pods                - 1 (capture):
            connects PV-Claim to it and mounts it on /usr/share/nginx/html (for image nginx:alpine webpage use)
        Service             - 1 (NodePort)
        Ingress             - 1 (capture):
            exposes every service on a specific domain name (+ "ingressClassName: nginx")
    webtest:
        Deployments         - 8 replicas of a pod:
            image: webpage that shows internal IP of the Pod
        Service             - 1 (LoadBalancer):
            when hiting REFRESH in the webpage - it will change IP to the next Pod
        Ingress             - 1 (webtest):
            exposes every service on a specific domain name (+ "ingressClassName: nginx")

    DokuWiki:
        created DokuWiki Pod using HELM. see: https://www.dokuwiki.org/dokuwiki

        ##### helm install wiki k8s-at-home/dokuwiki

        Ingress
            Ingress needs to be added manually. file: dokuwiki_ingress.yml

        After creation, use Logs to monitor propper Pod creation:
        ##### kubectl logs wiki-dokuwiki-69858dc8f5-4fld4 <<<<<"pod name"

