
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

# Configurar load-balancer para el ingress

## 1. Primero hay que registrar en la subscripcion la feature Microsoft.Kubernetes.Runtime

## 2. Habilitar la extension de load-balancer para el AKS

```
C:\Users\RRODRIGUEZ5\aks\aks>az k8s-runtime load-balancer enable --resource-uri /subscriptions/6b727810-e942-4f1a-92b7-f54a21d51ce6/resourceGroups/rg-labhci-aks-sandbox-sc/providers/Microsoft.Kubernetes/connectedClusters/akswelab01
Kubernetes Runtime RP has been registered in subscription 6b727810-e942-4f1a-92b7-f54a21d51ce6...
Installing Arc Networking Extension in cluster akswelab01...
{
  "extension": {
    "aksAssignedIdentity": null,
    "autoUpgradeMinorVersion": true,
    "configurationProtectedSettings": {},
    "configurationSettings": {
      "k8sRuntimeFpaObjectId": "d272627e-602d-4d2b-95d7-fd12a879f846"
    },
    "currentVersion": "0.2.6",
    "customLocationSettings": null,
    "errorInfo": null,
    "extensionType": "microsoft.arcnetworking",
    "id": "/subscriptions/6b727810-e942-4f1a-92b7-f54a21d51ce6/resourceGroups/rg-labhci-aks-sandbox-sc/providers/Microsoft.Kubernetes/connectedClusters/akswelab01/providers/Microsoft.KubernetesConfiguration/extensions/arcnetworking",
    "identity": {
      "principalId": "75716d35-7d4d-4572-ba5e-78ec377faab8",
      "tenantId": null,
      "type": "SystemAssigned"
    },
    "isSystemExtension": false,
    "name": "arcnetworking",
    "packageUri": null,
    "plan": null,
    "provisioningState": "Succeeded",
    "releaseTrain": "Stable",
    "resourceGroup": "rg-labhci-aks-sandbox-sc",
    "scope": {
      "cluster": {
        "releaseNamespace": "kube-system"
      },
      "namespace": null
    },
    "statuses": [],
    "systemData": {
      "createdAt": "2024-12-11T09:24:22.910304+00:00",
      "createdBy": null,
      "createdByType": null,
      "lastModifiedAt": "2024-12-11T09:24:22.910304+00:00",
      "lastModifiedBy": null,
      "lastModifiedByType": null
    },
    "type": "Microsoft.KubernetesConfiguration/extensions",
    "version": null
  }
}

C:\Users\RRODRIGUEZ5\aks\aks>

```

esto crea los siguientes CRDs

```
$ kubectl get crd | grep metal
bfdprofiles.metallb.io                                 2024-12-11T09:26:18Z
bgpadvertisements.metallb.io                           2024-12-11T09:26:18Z
bgppeers.metallb.io                                    2024-12-11T09:26:18Z
communities.metallb.io                                 2024-12-11T09:26:18Z
ipaddresspools.metallb.io                              2024-12-11T09:26:18Z
l2advertisements.metallb.io                            2024-12-11T09:26:18Z
servicel2statuses.metallb.io                           2024-12-11T09:26:18Z
```



## 3. Crear el load balancer

