# Configurandos serviços básicos

Com o cluster ativo, vamos agora configurar os seguintes serviços:

* Rede
* DNS

#### Rede

Precisamos configurar um [plugin de rede](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
para permitir a comunicação entre os containers localizados em diferentes nodes.

Entre as opções disponíveis temos o [Flannel](https://github.com/coreos/flannel), 
[Calico](https://www.projectcalico.org/) e o [Canal](https://github.com/projectcalico/canal).
Cada uma tem suas vantagens e desvantagens, cabendo ao administrador escolher o que melhor 
atende sua necessidade.

Aqui vamos utilizar o Flannel devido á sua simipliciadade.

1 - Configurando o Flannel

No master, execute:

```bash
kubectl apply -f https://raw.githubusercontent.com/QuintilianoB/kubernetes-hands-on/master/arquivos/servicos/flannel.yaml
``` 

Deve ser criado um pod por node existente. Verifique o estado de cada pod com:

```bash
kubecetl get pods -n kube-system
```

Com a saída esperada:

```bash
NAME                          READY     STATUS    RESTARTS   AGE
kube-flannel-ds-amd64-b5mxl   1/1       Running   0          45m
kube-flannel-ds-amd64-mmkzh   1/1       Running   0          45m
```

#### DNS

À partir da versão 1.10, o [CoreDNS](https://coredns.io/) passou a ser a solução oficial de DNS. De forma 
similar à instalação do Flannel, execute:

```bash
kubectl apply -f kubectl apply -f https://raw.githubusercontent.com/QuintilianoB/kubernetes-hands-on/master/arquivos/servicos/coredns.yaml
```

Verifique os pods criados:

```bash
kubecetl get pods -n kube-system
```

Com a saída esperada:

```bash
NAME                          READY     STATUS    RESTARTS   AGE
coredns-55f86bf584-n6qk8      1/1       Running   0          37s
coredns-55f86bf584-x8k79      1/1       Running   0          37s
kube-flannel-ds-amd64-b5mxl   1/1       Running   0          53m
kube-flannel-ds-amd64-mmkzh   1/1       Running   0          53m
```

#### Testar!

Vamos agora criar 2 containers para testar a rede e o DNS.

1 - Crie dois deployments para testarmos o cluster:

```bash
kubectl run busybox1 --image=busybox -- sleep 1000
kubectl run busybox2 --image=busybox -- sleep 1000
```

2 - Liste os containers criados:

```
kubectl get pods 

NAME                        READY     STATUS    RESTARTS   AGE
busybox1-6d4d599cf9-xptmm   1/1       Running   0          49s
busybox2-77dd7d7579-fvdvx   1/1       Running   0          12m
```

3 - Acesse o shell de cada um e teste conectividade e resolução de DNS.

Substitua a flag < NOME DO POD > por um dos nomes obtidos com o comando anterior.

```bash
kubectl exec -it < NOME DO POD > sh
```

Você deve conseguir:

* Pingar o google.com
* Pingar o outro container. Descubra o IP acessando o shell!
* Pingar o serviço principal: kubernetes

Caso algum desses testes falhe, verifique suas configurações.

Próximo: [Expondo um serviço](publicar.md) 