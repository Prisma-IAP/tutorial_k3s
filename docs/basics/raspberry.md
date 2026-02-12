# Configurações básicas - Raspberry Pi

O conteúdo a seguir tem como objetivo demonstrar como realizar as configurações básicas de rede no **[Raspberry Pi](https://www.raspberrypi.com/)**.

## Passo 01: Instalar o sistema operacional
- Para encontrar o sistema operacional desejado, realizar o download do ***Raspberry Pi Imager*** na seção *[Software](https://www.raspberrypi.com/software/)* no site oficial do **Raspberry Pi**

- Siga os passos no *Raspberry Pi Imager* e realize a instalação do SO desejado

- Ao término da instalação, realize as atualizações necessárias no SO


## Passo 02: Endereço IP fixo 
### 01: Distribuição Desktop
- Realize as atualizações de rede no arquivo ``/etc/dhcpcd.conf``, conforme o exemplo abaixo e salve as alterações em seguida
```shell
# fallback to static profile on eth0
interface eth0
static ip_address=192.168.0.10
static routers=192.168.0.1
static domain_name_servers=192.168.0.1
fallback static_eth0
```

### 02: Distribuição Server
- Edite as informações ao final da linha no arquivo ``/boot/cmdline.txt`` com o conteúdo abaixo. Ao término da edição, as informações editadas devem ficar no formato do exemplo a seguir.
```shell
#cgroup_memory=1 cgroup_enable=memory ip=ip_address::default_gateway:subnet_mask:seu_hostname:eth0:off
cgroup_memory=1 cgroup_enable=memory ip=192.168.0.10::192.168.0.1:255.255.255.0:node-pi:eth0:off
```
- Edite o arquivo ``/boot/config.txt`` e adicione a seguinte configuração ao final do arquivo
```shell
arm_64bit=1
```
- Crie um arquivo em branco ssh na pasta boot
- Como administrador, aplique o comando ``sudo iptables -F``
- Ao término das atualizações, use o comando ``sudo reboot`` para reiniciar o computador

## Passo 03: Atualizar o hostname
- No Raspberry é possível alterar através do *Raspberry Pi Imager* durante o processo de criação da imagem

---

# [Extra] Configurações para Ubuntu Server

## Em breve...
