
# Instalar az-cli e instalar extensiones
```bash
az extension add --name aksarc
az extension add --name k8s-runtime --upgrade
``` 

# Obtener kubeconfig de admin y usuario

```bash
az aks install-cli
az aksarc get-credentials --name akswelab01 --resource-group rg-labhci-aks-sanbox-sc --admin
az aksarc get-credentials --name akswelab01 --resource-group rg-labhci-aks-sanbox-sc

kubectl create clusterrolebinding rrodriguez5-cluster-admin --clusterrole cluster-admin --user RRODRIGUEZ5@gcelsa.com
```

# Instalar load-balancer para ingress

Primero hay que registrar en la subscripcion la feature Microsoft.Kubernetes.Runtime

```


# Instalar ingress en modo nodeport
```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
kubectl create ns ingress-nginx
openssl req -nodes -newkey rsa:4096 -keyout default-ssl.key.pem -new -x509 -days 36500 -sha256 -out default-ssl.cert.pem -subj "/C=ES/ST=ES/L=ES/O=CELSA/OU=CELSA/CN=default-ssl"

kubectl -n ingress-nginx create secret tls default-ssl-secret --key=default-ssl.key.pem --cert=default-ssl.cert.pem


helm install -n ingress-nginx ingress-nginx ingress-nginx/ingress-nginx -f ingress-nginx-nodeport-values.yaml
```

# Instalar app de ejemplo
```bash
kubectl create ns test-ns
kubectl -n test-ns apply -f test-deploy.yaml
```

Comprobamos que funciona

```bash
RRODRIGUEZ5@S006-0544 MINGW64 ~/aks/aks (main)
$ curl http://test-aks.10.1.5.35.nip.io:30080 -I
HTTP/1.1 200 OK
Date: Mon, 09 Dec 2024 15:28:49 GMT
Content-Type: text/html
Content-Length: 615
Connection: keep-alive
Last-Modified: Tue, 26 Nov 2024 15:55:00 GMT
ETag: "6745ef54-267"
Accept-Ranges: bytes
```

# Instalar monitorizacion
```bash
kubectl create ns monitoring
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install -n monitoring monitoring prometheus-community/kube-prometheus-stack -f monitoring.yaml
```