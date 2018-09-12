# Expondo um serviço.

O último passo desse laboratório será a criação e publicação de um serviço para 
acesso externo. No caso, criaremos um pod com Ngnix e vamos expó-lo através de NAT
em nossos nodes.

1 - Crie o deploy e serviço para o Nginx.

```bash
kubectl apply -f https://raw.githubusercontent.com/QuintilianoB/kubernetes-hands-on/master/arquivos/servicos/deploy_nginx.yaml
```

2 - Verifique o que foi criado:

```bash
kubectl get pods

NAME                                READY     STATUS    RESTARTS   AGE
busybox1-6d4d599cf9-xptmm           1/1       Running   2          53m
busybox2-77dd7d7579-fvdvx           1/1       Running   3          1h
nginx-deployment-67594d6bf6-4v4bq   1/1       Running   0          10m


kubectl get services

NAME            TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
kubernetes      ClusterIP   10.32.0.1     <none>        443/TCP        1d
nginx-service   NodePort    10.32.0.95    <none>        80:30298/TCP   1m
```

Após a criação do container, uma porta será aberta em cada node, redirecionando o tráfego
a ela para o serviço do nginx.

Á partir do master, testando a conexão ao NGinx:

Node 1 - 10.100.100.5

```bash
curl http://10.100.100.5:30298

<html>
<head><title>403 Forbidden</title></head>
<body bgcolor="white">
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.7.9</center>
</body>
</html>
```

Node 2 - 10.100.100.7

```bash
curl http://10.100.100.7:30298

<html>
<head><title>403 Forbidden</title></head>
<body bgcolor="white">
<center><h1>403 Forbidden</h1></center>
<hr><center>nginx/1.7.9</center>
</body>
</html>
```

