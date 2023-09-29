## How to use Janus IDP

Instructions: https://github.com/janus-idp/backstage-showcase/blob/main/showcase-docs/helm-chart.md

## HowTO

- Create a local kind cluster: `curl -s -L "https://raw.githubusercontent.com/snowdrop/k8s-infra/main/kind/kind.sh" | bash -s install --delete-kind-cluster`
- Add the Helm repositories
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo add backstage https://backstage.github.io/charts
helm repo add janus-idp https://janus-idp.github.io/helm-backstage
```
- Build the image of the showcase project: https://github.com/janus-idp/backstage-showcase
```bash
git clone https://github.com/janus-idp/backstage-showcase; cd backstage-showcase
docker build -t quay.io/ch007m/janus-idp:latest . -f docker/Dockerfile
docker push quay.io/ch007m/janus-idp:latest
```
- Create a values.yml file
```bash
cat <<EOF > backstage-values.yml
backstage:
  image:
    registry: quay.io/ch007m
    repository: janus-idp
    tag: latest
  extraEnvVars:
    - name: 'APP_CONFIG_app_baseUrl'
      value: 'http://{{ .Values.ingress.host }}:7007'
    - name: 'APP_CONFIG_backend_baseUrl'
      value: 'http://{{ .Values.ingress.host }}:7007'
    - name: 'APP_CONFIG_backend_cors_origin'
      value: 'http://{{ .Values.ingress.host }}:7007'

ingress:
  enabled: true
  host: idp.127.0.0.1.nip.io
EOF
```
- Deploy the helm chart
```bash
helm upgrade -i janus-idp backstage/backstage \
  --create-namespace \
  -n idp \
  -f backstage-values.yml
```

To remove it
```bash
helm uninstall my-janus -n idp
```
