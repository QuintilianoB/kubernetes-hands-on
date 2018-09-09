# Configurandos os nodes

Os nodes são compostos dos seguintes serviços:

* Containerd
* Kube-proxy
* Kubelet 

Os passos aqui devem ser realizados nos dois nodes! Execute como root.

#### Instalando

1 - Instalando pacotes adicionais.

São pacotes que permitem a utilização do comando kubectl port-foward para expor
um serviço do cluster.

```bash
yum install socat conntrack ipset ipvsadm -ymodprobe ip_vs
modprobe ip_vs_rr
modprobe ip_vs_wrr
modprobe ip_vs_sh
modprobe nf_conntrack_ipv4
modprobe br_netfilter
echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
```

2 - Baixando os arquivos

```bash
wget https://github.com/opencontainers/runc/releases/download/v1.0.0-rc5/runc.amd64 \
  https://github.com/containernetworking/plugins/releases/download/v0.6.0/cni-plugins-amd64-v0.6.0.tgz \
  https://github.com/containerd/containerd/releases/download/v1.1.0/containerd-1.1.0.linux-amd64.tar.gz \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubectl \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kube-proxy \
  https://storage.googleapis.com/kubernetes-release/release/v1.10.2/bin/linux/amd64/kubelet
```

3 - Criando os diretório:

```bash
mkdir -p /etc/containerd /etc/cni/net.d /opt/cni/bin /var/lib/kubelet /var/lib/kube-proxy /var/lib/kubernetes /var/run/kubernetes
```

4 - Movendo os binários

```bash
chmod +x kubectl kube-proxy kubelet runc.amd64
mv runc.amd64 runc
mv kubectl kube-proxy kubelet runc /usr/local/bin/
tar -xvf cni-plugins-amd64-v0.6.0.tgz -C /opt/cni/bin/
tar -xvkf containerd-1.1.0.linux-amd64.tar.gz -C /
 ```
 
 #### Containerd
 
 Containerd é a solução escolhida para executar os containers. 
 Ele trabalha com o conceito de plugins para configurar e executar os containers, 
 o sistema de arquivos e a rede.
 Substitui o docker. 
 
 1 - Configurando o Containerd
 
Crie o arquivo com o **/etc/containerd/config.toml** com o conteúdo:  
 
 ```bash 
[plugins]
  [plugins.cri.containerd]
    snapshotter = "native"
    [plugins.cri.containerd.default_runtime]
      runtime_type = "io.containerd.runtime.v1.linux"
      runtime_engine = "/usr/local/bin/runc"
      runtime_root = ""
```

E também crie o arquivo **/etc/cni/net.d/99-loopback.conf** com o conteúdo:

```bash
{
    "cniVersion": "0.3.1",
    "type": "loopback"
}
```

2 - Configure o serviço: 

Crie o arquivo **/etc/systemd/system/containerd.service** com o conteúdo:


```bash
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target

[Service]
ExecStartPre=/sbin/modprobe overlay
ExecStart=/bin/containerd
Restart=always
RestartSec=5
Delegate=yes
KillMode=process
OOMScoreAdjust=-999
LimitNOFILE=1048576
LimitNPROC=infinity
LimitCORE=infinity

[Install]
WantedBy=multi-user.target
```

3 - Inicie o serviço

```bash
systemctl daemon-reload
systemctl enable containerd
systemctl start containerd
```

#### Kube-proxy

1 - Kube-proxy.kubeconfig

Enquanto configuravamos o master, criamos o arquivo **kube-proxy.kubeconfig**.
Salve uma cópia em **/var/lib/kube-proxy/kubeconfig**

2 - Arquivo de configuração

Assim como o kube-scheduler, as configurações do kube-proxy estão sendo migradas para
um arquivo yaml, permitindo a configuração dinâmica do mesmo.

Crei o arquivo **/var/lib/kube-proxy/kube-proxy-config.yaml** com o conteúdo:

