## How to use Janus IDP

Instructions: https://github.com/janus-idp/backstage-showcase/blob/main/showcase-docs/helm-chart.md

## How To

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
- Create a values.yml file to override helm's default values
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
  #args:
  #  - '--config'
  #  - '/app/app-config.yaml'
  extraAppConfig:
    - filename: app-config.yaml
      configMapRef: app-config
ingress:
  enabled: true
  host: idp.127.0.0.1.nip.io
EOF
```
- Create also your backstage `app-config.yaml` file
```bash
cat <<EOF > $(pwd)/app-config.yaml
backend:
  auth:
    keys:
      - secret: temp
  database:
    client: better-sqlite3
    connection: ':memory:'

techdocs:
  builder: 'local' # Alternatives - 'external'
  generator:
    runIn: 'local' # Alternatives - 'local'
  publisher:
    type: 'local'

kubernetes:
  serviceLocatorMethod:
    type: 'multiTenant'
  clusterLocatorMethods:
    - type: 'config'
      clusters:
        - url: https://kubernetes.default.svc
          name: kind
          authProvider: 'serviceAccount'
          skipTLSVerify: true
          skipMetricsLookup: true
          serviceAccountToken: /var/run/secrets/kubernetes.io/serviceaccount/token

catalog:
  import:
    entityFilename: catalog-info.yaml
    pullRequestBranchName: backstage-integration
  rules:
    - allow: [Component, System, Group, Resource, Location, Template, API]
  locations:
    - type: url
      target: https://github.com/janus-idp/backstage-showcase/blob/main/catalog-entities/all.yaml
    - type: url
      target: https://github.com/janus-idp/software-templates/blob/main/showcase-templates.yaml

integrations:
  bitbucketServer:
    - host: bitbucket.com
      apiBaseUrl: temp
      username: temp
      password: temp
EOF
```
- And deploy it as ConfigMap
```bash
kubectl delete configmap app-config -n idp
kubectl create configmap app-config -n idp \
  --from-file=app-config.yaml=$(pwd)/app-config.yaml
  
kubectl rollout restart deployment/janus-idp-backstage -n idp

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
helm uninstall janus-idp -n idp
```
