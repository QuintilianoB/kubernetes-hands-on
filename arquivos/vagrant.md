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
```
vagrant up
```
Aguarde o final da instalação:
