# Vagrant

Utilizaremos o Vagrant apenas para configuração do sistema operacional.
Definição de disco, memória e instalação do SO são itens que não
afetam diretamente esse laboratório e por isso passamos direto.

Para mais informações sobre o Vagrant ou sobre a estrutura do
Vagrantfile: [RTFM](https://www.vagrantup.com/docs/index.html).

#### Vagrantfile

O [arquivo](Vagrantfile) possui a configuração básica necessária
para configurar os 3 hosts apresentados no layout inicial.

Após a execução, será necessário um passo adicional de configuração.
Precisaremos alterar a rede do VirtualBox para permitir que todos os
hosts se comuniquem através do mesmo enlace virtual.

#### Iniciando o Vagrant
É necessário que o Vagrant já esteja instalado!

1 - Baixe o arquivo [Vagrantfile](Vagrantfile)

2 - Abra o terminal/shell, navegue até onde salvou o Vagrantfile e execute:
```bash
vagrant up
```
Aguarde o final da instalação:

```bash
    node2: Complete!
==> node2: Running provisioner: shell...
    node2: Running: inline script
==> node2: Running provisioner: shell...
    node2: Running: inline script
```

Além de configurar os hosts, o Vagrant também configura portas
de acesso (NAT), permitindo o SSH nos 3 hosts criados:

```
[quintiliano@Jhon arquivos]$ ss -natl | grep 70
LISTEN     0      10           *:7022
LISTEN     0      10           *:7023
LISTEN     0      10           *:7024
```

A relação de portas:

* Porta 7022 -> Master
* Porta 7023 -> Node 1
* Porta 7024 -> Node 2

**Importante**

Teste o acesso aos 3 hosts antes de continuar!

As credenciais de acesso padrão:
```
Usuário: vagrant
Senha:  vagrant
```

- Master: ```ssh vagrant@127.0.0.1 -p 7022```
- Node 1: ```ssh vagrant@127.0.0.1 -p 7023```
- Node 2: ```ssh vagrant@127.0.0.1 -p 7024```

Caso não funcione, verifique os passos realizados. O próximo passo
poderá deixar você trancado de fora caso não tenha verificado esse
acesso.

**Aproveite para desligar as 3 VMs!!** `[#f03c15]`

#### Criar rede local!

O último passo é realizar a criação de uma rede local para permitir
a conexão direta entre as máquinas virtuais. Isso é feito direto no
VirtualBox:

- 1 - Selecione a opção **Arquivos (Files)** -> **Preferências (Preferences)**
- 2 - No menu apresentado, selecione a opção **Network**:

![virtual_box_network](virtual_box_network.png)

- 3 - No menu à esquerda, adicione uma nova rede chamada **k8s**.
A faixa de rede indicada é 10.100.100.0/24. Habilite a utilização do
DHCP.

![k8s_network](k8s_network.png)

#### Configurar o IP nos hosts!

Após criar a rede local, é necessário mover os três hosts criados
para a nova rede. Para isso, em cada um dos hosts, com o botão direito
selecione a opção **Configuração (Settings)**:





- 4 - Ainda no menu de configurações da rede **k8s**, selecione
a opção **Redirecionamento de porta (Port Forwarding)**










