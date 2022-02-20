# Kubernetes

Instalation:

    minikube - https://minikube.sigs.k8s.io/docs/start/

    kubeadm, kubectl, kubelet - https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#installing-kubeadm-kubelet-and-kubectl

        After creation, use Logs to monitor propper Pod creation:
        ##### kubectl logs "pod_name"

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
        ##### kubectl apply -f calico.yaml

    In order to use PersistentVolumes, master NODE needs to be untainted:
        ##### kubectl taint node k8s01 node-role.kubernetes.io/master="":NoSchedule-

    installed ingress-nginx via "HELM" https://artifacthub.io/packages/helm/ingress-nginx/ingress-nginx
        ##### helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
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
        see: https://www.dokuwiki.org/dokuwiki

        Pulled DokuWiki repository using "helm pull".
            ##### helm pull k8s-at-home/dokuwiki
            directory: "dokuwiki"

        ##### cd dokuwiki           <<< move into repository >>>
        I added Persistent Volume to the Helm chart by editing [./dokuwiki/charts/common/values.yaml] file.
            persistence:
              config:
                enabled: true
                ...
                mountPath: /config/dokuwiki/data          <<< where the data of dokuwiki is stored >>>


        I created a YAML file that contains PV and Ingress to run locally with the helm chart > "apply-locally.yaml"

        To run everything:
            ##### helm install wiki ./          <<<<< from within the repository directory >>>>>
            ##### kubectl apply -f apply-locally.yaml         <<<<< to add missing components >>>>>

    Redmine:
        see: https://www.redmine.org/

        Pulled Redmine repository using "helm pull".
            ##### helm pull bitnami/redmine
            directory: "redmine"

        ##### cd redmine           <<< move into repository >>>

        I removed Persistent Volume from "mariadb" and "postgresql" [./redmine/values.yaml] file.
        ** it needs a different provitioner in order to work.
            mariadb:
              primary:
                persistence:
                  enabled: false
                ...
            postgresql:
              persistence:
                enabled: false

        I added Ingress by editing [./redmine/values.yaml] file.
            ingress:
              enabled: true
              className: nginx                         <<<<< nginx-ingress-controller is used from k8s instalation >>>>>
              hostname: capture.nif.catchmedia.com           <<<<< domain name pointing to master node IP >>>>>

        I created a YAML file that contains PV to run locally with the helm chart > "apply-locally.yaml"

        To run everything:
            ##### helm install redmine ./          <<<<< from within the repository directory >>>>>
            ##### kubectl apply -f apply-locally.yaml         <<<<< to add missing components >>>>>

    Gitlab:
        see: https://about.gitlab.com//
        info about instalation: https://medium.com/@SergeyNuzhdin/how-to-easily-deploy-gitlab-on-kubernetes-75f5868cea78

        I built a YAML file from a git repository: https://github.com/lwolf/kubernetes-gitlab
        File: gitlab.yaml

        To run:
            ##### kubectl apply -f gitlab.yaml
        
        *** there are no persistent volumes attached to the entities, to add it follow the instructions in the instalation guide in the Git Repository linked above.
            *** in general, create PV and PVC, then change the "emptyDir: {}" to your desired persistent volume.

    Seafile:
        see: https://www.seafile.com//

        I built a YAML file from a HELM repository: n0rad/seafile
        File: seafile.yaml

        To run:
            ##### kubectl apply -f gitlab.yaml
        
        after running there are a few more steps:...
        enter the pod and start "seafile":
            ##### kubectl exec -it seafile-55575b79f9-tcwmw bash      <<<<< to enter the pod >>>>>
        once inside the pod, run /opt/seafile/seafile-server-6.3.4/setup-seafile.sh and follow the instructions:
            ##### /opt/seafile/seafile-server-6.3.4/setup-seafile.sh
                follow setup instructions:
                    # Where would you like to store your seafile data? 
                    # Note: Please use a volume with enough free space.
                    # [default: /opt/seafile/seafile-data ] 
                for path to store seafile data write: <<<<< /shared/data >>>>>
        after finishing the instalation, seafile should be accesible from webpage.