```
C:\Users\RRODRIGUEZ5\aks\aks-hci>az k8s-runtime load-balancer create --resource-uri /subscriptions/6b727810-e942-4f1a-92b7-f54a21d51ce6/resourceGroups/rg-labhci-aks-sandbox-sc/providers/Microsoft.Kubernetes/connectedClusters/akswelab01 --load-balancer-name lb-ingress --addresses "10.1.5.130/32" --advertise-mode ARP --service-selector "{}"
{
  "addresses": [
    "10.1.5.130/32"
  ],
  "advertiseMode": "ARP",
  "id": "/subscriptions/6b727810-e942-4f1a-92b7-f54a21d51ce6/resourceGroups/rg-labhci-aks-sandbox-sc/providers/Microsoft.Kubernetes/connectedClusters/akswelab01/providers/Microsoft.KubernetesRuntime/loadBalancers/lb-ingress",
  "name": "lb-ingress",
  "provisioningState": "Succeeded",
  "resourceGroup": "rg-labhci-aks-sandbox-sc",
  "serviceSelector": {},
  "systemData": {
    "createdAt": "2024-12-11T10:12:18.5073861Z",
    "createdBy": "RRODRIGUEZ5@gcelsa.com",
    "createdByType": "User",
    "lastModifiedAt": "2024-12-11T10:12:18.5073861Z",
    "lastModifiedBy": "RRODRIGUEZ5@gcelsa.com",
    "lastModifiedByType": "User"
  },
  "type": "microsoft.kubernetesruntime/loadbalancers"
}

C:\Users\RRODRIGUEZ5\aks\aks-hci>
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

# Upgrade AKS a 1.29.4

```bash
az aksarc upgrade --name akswelab01 --resource-group rg-labhci-aks-sandbox-sc --kubernetes-version 1.29.4 --yes
```


# Instalar monitorizacion
```bash
kubectl create ns monitoring
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install -n monitoring monitoring prometheus-community/kube-prometheus-stack -f monitoring.yaml
```

# Instalar kubernetes dashboard

```bash
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard
kubectl -n kubernetes-dashboard apply -f kubernetes-dashboard/ingress.yml
```

obtener token

````bash
kubectl create serviceaccount k8sadmin -n kube-system
kubectl create clusterrolebinding k8sadmin --clusterrole=cluster-admin --serviceaccount=kube-system:k8sadmin
kubectl -n kube-system create token k8sadmin
eyJhbGciOiJSUzI1NiIsImtpZCI6InZDeE1LbzFUcU5IdDNjTzJTbThLLWtTa1NlTGNUSVc1cVZSUWtRNFVvTEUifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzM0MzY5NDc1LCJpYXQiOjE3MzQzNjU4NzUsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJrOHNhZG1pbiIsInVpZCI6ImQwYjc2MDg4LTE4MWEtNGRmNS1iZTNiLWE2YTNlODVhZmQyZCJ9fSwibmJmIjoxNzM0MzY1ODc1LCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06azhzYWRtaW4ifQ.hlNjNY9CMmjxxFktCuwREJ_RrOXLvMXlhJX-6ezi-nWLbKBXj0LcsAZvgWjNawGHAEhtOQi0UzMh_f5XddSaXOP6GRklmaFqva60e82IHHqJp8qrgWQBIBfjc7-4VtwpB2LTxTENexkvm96hVb8YEMhOmwKr0yPEk49d4c1NRDHLc6a0F69qk893T1COb-Bi7Ms-8MQk-LRdd0KW1zG9fBl6jBWpw29CQlOArVXb0sj_GEHsHJMh8ygILZ89VT6XDcNOTDTnzO1hp72Qzvpx8SdR0_iC44K6rB_RB22IQ0snpy9QNx8f2I8XqRUDwq37gbg-H5a8UpjeAatDDEeA2g
```

# URLs

```bash
$ kubectl get ingress -A
NAMESPACE              NAME                                      CLASS   HOSTS                            ADDRESS      PORTS     AGE
kubernetes-dashboard   svc-dashboard                             nginx   dashboard.10.1.5.130.nip.io      10.1.5.130   80        50m
monitoring             monitoring-grafana                        nginx   grafana.10.1.5.130.nip.io        10.1.5.130   80, 443   3d5h
monitoring             monitoring-kube-prometheus-alertmanager   nginx   alertmanager.10.1.5.130.nip.io   10.1.5.130   80, 443   3d5h
monitoring             monitoring-kube-prometheus-prometheus     nginx   prometheus.10.1.5.130.nip.io     10.1.5.130   80, 443   3d5h
test-ns                ingress-nginx                             nginx   test-aks.10.1.5.130.nip.io       10.1.5.130   80        5d6h
```
