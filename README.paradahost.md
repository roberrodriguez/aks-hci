# Se tira el nodo

```bash
$ kubectl -n test-ns get node -o wide -w
NAME              STATUS     ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE            KERNEL-VERSION     CONTAINER-RUNTIME
moc-l66br5vnaz0   Ready      control-plane   45h   v1.29.4   10.1.5.32     <none>        CBL-Mariner/Linux   5.15.160.1-1.cm2   containerd://1.6.26
moc-ld53p25a1ty   Ready      control-plane   45h   v1.29.4   10.1.5.37     <none>        CBL-Mariner/Linux   5.15.160.1-1.cm2   containerd://1.6.26
moc-lg7fzixzebk   Ready      <none>          44h   v1.29.4   10.1.5.34     <none>        CBL-Mariner/Linux   5.15.160.1-1.cm2   containerd://1.6.26
moc-lojr2nz70jg   Ready      control-plane   45h   v1.29.4   10.1.5.33     <none>        CBL-Mariner/Linux   5.15.160.1-1.cm2   containerd://1.6.26
moc-lzmnl1ev0lo   NotReady   <none>          44h   v1.29.4   10.1.5.35     <none>        CBL-Mariner/Linux   5.15.160.1-1.cm2   containerd://1.6.26
```

# No reinicia los pods porque parece que espera si el nodo recupera

```bash
$ kubectl -n test-ns get pod -o wide
NAME                            READY   STATUS    RESTARTS   AGE   IP               NODE              NOMINATED NODE   READINESS GATES
nginx-deploy-7c5ddbdf54-9bf54   1/1     Running   0          44h   10.244.16.4      moc-lzmnl1ev0lo   <none>           <none>
nginx-deploy-7c5ddbdf54-jvz69   1/1     Running   0          61s   10.244.187.158   moc-lg7fzixzebk   <none>           <none>
```

# Hacemos cordon del otro worker y borramos el pod

```bash
$ kubectl -n test-ns get node -o wide
NAME              STATUS                     ROLES           AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE            KERNEL-VERSION     CONTAINER-RUNTIME
moc-l66br5vnaz0   Ready                      control-plane   45h   v1.29.4   10.1.5.32     <none>        CBL-Mariner/Linux   5.15.160.1-1.cm2   containerd://1.6.26
moc-ld53p25a1ty   Ready                      control-plane   45h   v1.29.4   10.1.5.37     <none>        CBL-Mariner/Linux   5.15.160.1-1.cm2   containerd://1.6.26
moc-lg7fzixzebk   Ready,SchedulingDisabled   <none>          44h   v1.29.4   10.1.5.34     <none>        CBL-Mariner/Linux   5.15.160.1-1.cm2   containerd://1.6.26
moc-lojr2nz70jg   Ready                      control-plane   44h   v1.29.4   10.1.5.33     <none>        CBL-Mariner/Linux   5.15.160.1-1.cm2   containerd://1.6.26
moc-lzmnl1ev0lo   NotReady                   <none>          44h   v1.29.4   10.1.5.35     <none>        CBL-Mariner/Linux   5.15.160.1-1.cm2   containerd://1.6.26
```

# El otro pod se queda en pending porque no hay nodo ready

```bash
$ kubectl -n test-ns get pod -o wide
NAME                            READY   STATUS    RESTARTS   AGE   IP            NODE              NOMINATED NODE   READINESS GATES
nginx-deploy-7c5ddbdf54-9bf54   1/1     Running   0          44h   10.244.16.4   moc-lzmnl1ev0lo   <none>           <none>
nginx-deploy-7c5ddbdf54-vdv66   0/1     Pending   0          18s   <none>        <none>            <none>           <none>
```

# Al cabo de un rato mata el pod del nodo not ready

```bash
$ kubectl -n test-ns get pod -o wide
NAME                            READY   STATUS        RESTARTS   AGE    IP            NODE              NOMINATED NODE   READINESS GATES
nginx-deploy-7c5ddbdf54-9bf54   1/1     Terminating   0          44h    10.244.16.4   moc-lzmnl1ev0lo   <none>           <none>
nginx-deploy-7c5ddbdf54-vdv66   0/1     Pending       0          2m7s   <none>        <none>            <none>           <none>
nginx-deploy-7c5ddbdf54-zb7t9   0/1     Pending       0          61s    <none>        <none>            <none>           <none>
```

# Quitamos el cordon y levantan ambos pods en el otro nodo

```bash
$ kubectl -n test-ns get pod -o wide
NAME                            READY   STATUS    RESTARTS   AGE     IP               NODE              NOMINATED NODE   READINESS GATES
nginx-deploy-7c5ddbdf54-vdv66   1/1     Running   0          7m13s   10.244.187.161   moc-lg7fzixzebk   <none>           <none>
nginx-deploy-7c5ddbdf54-zb7t9   1/1     Running   0          6m7s    10.244.187.159   moc-lg7fzixzebk   <none>           <none>
```