```bash
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
clientConnection:
  kubeconfig: "/var/lib/kube-proxy/kubeconfig"
mode: "ipvs"
clusterCIDR: "10.200.0.0/16"
```

3 - Criando o serviço.

Crie o arquivo **/etc/systemd/system/kube-proxy.service** com o conteúdo:

```
[Unit]
Description=Kubernetes Kube Proxy
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-proxy \
  --config=/var/lib/kube-proxy/kube-proxy-config.yaml
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

6 - Inicie o serviço:

```bash
systemctl daemon-reload
systemctl enable kube-proxy
systemctl start kube-proxy
```

#### Kubelet

1 - Arquivos de configuração

Copie os seguintes arquivos para a pasta **/var/lib/kubelet/**

* ca.pem
* bootstrap.kubeconfig
* kubelet-tls.pem
* kubelet-tls-key.pem

Crei o arquivo **/var/lib/kubelet/kubelet-config.yaml** com o conteúdo:

```bash
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    enabled: true
  x509:
    clientCAFile: "/var/lib/kubelet/ca.pem"
authorization:
  mode: Webhook
clusterDomain: "hands-on"
clusterDNS:
  - "10.32.0.10"
runtimeRequestTimeout: "15m"
rotateCertificates: true
rotateKubeletClientCertificate: true
tlsCertFile: "/var/lib/kubelet/kubelet-tls.pem"
tlsPrivateKeyFile: "/var/lib/kubelet/kubelet-tls-key.pem"

``` 

2 - Crie o serviço:

Crie o arquivo **/etc/systemd/system/kubelet.service** com o conteúdo:

```bash
[Unit]
Description=Kubernetes Kubelet
Documentation=https://github.com/kubernetes/kubernetes
After=containerd.service
Requires=containerd.service

[Service]
ExecStart=/usr/local/bin/kubelet \
  --config=/var/lib/kubelet/kubelet-config.yaml \
  --cert-dir=/var/lib/kubelet \
  --container-runtime=remote \
  --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock \
  --image-pull-progress-deadline=2m \
  --kubeconfig=/var/lib/kubelet/kubeconfig \
  --network-plugin=cni \
  --register-node=true \
  --bootstrap-kubeconfig=/var/lib/kubelet/bootstrap.kubeconfig \
  --fail-swap-on=false \
  --allow-privileged=true \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

3 - Inicie o serviço:

```bash
systemctl daemon-reload
systemctl enable kubelet
systemctl start kubelet
```

#### Aprovando os novos nodes.

Após a configuração dos nodes, falta agora aprovar a entrada deles no cluster.
Deixamos a criação dos certificados de cada node para o cluster. Resta agora aprová-los.

De volta ao master, execute:

```bash
kubectl get csr
```

Será listado os certificados pendentes de aprovação:

```bash
NAME                                                   AGE       REQUESTOR                 CONDITION
node-csr-i0QKqoEMMxT-aiSv_WVGQm8rtjw-cN7oimXhYZWTHAY   16s       system:bootstrap:57f123   Pending
node-csr-odZfFv_kEfDhlBb6SKLtOOSFS9bZOzyhDTkow4r-MZk   17s       system:bootstrap:57f123   Pending
```

Basta aprová-los:
```bash
kubectl certificate approve < NOME >
```

E listar os nodes disponíveis no cluster
```bash
[root@master ]# kubectl get nodes -o wide
NAME          STATUS     ROLES     AGE       VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION               CONTAINER-RUNTIME
node1.local   NotReady   <none>    11s       v1.10.2   10.100.100.5   <none>        CentOS Linux 7 (Core)   3.10.0-862.11.6.el7.x86_64   containerd://1.2.0-beta.2
node2.local   NotReady   <none>    11s       v1.10.2   10.100.100.6   <none>        CentOS Linux 7 (Core)   3.10.0-862.11.6.el7.x86_64   containerd://1.2.0-beta.2
```

O cluster está pronto para utilização!

Próximo: [Configurando serviços básicos](servicos.md)