# Configurando os certificados

Vamos gerar os certificados utilizados no cluster. 

O openssl vem por padrão na maioria das distribuições Linux, inclusive no Centos 7 utilizado
como base das imagens.

Escolha um dos servidores criados, de preferência o **Master**, crie a pasta em
/home/vagrant/certificados e salve o arquivo [crt.cnf](crt.cnf) nessa pasta.

Ao copiar o arquivo crt.cnf, altere os campos pelos respectivos IPs. Ex:

* <IP_MASTER> -> 10.100.100.4
* <IP_NODE_1> -> 10.100.100.5
* <IP_NODE_2> -> 10.100.100.6

#### Criação da AC
```bash
openssl genrsa -out ca-key.pem 4096

openssl req -x509 -new -extensions v3_ca -key ca-key.pem -days 3650 \
    -out ca.pem -subj '/C=BR/O=hands-on/CN=AC Kubernetes/' -config crt.cnf
```

#### Criação do API Server
```bash
openssl genrsa -out kubernetes-key.pem 2048

openssl req -new -key kubernetes-key.pem -newkey rsa:2048 -nodes \
    -config crt.cnf -subj '/C=BR/O=hands-on/CN=kubernetes' -outform pem \
    -out kubernetes-req.pem -keyout kubernetes-req.key

openssl x509 -req -in kubernetes-req.pem -CA ca.pem -CAkey ca-key.pem \
    -CAcreateserial -out kubernetes.pem -days 3650 -extensions kubernetes_v3 \
    -extfile crt.cnf
```

#### Criação do certificado utilizado pelo controller manager para a gerência de serviços.
```bash
openssl genrsa -out service-account-key.pem 2048

openssl req -new -key service-account-key.pem -newkey rsa:2048 -nodes \
    -config crt.cnf -subj '/C=BR/O=hands-on/CN=service-accounts' -outform pem \
    -out service-account-req.pem -keyout service-account-req.key

openssl x509 -req -in service-account-req.pem -CA ca.pem -CAkey ca-key.pem \
    -CAcreateserial -out service-account.pem -days 3650 \
    -extensions service_v3 -extfile crt.cnf
```

#### Certificado para o Controller Manager
```bash
openssl genrsa -out kube-controller-manager-key.pem 2048

openssl req -new -key kube-controller-manager-key.pem -newkey rsa:2048 \
    -nodes -config crt.cnf -subj '/C=BR/O=hands-on/CN=system:kube-controller-manager'\
    -outform pem -out kube-controller-manager-req.pem \
    -keyout kube-controller-manager-req.key

openssl x509 -req -in kube-controller-manager-req.pem -CA ca.pem \
    -CAkey ca-key.pem -CAcreateserial -out kube-controller-manager.pem \
    -days 3650 -extensions service_v3 -extfile crt.cnf
```

#### Certificado para o Scheduler
```bash
openssl genrsa -out kube-scheduler-key.pem 2048

openssl req -new -key kube-scheduler-key.pem -newkey rsa:2048 -nodes \
    -config crt.cnf -subj '/C=BR/O=hands-on/CN=system:kube-scheduler' \
    -outform pem -out kube-scheduler-req.pem -keyout kube-scheduler-req.key

openssl x509 -req -in kube-scheduler-req.pem -CA ca.pem -CAkey ca-key.pem \
    -CAcreateserial -out kube-scheduler.pem -days 3650 -extensions service_v3 \
    -extfile crt.cnf
```

#### Certificado para administração
```bash
openssl genrsa -out admin-key.pem 2048

openssl req -new -key admin-key.pem -newkey rsa:2048 -nodes -config crt.cnf \
    -subj '/C=BR/O=hands-on/CN=admin' -outform pem -out admin-req.pem \
    -keyout admin-req.key

openssl x509 -req -in admin-req.pem -CA ca.pem -CAkey ca-key.pem \
    -CAcreateserial -out admin.pem -days 3650 -extensions service_v3 \
    -extfile crt.cnf
```

#### Certificado para o Kube-proxy
```bash
openssl genrsa -out kube-proxy-key.pem 2048

openssl req -new -key kube-proxy-key.pem -newkey rsa:2048 -nodes -config crt.cnf \
    -subj '/C=BR/O=hands-on/CN=system:kube-proxy' -outform pem \
    -out kube-proxy-req.pem -keyout kube-proxy-req.key

openssl x509 -req -in kube-proxy-req.pem -CA ca.pem -CAkey ca-key.pem \
    -CAcreateserial -out kube-proxy.pem -days 3650 -extensions service_v3 \
    -extfile crt.cnf 
```

Com os certificados gerados, vamos iniciar a configuração do master.

Próximo: [Configurando o Kube Master](kube-master.md) 