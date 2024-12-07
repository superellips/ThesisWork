# Kubernetes

## Tjänster och verktyg

- Linode Kubernetes Engine
- Linode CLI
- ArgoCD & ArgoCD CLI
- kubectl
- jquery

## Installera och konfigurera ArgoCD

### På linodes hemsida

1. Navigera till https://cloud.linode.com/kubernetes/clusters
2. Tryck _Create_
3. Mina val är enligt följande:
    - _Label:_ linode-1
    - _Region:_ se-sto (stockholm)
    - _HA Control Plane:_ no
    - _Control Plane ACL:_ disabled
    - _Node Pool:_
        - 1x Shared CPU - Linode 2GB
4. Tryck på _Create Cluster_

### Github container registry (ghcr.io)

1. Skapa en accesskey
2. Lägg till den i kubernetes: `kubectl --kubeconfig k8s-config.yaml create secret docker-registry regcred --docker-server=ghcr.io --docker-username=<GH_NAME> --docker-password <ACCESS_TOKEN> --docker-email=<EMAIL>`

### I en terminal

1. Generera en kubernetes konfigurationsfil med: `linode lke kubeconfig-view $(linode lke clusters-list --json | jq .[0].id) --text | sed -n '2 p' | base64 --decode > k8s-config.yaml`
2. Installera ArgoCD med: `kubectl apply --kubeconfig k8s-config.yaml -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`
3. När ArgoCD är installarat, gör _argocd-server_ tillgängligt på `localhost:8080` med _port-forward:_ `kubectl --kubeconfig k8s-config.yaml -n argocd port-forward svc/argocd-server 8080:443`
4. Logga in argocd-cli med: `argocd login --kube-context k8s-config.yaml --insecure --username admin --password $(argocd admin initial-password --kubeconfig=k8s-config.yaml --kube-context=$(kubectl config current-context --kubeconfig k8s-config.yaml) -n argocd | sed -n '1 p') localhost:8080`
5. Updatera lösenordet med: `argocd account update-password --current-password=$(argocd admin initial-password --kubeconfig=k8s-config.yaml --kube-context=$(kubectl config current-context --kubeconfig k8s-config.yaml) -n argocd | sed -n '1 p')`
6. Lägg till anslutningsuppgiter för repositories (körs en gång för varje repositry): `argocd repo add git@github.com:<GH_USER>/<REPO_NAME>.git --insecure-ignore-host-key --ssh-private-key-path ~/.ssh/id_rsa`
7. Skapa en accesskey för ghcr.io (github container registry) och lägg till den som en hemlighet i kubernetes: `kubectl --kubeconfig k8s-config.yaml create secret docker-registry regcred --docker-server=ghcr.io --docker-username=<GH_NAME> --docker-password <ACCESS_TOKEN> --docker-email=<EMAIL>`
8. Skapa applikationer i ArgoCD (körs en gång för varje repository): `cd <REPO_FOLDER> && argocd app create <APP_NAME> --repo=$(git remote get-url origin) --path='./manifests' --auto-prune --dest-server https://kubernetes.default.svc --dest-namespace guestbook --sync-policy auto --sync-option='CreateNamespace=true' --self-heal --project default --kube-context=$(kubectl config current-context --kubeconfig k8s-config.yaml) && cd ..`

