<h2>GENERAL</h2>

GitHub repository is https://github.com/epylkkan/4.07-4.08

Changes in the directories ./backend or ./broadcaster or ./frontend or ./wiki trigger a GitHub Action
gitops-gha_be.yaml or gitops-gha_broadcaster.yaml or gitops-gha_fe.yaml or gitops-gha_wiki.yaml respectively

GitHub Actions need to be manually started  "on workflow_dispatch" instead "on push"

Each Github action...

1) creates a new image in one of the following Docker Hub repositories: 
epylkkan/gitops-be or epylkkan/gitops-broadcaster or epylkkan/gitops-fe or epylkkan/gitops-wiki

and 

2) deploys the new image into a running cluster


Sealed secrets were created in the same way as described at 
https://aws.amazon.com/blogs/opensource/managing-secrets-deployment-in-kubernetes-using-sealed-secrets


<h2>COMMANDS</h2>

k3d cluster delete

k3d cluster create --port '8082:30080@agent[0]' -p 8081:80@loadbalancer --agents 2

kubectl create namespace todo

kubectl get nodes --output wide

git clone https://github.com/epylkkan/4.07-4.08

curl -s https://toolkit.fluxcd.io/install.sh | sudo bash

flux check --pre

export GITHUB_TOKEN=...

flux bootstrap github --owner=epylkkan --repository=epylkkan/4.07-4.08 --personal --private=false --token-auth

flux logs --level=error



<h2>START NATS, POSTGRE AND INGRESS (no own code)</h2>

<h4>./4.07-4.08/manifests</h4>

kubectl create configmap nats-config --from-file /etc/nats-config/nats.conf -n todo // copied also to ./4.06/
nats/conf -directory

kubectl apply -k .



<h2>SEALED SECRETS</h2>

<h4>Steps</h4>

https://aws.amazon.com/blogs/opensource/managing-secrets-deployment-in-kubernetes-using-sealed-secrets

wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.10.0/kubeseal-linux-amd64 -O kubeseal

sudo install -m 755 kubeseal /usr/local/bin/kubeseal

wget https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.12.1/controller.yaml

kubectl apply -f controller.yaml

kubectl get po -n kube-system

k logs sealed-secrets-controller-5cbc6dc7d8-jsxqr -n kube-system

kubectl port-forward sealed-secrets-controller-7f6996f967-rcthp 8080:8080 -n kube-system

(curl -O localhost:8080/v1/public-key-cert.pem)

kubectl get secret -n kube-system -l sealedsecrets.bitnami.com/sealed-secrets-key -o yaml


<h4>Create sealed secret for posgre</h4>

kubectl kustomize . > secret.yaml

kubeseal --format=yaml < secret.yaml > postgre-sealed-secret.yaml


(kubeseal --format=yaml --cert=public-key-cert.pem < secret.yaml > sealed-secret.yaml // not needed)

(kubectl apply -f sealed-secret.yaml // not used)


<h4>Create sealed secret for telegram</h4>

kubectl kustomize . > secret.yaml

kubeseal --format=yaml < secret.yaml > broadcaster-sealed-secret.yaml

(kubeseal --format=yaml --cert=public-key-cert.pem < secret.yaml > sealed-secret.yaml // not needed)

(kubectl apply -f sealed-secret.yaml // not used)



