#### [Componentes](https://kubernetes.io/docs/concepts/overview/components/)

##### Detalhes do cluster
- [Kubernetes](https://github.com/kubernetes/kubernetes/tree/release-1.11) versão 1.11
- [ContainerD](https://containerd.io/) versão 1.0.1
- [ETCD](https://github.com/etcd-io/etcd/tree/v3.3.5) versão 3.3.5


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
