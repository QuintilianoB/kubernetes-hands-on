#### [Componentes](https://kubernetes.io/docs/concepts/overview/components/)

##### Detalhes do cluster
- [Centos 7 x86_64](http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1804.iso)
- [Kubernetes 1.11.2](https://github.com/kubernetes/kubernetes/releases/tag/v1.11.2)
- [ContainerD 1.0.1](https://github.com/containerd/containerd/releases/tag/v1.1.3)
- [Etcd 3.3.5](https://github.com/etcd-io/etcd/tree/v3.3.5)


1 - Master
- Servidor responsável pela orquestração do cluster.
  - API Server
    - Um serviço que expõe uma API rest para definir e validar os recursos
    disponíveis no cluster.
  - Controller-manager
    - Um conjunto de controladores responsáveis por manter o cluster no
    estado definido pelo api server.
    - 
  - Scheduler
    - Responsável por definir onde os novos pods devem ser executados
    de acordo com uma série de regras definidas.
  - ETCD
    - Banco de dados utilizado para armazenar o estado do cluster.

2 - Nodes

- Servidores responsáveis pela execução do containers.
  - ContainerD
    - Gerenciador de containers
  - Kubelet
    - Recebe as especificações do pod para execução em um determinado nó
    e garante que ele seja executado de acordo.
  - Kube-proxy
    - Um proxy de rede utilizado para permitir a comunicação entre os serviços e containers 
    do cluster. É o kube-proxy quem distribui a carga os pods que compõe um serviço.

---
Próximo: [Vagrant](vagrant.md)
