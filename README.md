# Kubernetes Hands ON

Goiânia - 14 de setembro de 2018

Tutorial utilizado no hands on de Kubernetes.

#### Pré-requisitos

* Computador
  * 4GB de ram +
  * Se você utiliza o computador no dia-a-dia, seu processador já é
    suficiente.
  * 20GB de disco para as VMs.
* Sistema operacional Windows/Linux compatível com VirtualBox.
  * Esse material foi desenvolvido em Windows mas deve funcionar sem
    maiores problemas em Linux.
* [VirtualBox 5.2.8](https://www.virtualbox.org/wiki/Downloads) ou
  posterior +
  * Não testei em outras soluções. Pode ser necessário ajustar algum
    detalhe.

#### Layout

![network_layout](network_layout.png)


#### [Componentes](https://kubernetes.io/docs/concepts/overview/components/)

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
  - [ContainerD](https://containerd.io/)
    - Gerenciador de containers
  - Kubelet
    - Recebe as especificações do pod para execução em um determinado nó
    e garante que ele seja executado de acordo.
  - Kube-proxy
    - Responsável por manter a configuração de rede entre os nós do cluster.


---

Todo material aqui pode ser baixado e modificado à vontade, desde que
seja respeitado os direitos definidos pelos autores das referências
utilizadas. Não me resposabilizo pela má útlização, defeito, ou qualquer
problema causado pela utilização desse material.

