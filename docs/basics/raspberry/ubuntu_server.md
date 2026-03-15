# Configurações básicas - Ubuntu Server

Ao optar pela utilização do Ubuntu Server, configurações como a de rede local 
para determinar um IP fixo são um pouco diferentes, para isso, acesse o arquivo 
``/etc/netplan/50-cloud-init.yaml`` e deixe conforme o exemplo abaixo:
```yaml
network:
    ethernets:
        eth0:
            addresses: [192.168.0.10/24]
            gateway4: 192.168.0.1
            nameservers:
              addresses: [8.8.8.8, 1.1.1.1]
    version: 2
```

Atualize o horário das máquinas de acordo com o seu fuso horário com o comando:
``sudo dpkg-reconfigure tzdata``

Obs.: É possível que seja necessário instalar as bibliotecas: net-tools e ntpdate
