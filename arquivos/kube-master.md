# Configurando o Kube Master.

Para ter o kube-master funcionando, precisamos dos seguintes componentes configurados:
- ETCD
- Kube-apiserver
- Kubeconigs
- Kube-controller-manager
- Kube-scheduler

Acesse o servidor master e execute cada passo em sequência. Execute como root!

#### ETCD

O [ETCD](https://coreos.com/etcd/) é uma solução de banco de dados distribuído, utilizado
pelo cluster Kubernetes para armazenamento do estado do cluster. Em um ambiente de
produção normalmente são instalados entre 3 e 5 instâncias, de acordo com o tamanho do cluster.

1 - Download:
```bash
wget https://github.com/coreos/etcd/releases/download/v3.3.5/etcd-v3.3.5-linux-amd64.tar.gz
```

2 - Extraindo e movendo o binário:
```bash
tar -xvf etcd-v3.3.5-linux-amd64.tar.gz
mv etcd-v3.3.5-linux-amd64/etcd* /usr/local/bin/

```

3 - Criando os diretórios
```bash
mkdir -p /etc/etcd /var/lib/etcd
```

4 - Copiando os certificados:

A conexão com o ETCD é realizada de forma segura (TLS). Utilizamos os mesmos certificados
criados para o cluster no banco de dados.

Copie os arquivos ca.pem kubernetes-key.pem e kubernetes.pem para a pasta /etc/etcd/

5 - Criando o serviço do ETCD.

**Atenção!** Caso esteja configurando um cluster ETCD, é necessário identificar cada
instância do banco individualmente.

Crei o arquivo **/etc/systemd/system/etcd.service** com o conteúdo abaixo, substituindo
a flag <IP MASTER> pelo IP do Master:

```
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
ExecStart=/usr/local/bin/etcd \
  --name master \
  --cert-file=/etc/etcd/kubernetes.pem \
  --key-file=/etc/etcd/kubernetes-key.pem \
  --peer-cert-file=/etc/etcd/kubernetes.pem \
  --peer-key-file=/etc/etcd/kubernetes-key.pem \
  --trusted-ca-file=/etc/etcd/ca.pem \
  --peer-trusted-ca-file=/etc/etcd/ca.pem \
  --peer-client-cert-auth \
  --client-cert-auth \
  --initial-advertise-peer-urls https://<IP MASTER>:2380 \
  --listen-peer-urls https://<IP MASTER>:2380 \
  --listen-client-urls https://<IP MASTER>:2379,https://127.0.0.1:2379 \
  --advertise-client-urls https://<IP MASTER>:2379 \
  --initial-cluster-token etcd-cluster-0 \
  --initial-cluster master=https://<IP MASTER>:2380 \
  --initial-cluster-state new \
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

6 - Inicie o serviço:

```bash
systemctl daemon-reload
systemctl enable etcd
systemctl start etcd

```

7 - Teste!

```bash
ETCDCTL_API=3 /usr/local/bin/etcdctl member list --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem --cert=/etc/etcd/kubernetes.pem --key=/etc/etcd/kubernetes-key.pem
```

A saída esperada:
```bash
2f46c6deb98945ec, started, master, https://10.100.100.4:2380, https://10.100.100.4:2379
```

#### Kube-apiserver

O API Server centralizada todo o controle e estado do cluster. Por ele passam todas
as requisições de criação, alteração e verificação de estado.

1 - Criação dos diretórios:
```bash
mkdir -p /etc/kubernetes/config
mkdir -p /var/lib/kubernetes
```

2 - Instalação dos binários:
```bash
cd /opt/

wget "https://storage.googleapis.com/kubernetes-release/release/v1.11.2/bin/linux/amd64/kube-apiserver"\
 "https://storage.googleapis.com/kubernetes-release/release/v1.11.2/bin/linux/amd64/kube-controller-manager"\
 "https://storage.googleapis.com/kubernetes-release/release/v1.11.2/bin/linux/amd64/kube-scheduler"\
 "https://storage.googleapis.com/kubernetes-release/release/v1.11.2/bin/linux/amd64/kubectl"

chmod +x kube-apiserver kube-controller-manager kube-scheduler kubectl

mv kube* /usr/local/bin/

export PATH=$PATH:/usr/local/bin

echo "export PATH=$PATH:/usr/local/bin" >> /root/.bashrc
```

3 - Copiando os certificados:

Copie os seguintes arquivos para a pasta **/var/lib/kubernetes/**:

* ca.pem 
* ca-key.pem 
* kubernetes-key.pem 
* kubernetes.pem
* service-account-key.pem 
* service-account.pem

4 - Gerando uma chave de criptografia para os *secrets* armazenados no cluster:

```bash
CHAVE=$(head -c 32 /dev/urandom | base64)
cat > /var/lib/kubernetes/encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${CHAVE}
      - identity: {}
EOF
```

5 - Criando o serviço para o apiserver:

Crei o arquivo **/etc/systemd/system/kube-apiserver.service** com o conteúdo abaixo, substituindo
a flag <IP MASTER> pelo IP do Master:

```bash
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-apiserver \
  --advertise-address=<IP MASTER> \
  --allow-privileged=true \
  --apiserver-count=1 \
  --audit-log-maxage=30 \
  --audit-log-maxbackup=3 \
  --audit-log-maxsize=100 \
  --audit-log-path=/var/log/audit.log \
  --authorization-mode=Node,RBAC \
  --bind-address=0.0.0.0 \
  --client-ca-file=/var/lib/kubernetes/ca.pem \
  --enable-admission-plugins=Initializers,NamespaceLifecycle,NodeRestriction,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota \
  --enable-swagger-ui=false \
  --etcd-cafile=/var/lib/kubernetes/ca.pem \
  --etcd-certfile=/var/lib/kubernetes/kubernetes.pem \
  --etcd-keyfile=/var/lib/kubernetes/kubernetes-key.pem \
  --etcd-servers=https://<IP MASTER>:2379 \
  --event-ttl=1h \
  --experimental-encryption-provider-config=/var/lib/kubernetes/encryption-config.yaml \
  --kubelet-certificate-authority=/var/lib/kubernetes/ca.pem \
  --kubelet-client-certificate=/var/lib/kubernetes/kubernetes.pem \
  --kubelet-client-key=/var/lib/kubernetes/kubernetes-key.pem \
  --kubelet-https=true \
  --runtime-config=api/all \
  --service-account-key-file=/var/lib/kubernetes/service-account.pem \
  --service-cluster-ip-range=10.32.0.0/24 \
  --service-node-port-range=30000-32767 \
  --tls-cert-file=/var/lib/kubernetes/kubernetes.pem \
  --tls-private-key-file=/var/lib/kubernetes/kubernetes-key.pem \
  --enable-bootstrap-token-auth=true \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

6 - Inicie o serviço:

```bash
systemctl daemon-reload
systemctl enable kube-apiserver
systemctl start kube-apiserver
```

7 - Verifique o estado do serviço:

```bash
systemctl status kube-apiserver
```

8 - Verifique o estado do cluster
```bash
kubectl get componentstatus
```

Note que apenas o ETCD está ok mas o api server já responde!

```bash
NAME                 STATUS      MESSAGE                                                                                     ERROR
scheduler            Unhealthy   Get http://127.0.0.1:10251/healthz: dial tcp 127.0.0.1:10251: connect: connection refused   
controller-manager   Unhealthy   Get http://127.0.0.1:10252/healthz: dial tcp 127.0.0.1:10252: connect: connection refused   
etcd-0               Healthy     {"health":"true"}  
```

9 - Configurar permissões

Por último, para permitir que o apiserver converse com os nodes, é necessário criar
a permissão para tal. Abaixo, segue a regra de criação do clusterole que permite
a conexão do API-Server à API Kubelet existente nos nodes.

```bash
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
  labels:
    kubernetes.io/bootstrapping: rbac-defaults
  name: system:kube-apiserver-to-kubelet
rules:
  - apiGroups:
      - ""
    resources:
      - nodes/proxy
      - nodes/stats
      - nodes/log
      - nodes/spec
      - nodes/metrics
    verbs:
      - "*"
EOF
```

#### Kubeconfigs

Os componentes do cluster utilizam arquivos de formato especial, chamados de 
[kubeconfig](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/).
Após a instalação do kubectl e do apiserver, vamos agora gerar os arquivos necessários
para os outros componentes:

Execute os comandos a seguir na mesma pasta em que criou os certificados.

1 - Kubeconfig para o controller-manager:

```bash
kubectl config set-cluster hands-on --certificate-authority=ca.pem --embed-certs=true \
    --server=https://127.0.0.1:6443 --kubeconfig=kube-controller-manager.kubeconfig

kubectl config set-credentials system:kube-controller-manager \
    --client-certificate=kube-controller-manager.pem \
    --client-key=kube-controller-manager-key.pem --embed-certs=true \
    --kubeconfig=kube-controller-manager.kubeconfig

kubectl config set-context default --cluster=hands-on \
    --user=system:kube-controller-manager --kubeconfig=kube-controller-manager.kubeconfig

kubectl config use-context default --kubeconfig=kube-controller-manager.kubeconfig
```

2 - Kubeconfig para o scheduler:

```bash
kubectl config set-cluster hands-on --certificate-authority=ca.pem \
    --embed-certs=true --server=https://127.0.0.1:6443 --kubeconfig=kube-scheduler.kubeconfig

kubectl config set-credentials system:kube-scheduler \
    --client-certificate=kube-scheduler.pem --client-key=kube-scheduler-key.pem \
    --embed-certs=true --kubeconfig=kube-scheduler.kubeconfig

kubectl config set-context default --cluster=hands-on --user=system:kube-scheduler \
    --kubeconfig=kube-scheduler.kubeconfig

kubectl config use-context default --kubeconfig=kube-scheduler.kubeconfig
```

3 - Kubeconfig para o administrador do cluster:
```bash
kubectl config set-cluster hands-on --certificate-authority=ca.pem \
    --embed-certs=true --server=https://127.0.0.1:6443 --kubeconfig=admin.kubeconfig

kubectl config set-credentials admin --client-certificate=admin.pem \
    --client-key=admin-key.pem --embed-certs=true --kubeconfig=admin.kubeconfig

kubectl config set-context default --cluster=hands-on --user=admin \
    --kubeconfig=admin.kubeconfig

kubectl config use-context default --kubeconfig=admin.kubeconfig
```

4 - Configuração para geração de certificados dos nodes de forma automática

**Altere a flag < MASTER IP > para o endereço do seu master.**

```bash
TOKEN_PUB=$(openssl rand -hex 3)
TOKEN_SECRET=$(openssl rand -hex 8)
BOOTSTRAP_TOKEN="${TOKEN_PUB}.${TOKEN_SECRET}"

kubectl config set-cluster hands-on --certificate-authority=ca.pem --embed-certs=true \
    --server=https://< MASTER IP >:6443 --kubeconfig=bootstrap.kubeconfig

kubectl config set-context kubelet-bootstrap@hands-on --cluster=hands-on --user=kubelet-bootstrap \
    --kubeconfig=bootstrap.kubeconfig

kubectl config use-context kubelet-bootstrap@hands-on --kubeconfig=bootstrap.kubeconfig

kubectl config set-credentials kubelet-bootstrap --token=${BOOTSTRAP_TOKEN} --kubeconfig=bootstrap.kubeconfig

kubectl -n kube-system create secret generic bootstrap-token-${TOKEN_PUB} --type 'bootstrap.kubernetes.io/token' \
        --from-literal description="cluster bootstrap token" --from-literal token-id=${TOKEN_PUB} \
        --from-literal token-secret=${TOKEN_SECRET} --from-literal usage-bootstrap-authentication=true \
        --from-literal usage-bootstrap-signing=true
        
kubectl config set-cluster hands-on --certificate-authority=ca.pem \
    --embed-certs=true --server=https://< MASTER IP >:6443 \
    --kubeconfig=bootstrap.kubeconfig

kubectl config set-context kubelet-bootstrap@hands-on --cluster=hands-on \
    --user=kubelet-bootstrap --kubeconfig=bootstrap.kubeconfig

kubectl config use-context kubelet-bootstrap@hands-on --kubeconfig=bootstrap.kubeconfig

kubectl create clusterrolebinding kubelet-bootstrap --clusterrole system:node-bootstrapper \
    --group system:bootstrappers       
```

5 - Kubeconfig para o kube-proxy

**Altere a flag < MASTER IP > para o endereço do seu master.**

```bash
kubectl config set-cluster hands-on --certificate-authority=ca.pem --embed-certs=true \
    --server=https://< MASTER IP >:6443 --kubeconfig=kube-proxy.kubeconfig

kubectl config set-credentials system:kube-proxy --client-certificate=kube-proxy.pem \
    --client-key=kube-proxy-key.pem --embed-certs=true --kubeconfig=kube-proxy.kubeconfig

kubectl config set-context default --cluster=hands-on --user=system:kube-proxy \
    --kubeconfig=kube-proxy.kubeconfig

kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig
```

#### Controller Manager

O controller manager é responsável por manter o estado do cluster à partir do que
está definido no API Server.

1 - Criando o serviço:

Crei o arquivo **/etc/systemd/system/kube-controller-manager.service** com o conteúdo abaixo.

```bash
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-controller-manager \
  --address=0.0.0.0 \
  --cluster-cidr=10.200.0.0/16 \
  --allocate-node-cidrs=true \
  --cluster-name=hands-on \
  --cluster-signing-cert-file=/var/lib/kubernetes/ca.pem \
  --cluster-signing-key-file=/var/lib/kubernetes/ca-key.pem \
  --kubeconfig=/var/lib/kubernetes/kube-controller-manager.kubeconfig \
  --leader-elect=true \
  --root-ca-file=/var/lib/kubernetes/ca.pem \
  --service-account-private-key-file=/var/lib/kubernetes/service-account-key.pem \
  --service-cluster-ip-range=10.32.0.0/24 \
  --use-service-account-credentials=true \
  --controllers=*,tokencleaner \
  --feature-gates=RotateKubeletServerCertificate=true \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

2 - O arquivo de configuração:

Copie o arquivo kube-controller-manager.kubeconfig para **/var/lib/kubernetes/**

3 - Inicie o serviço:

```bash
systemctl daemon-reload
systemctl enable kube-controller-manager
systemctl start kube-controller-manager
```

4 - Verifique o estado do serviço:

```bash
systemctl status kube-controller-manager
```

5 - Verifique o estado do cluster
```bash
kubectl get componentstatus
```

Note que o controllermanager agora está ativo para o cluster.

```bash
NAME                 STATUS      MESSAGE                                                                                     ERROR
scheduler            Unhealthy   Get http://127.0.0.1:10251/healthz: dial tcp 127.0.0.1:10251: connect: connection refused   
controller-manager   Healthy     ok                                                                                          
etcd-0               Healthy     {"health":"true"}  
```

#### Scheduler

O scheduler é o responsável por definir quando e onde os pods serão criados no cluster.
Ele verifica uma sequencia de regras definidas pelo administrador para decidir onde um
pod específico deve ser executado. Entre as regras verificadas estão:

* Quantidade de recursos necessário
* Afinidade ou não entre pods
* Labels

1 - Copiando o arquivo de configuração.

Copie o arquivo kube-scheduler.kubeconfig para **/var/lib/kubernetes/**

2 - Crie o arquivo de configuração dinâmico:

Á partir da versão 1.10 a configuração de alguns componentes estão sendo migradas
para arquivos do tipo yaml, permitindo a configuração dinâmica do cluster.
Está no road map do kubernetes a remoção das configurações por flags passadas aos binários.

Crie o arquivo /etc/kubernetes/config/kube-scheduler.yaml

```bash\
#
---
apiVersion: componentconfig/v1alpha1
kind: KubeSchedulerConfiguration
clientConnection:
  kubeconfig: "/var/lib/kubernetes/kube-scheduler.kubeconfig"
leaderElection:
  leaderElect: true
```

3 - Criando o serviço.

rei o arquivo **/etc/systemd/system/kube-scheduler.service** com o conteúdo abaixo.

```bash
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes

[Service]
ExecStart=/usr/local/bin/kube-scheduler \
  --config=/etc/kubernetes/config/kube-scheduler.yaml \
  --v=2
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

4 - Inicie o serviço:

```bash
systemctl daemon-reload
systemctl enable kube-scheduler
systemctl start kube-scheduler
```

4 - Verifique o estado do serviço:

```bash
systemctl status kube-scheduler
```

5 - Verifique o estado do cluster
```bash
kubectl get componentstatus
```

Agora todos os componentes do master devem estar OK.

```bash
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok                  
scheduler            Healthy   ok                  
etcd-0               Healthy   {"health":"true"} 
```

Próximo: [Configurando os nodes](nodes.md)