# jwt-multi-rl

Test Rate limiting based on JWT claim value

## Deploy Traefik Hub

### Create namespace

```bash
kubectl create ns traefik
```

### Create Hub token secret

```bash
kubectl create secret generic hub-license --from-literal=token="${HUB_TOKEN}" -n traefik
```

### deploy Traefik

```bash
helm upgrade --install traefik traefik/traefik --create-namespace --namespace traefik --values hub/hub-values.yaml
```

## Deploy Keycloak

### Create keycloak namespace

```bash
kubectl create ns keycloak
```

### Create keycloak admin password

```bash
kubectl create secret generic keycloak-admin --from-literal=password="${KEYCLOAK_PASSWORD}" -n keycloak
```

### Create configmap for Traefik realm

```bash
envsubst < keycloak/realm-cm.yaml | kubectl apply -f -
```

### Set ingressRoute for keycloak

```bash
envsubst < keycloak/ingress.yaml | kubectl apply -f - && sleep 30
```

### Install Keycloak with realm configuration

```bash
helm upgrade --install keycloak bitnami/keycloak --create-namespace --namespace keycloak --set keycloakConfigCli.extraEnvVars\[0\].name="KEYCLOAK_URL" --set keycloakConfigCli.extraEnvVars\[0\].value="https://keycloak.${CLUSTERNAME}.${DOMAINNAME}" --values keycloak/keycloak-values.yaml
```

## Deploy app with JWT and RL

### Create apps namespace

```bash
kubectl create ns whoami
```

### Deploy whoami, middleware and ingress

```bash
kubectl apply -f whoami/whoami.yaml
envsubst < whoami/middlewares.yaml | kubectl apply -f -
envsubst < whoami/ingress.yaml | kubectl apply -f -
```
