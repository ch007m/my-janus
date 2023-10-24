## How to use Janus IDP

Instructions to only install janus backstage on a cluster without the components: https://github.com/janus-idp/backstage-showcase/blob/main/showcase-docs/helm-chart.md

## Prerequisite

- Install Nodejs, npm, yarn and backstage cli `npm install -g @backstage/cli`
- Create a GitHub App using `backstage-cli create-github-app` - [see](https://github.com/organizations/ch007m/settings/apps/backstage-janus-idp)
- Install smee: `npm install --global smee-client` (optional)

## How To build and deploy Janus

- Build the image of the showcase project: https://github.com/janus-idp/backstage-showcase
```bash
git clone https://github.com/janus-idp/backstage-showcase; cd backstage-showcase
docker build -t quay.io/ch007m/janus-idp:latest . -f docker/Dockerfile
docker push quay.io/ch007m/janus-idp:latest
```
- Build locally the IDPBuilder or use the executable when it will be released
```bash
git clone https://github.com/cnoe-io/idpbuilder.git; cd ipdbuilder
make
./idpbuilder create --buildName local --recreate
```
- Wait till the ArgoCD applications are sync/health and ingress controller up and running
```bash
kubectl get apps -A
NAMESPACE   NAME                                       SYNC STATUS   HEALTH STATUS
argocd      idpbuilder-local-gitserver-argocd          Synced        Healthy
argocd      idpbuilder-local-gitserver-backstage       Synced        Healthy
argocd      idpbuilder-local-gitserver-crossplane      Synced        Healthy
argocd      idpbuilder-local-gitserver-nginx-ingress   Synced        Healthy

kubectl rollout status deployment/ingress-nginx-controller -n ingress-nginx
deployment "ingress-nginx-controller" successfully rolled out
```
- Deploy the needed ArgoCD components: gitea, certmanager, vault
```bash
kubectl apply -f ./argocd/idp-bootstrap.yml
```

## TODO

```bash
  --set global.hosts.externalIP=10.10.10.10 \
helm repo add gitlab https://charts.gitlab.io/
helm repo update
helm upgrade --install gitlab gitlab/gitlab \
  --timeout 600s \
  --namespace gitlab \
  --create-namespace \
  --set global.hosts.domain=127.0.0.1.nip.io \
  --set certmanager-issuer.email=cmoulliard@redhat.com \
  --set certmanager.install=false \
  --set postgresql.image.tag=13.6.0
```