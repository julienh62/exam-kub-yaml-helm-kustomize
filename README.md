Exécutez la commande suivante pour désinstaller K3s proprement :

sudo /usr/local/bin/k3s-killall.sh sudo /usr/local/bin/k3s-uninstall.sh

Cela arrêtera tous les services K3s et supprimera les fichiers liés à K3s. 2. Vérifier que K3s a été complètement désinstallé

Assurez-vous que le service K3s a été supprimé :

sudo systemctl status k3s

Cela doit renvoyer que le service n'existe plus ou est inactif. 3. Supprimer les fichiers de configuration et de données K3s

Vous pouvez également supprimer les fichiers restants dans /etc/rancher/k3s/ et /var/lib/rancher/k3s/ :

sudo rm -rf /etc/rancher/k3s sudo rm -rf /var/lib/rancher/k3s

Cela supprimera toute configuration et tout état laissé par K3s. 4. Supprimer les containers Docker/Kubernetes (si nécessaire)

Si vous souhaitez également nettoyer les conteneurs Docker (en cas de réutilisation de Docker), vous pouvez supprimer tous les conteneurs et images Docker :

sudo docker system prune -a

Vérifie si des contrôleurs (Deployments, DaemonSets) gèrent les pods avec kubectl get all --all-namespaces.
Supprime ces contrôleurs pour éviter la régénération automatique des pods.
Vérifie les ressources de ton cluster avec kubectl top nodes pour voir s'il y a suffisamment de ressources pour démarrer les pods.

Cela devrait t'aider à résoudre la situation où les pods restent bloqués en "Pending". Vous avez dit : root@ubuntu:~# kubectl get all --all-namespaces NAMESPACE NAME READY STATUS RESTARTS AGE kube-system pod/coredns-ccb96694c-r99qs 0/1 Pending 0 6m25s kube-system pod/helm-delete-traefik-crd-7n69q 0/1 Pending 0 5m14s kube-system pod/helm-delete-traefik-drnd6 0/1 Pending 0 4m22s kube-system pod/helm-install-traefik-crd-jw4g6 0/1 Pending 0 3m49s kube-system pod/helm-install-traefik-pcqkf 0/1 Pending 0 3m15s kube-system pod/local-path-provisioner-5b5f758bcf-qmvxf 0/1 Pending 0 2m18s kube-system pod/metrics-server-7bf7d58749-hbjfw 0/1 Pending 0 98s

NAMESPACE NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE default service/kubernetes ClusterIP 10.43.0.1 443/TCP 25m kube-system service/kube-dns ClusterIP 10.43.0.10 53/UDP,53/TCP,9153/TCP 25m kube-system service/metrics-server ClusterIP 10.43.47.21 443/TCP 25m

NAMESPACE NAME READY UP-TO-DATE AVAILABLE AGE kube-system deployment.apps/coredns 0/1 1 0 21m kube-system deployment.apps/local-path-provisioner 0/1 1 0 21m kube-system deployment.apps/metrics-server 0/1 1 0 21m

NAMESPACE NAME DESIRED CURRENT READY AGE kube-system replicaset.apps/coredns-ccb96694c 1 1 0 21m kube-system replicaset.apps/local-path-provisioner-5b5f758bcf 1 1 0 21m kube-system replicaset.apps/metrics-server-7bf7d58749 1 1 0 21m

NAMESPACE NAME STATUS COMPLETIONS DURATION AGE kube-system job.batch/helm-delete-traefik Running 0/1 25m 25m kube-system job.batch/helm-delete-traefik-crd Running 0/1 25m 25m kube-system job.batch/helm-install-traefik Running 0/1 25m 25m kube-system job.batch/helm-install-traefik-crd Running 0/1 25m 25m root@ubuntu:~# ChatGPT a dit :

Il semble que de nombreux pods soient en statut "Pending", ce qui peut être dû à plusieurs raisons possibles :

Les contrôleurs comme les Deployments sont toujours actifs : Ces contrôleurs tentent de recréer les pods, même s'ils sont en état "Pending". Cela inclut des objets comme coredns, local-path-provisioner et metrics-server.

Les jobs sont en cours d'exécution : Des jobs comme helm-delete-traefik et helm-install-traefik semblent être en cours, ce qui pourrait aussi bloquer certaines ressources ou empêcher une suppression propre.

Actions à entreprendre :

    Supprimer les Deployments associés aux pods en "Pending"

Les pods que tu vois sont gérés par des Deployments (comme coredns, local-path-provisioner, metrics-server). Pour éviter qu'ils ne soient recréés à chaque suppression de pod, il faut supprimer ces Deployments.

kubectl delete deployment coredns -n kube-system kubectl delete deployment local-path-provisioner -n kube-system kubectl delete deployment metrics-server -n kube-system

    Supprimer les Jobs en cours

Les Jobs comme helm-delete-traefik et helm-install-traefik sont en cours d'exécution. S'ils ne sont plus nécessaires, tu peux les supprimer pour libérer des ressources :

kubectl delete job helm-delete-traefik -n kube-system kubectl delete job helm-delete-traefik-crd -n kube-system kubectl delete job helm-install-traefik -n kube-system kubectl delete job helm-install-traefik-crd -n kube-system
