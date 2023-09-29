# Install the Vault Helm chart

```
$ helm repo add hashicorp https://helm.releases.hashicorp.com
$ helm repo update
$ helm install vault hashicorp/vault --set "server.dev.enabled=true"
$ helm get pods
```

# Set a secret in Vault
```
$ kubectl exec -it vault-0 -- /bin/sh
$ vault secrets enable -path=internal kv-v2
$ vault kv put internal/database/config username="wezvatech-dbadmin" password="learndevopseasy"
$ vault kv get internal/database/config
```

# Configure Kubernetes authentication
```
$ vault auth enable kubernetes
$ vault write auth/kubernetes/config \
      kubernetes_host="https://$KUBERNETES_PORT_443_TCP_ADDR:443"
$ vault policy write wezvatech-admin - <<EOF
path "internal/data/database/config" {
   capabilities = ["read"]
}
EOF
$ vault write auth/kubernetes/role/wezvatech-admin \
      bound_service_account_names=wezvatech-admin \
      bound_service_account_namespaces=default \
      policies=wezvatech-admin \
      ttl=24h
```
# Ensure Helm 3.7+ is available
# Define a Kubernetes service account && Launch an application

```
$ kubectl apply -f deployment-wezvatech-demo.yml
$ kubectl get pods
$ kubectl exec <pod> -- ls /vault/secrets
```

# Inject secrets into the pod & apply template to structure the data

```
( wezvatech-demo, vault-agent, vault-agent-init )
$ kubectl get pods --watch
$ kubectl patch deployment wezvatech-demo --patch "$(cat patch-inject-secrets-as-template.yaml)"
$ kubectl exec <pod> -- cat /vault/secrets/database-config.txt
```

# Update the secret value

```
$ kubectl exec -it vault-0 -- /bin/sh
$ vault kv put internal/database/config username="newuser" password="newpassowrd"
$ vault kv get internal/database/config
$ kubectl logs -f <pod> -c vault-agent
```
