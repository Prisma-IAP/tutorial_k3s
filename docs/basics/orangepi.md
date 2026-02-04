# Configurações básicas - Orange Pi

O conteúdo a seguir tem como objetivo demonstrar como realizar as configurações básicas de rede no **[Orange Pi](http://www.orangepi.org/)**.

## Passo 01: Instalar o sistema operacional
- Para encontrar o sistema operacional desejado, basta acessar a seção *[Downloads](http://www.orangepi.org/html/serviceAndSupport/index.html)* no site oficial do **Orange Pi**

- Após realizar o download, utilize a ferramenta de sua preferência para gravar no SO em seu SD Card. *ex: Balena Etcher, Rufus, etc*

- Ao término da instalação, realize as atualizações necessárias no SO


## Passo 02: Endereço IP fixo 
- Realize as atualizações de rede no arquivo ``/etc/network/interfaces``, conforme o exemplo abaixo e salve as alterações em seguida
```shell
#Static Ip Address
auto eth0
iface eth0 inet static
address 192.168.0.30
netmask 255.255.255.0
gateway 192.168.0.1
```

## Passo 03: Atualizar o hostname
- Utilize o comando ``hostname nome_desejado`` 
- Acesse o arquivo em ``/etc/hostname``, substitua pelo mesmo nome informado na etapa acima
- Ao término das atualizações, use o comando ``sudo reboot`` para reiniciar o computador