# Configuração básica - Single Board HPC Cluster

## Passo 00: Preparação do hardware
Como alternativa, iremos alterar o Raspberry Pi pelo Orange Pi, a fim de buscar por imcompatibilidades durante a construção do cluster.

- 3x Orange Pi Pc 
- 1x Raspberry Pi 3 Model B
- 4x MicroSD Cards
- 4x Fontes/Cabos de força
- 1x Switch 8-portas 10/100/1000
- 1x Pen Drive 64GB (estamos usando um menor com 16GB)
- Cabos de rede _Cat5e_

## Passo 01: Instalação do Sistema Operacional
Durante essa etapa, basta seguir as instruções de **[outro tutorial](https://github.com/Prisma-IAP/basic_network_configs)** presente neste repositório para a instalação do sistema operacional e definição dos _hostnames_ de cada computador.

**_Obs_**: Durante a definição de hostnames, de preferência para nomes sem espaços, ex: _node01_, _node02_, _node03_, _node04_.

## Passo 02: Sincronização de data/hora

O _SLURM_ e o _MUNGE_ necessitam que as configurações de data/hora estejam devidamente sincronizadas, para esta etapa basta realizar a seguinte instalação: 

```shell
sudo apt install ntpdate -y
```

Ao término do processo, basta reiniciar cada computador.

## Passo 03: Preparação do armazenamento compartilhado
- Conecte o pen drive no _node01_ que será definido como _Node Master_

- Acesse o _node01_ através de outro terminal de forma remota

- Identifique as informações do dispositivo móvel com o comando a seguir:
```shell
lsblk
```
- Provavelmente o caminho do dispositivo será _/dev/sda1_

- Realize a formatação do pen drive com o comando a seguir:
```shell
sudo mkfs.ext4 /dev/sda1
```

- Crie um novo diretório que servirá como ponto de montagem entre todos os nós, para isso, siga as instruções abaixo:
```shell
sudo mkdir /clusterfs
sudo chown nobody.nogroup -R /clusterfs
sudo chmod 777 -R /clusterfs
```

- Encontre as informações de UUID do ponto _/dev/sda1_, para isso, utilize o comando a seguir:
```shell
blkid
```

- Resultado será parecido com o demonstrado a seguir:
```shell
/dev/sda1: UUID="017c30aa-4589-4ccb-a8b5-44d2834f8af8"
```

- A seguir edite o arquivo ***/etc/fstab***, adicionando o UUID e outras informações de acordo com o exemplo abaixo
```shell
UUID=017c30aa-4589-4ccb-a8b5-44d2834f8af8 /clusterfs ext4 defaults 0 2
```

- Por fim, monte a unidade com o comando a seguir:
```shell
sudo mount -a
```

- Antes de finalizar esta etapa, é importante definir algumas permissões de acordo com os comandos a seguir:
```shell
sudo chown nobody.nogroup -R /clusterfs
sudo chmod -R 766 /clusterfs
```

## Passo 04: Configuração NFS (Network File System)

### Passo 01: Instalação do NFS Server
- Realize o processo de instalação no ***node01*** com o comando a seguir:
```shell
sudo apt install nfs-kernel-server -y
```

- Edite o arquivo _/etc/exports_ e adicione a linha abaixo:
```
/clusterfs    <ip addr>(rw,sync,no_root_squash,no_subtree_check)
```

- Substitua o _<ip addr>_ pelo endereço IP inicial da sua rede, dessa forma, qualquer outro computador poderá acessar o ponto de montagem. Por exemplo, se os endereços da sua LAN forem no padrão _192.168.1.X_, o resultado será conforme abaixo:
```shell
/clusterfs 192.168.1.0/24(rw,sync,no_root_squash,no_subtree_check)
```

- Por fim, atualize o NFS kernel server com o comando a seguir:
```shell
sudo exportfs -a
```

### Passo 02: Instalação do NFS Client
- Realize o processo de instalação nos outros ***nodes*** com o comando a seguir:
```shell
sudo apt install nfs-common -y
```

- Crie o mesmo novo diretório que será utilizado como ponto de montagem pelo ***node01***, para isso, siga as instruções abaixo:
```shell
sudo mkdir /clusterfs
sudo chown nobody.nogroup -R /clusterfs
sudo chmod 777 -R /clusterfs
```

- Edite o arquivo _/etc/fstab_ e adicione a seguinte linha:
```
<master node ip>:/clusterfs    /clusterfs    nfs    defaults   0 0
```
- A linha adicionada deve ficar conforme o exemplo abaixo:
```shell
192.168.0.20:/clusterfs    /clusterfs    nfs    defaults   0 0
```

- Por fim, para que seja possível criar arquivos e compartilhar entre todos os _nodes_, monte a unidade com o comando a seguir:
```shell
sudo mount -a
```

## Passo 05: Configuração SLURM (Master Node)

### Passo 01: Mapeamento dos hosts
- Para facilitar o mapeamento dos _nodes_ é necessário editar o arquivo _/etc/hosts_ no _node01_ e adicionar as seguintes linhas:
```
<ip addr of node02>      node02
<ip addr of node03>      node03
<ip addr of node04>      node04
```

### Passo 02: Instalação do SLURM
- No _node01_ realize a instalação com o seguinte comando:
```shell
sudo apt install slurm-wlm -y
```

- Faça uma cópia, extraia os arquivos base com os comandos a seguir:
```shell
cd /etc/slurm-llnl
cp /usr/share/doc/slurm-client/examples/slurm.conf.simple.gz .
gzip -d slurm.conf.simple.gz
mv slurm.conf.simple slurm.conf
```

- Edite o arquivo _/etc/slurm-llnl/slurm.conf_, adicionando as informações abaixo:
```shell
ControlMachine=node01
ControlAddr=192.168.0.20

SelectType=select/cons_res
SelectTypeParameters=CR_Core
ClusterName=HPCluster

NodeName=node01 NodeAddr=192.168.0.20 CPUs=4 State=UNKNOWN
NodeName=node02 NodeAddr=192.168.0.21 CPUs=4 State=UNKNOWN
NodeName=node03 NodeAddr=192.168.0.22 CPUs=4 State=UNKNOWN

PartitionName=mycluster Nodes=node[02-03] Default=YES MaxTime=INFINITE State=UP
```
***Obs:*** Dependendo da versão do _SLURM_ as configurações de _ControlMachine_ e _ControlAddr_ deverão ser substituídas pela seguinte configuração:
```shell
SlurmctldHost=node01(<ip addr of node01>)
# e.g.: node01(192.168.1.14)
```

- Crie o arquivo _/etc/slurm-llnl/cgroup.conf_ com as seguintes configurações:
```shell
CgroupMountpoint="/sys/fs/cgroup"
CgroupAutomount=yes
CgroupReleaseAgentDir="/etc/slurm-llnl/cgroup"
AllowedDevicesFile="/etc/slurm-llnl/cgroup_allowed_devices_file.conf"
ConstrainCores=no
TaskAffinity=no
ConstrainRAMSpace=yes
ConstrainSwapSpace=no
ConstrainDevices=no
AllowedRamSpace=100
AllowedSwapSpace=0
MaxRAMPercent=100
MaxSwapPercent=100
MinRAMSpace=30
```

- Por fim, crie o arquivo */etc/slurm-llnl/cgroup_allowed_devices_file.conf* e adicione as seguintes informações:
```shell
/dev/null
/dev/urandom
/dev/zero
/dev/sda*
/dev/cpu/*/*
/dev/pts/*
/clusterfs*
```

### Passo 03: Copiar os arquivos de configuração para a pasta compartilhada
- Realize os comandos a seguir para copiar os arquivos de configuração e a chave de autenticação do _Munge_
```shell
sudo cp slurm.conf cgroup.conf cgroup_allowed_devices_file.conf /clusterfs
sudo cp /etc/munge/munge.key /clusterfs
```

### Passo 04: Iniciar e habilitar os serviços do SLURM
- Ative o Munge:
```shell
sudo systemctl enable munge
sudo systemctl start munge
```

- Ative o SLURM daemon:
```shell
sudo systemctl enable slurmd
sudo systemctl start slurmd
```

- Ative o control daemon:
```shell
sudo systemctl enable slurmctld
sudo systemctl start slurmctld
```

### Passo 05: Reiniciar (Opcional)
- Caso ocorra algum problema com a autenticação do Munge ou não houver comunicação com o SLURM Controller, basta reiniciar o _node01_

## Passo 06: Configuração SLURM (Nodes)

### Passo 01: Instalação do SLURM Client
- Em cada um dos _nodes_ realize a instalação com o seguinte comando:
```shell
sudo apt install slurmd slurm-client -y
```

- Para facilitar o mapeamento dos _nodes_ da mesma forma que foi realizado no _node01_, para isso, edite o arquivo _/etc/hosts_ e adicione as seguintes linhas:
```shell
#node02:/etc/hosts
<ip addr of node01>      node01
<ip addr of node03>      node03
<ip addr of node04>      node04
```

- Copie os arquivos de configuração que foram compartilhados em _/clusterfs_, para isso siga os seguintes passos:
```shell
sudo cp /clusterfs/munge.key /etc/munge/munge.key
sudo cp /clusterfs/slurm.conf /etc/slurm-llnl/slurm.conf
sudo cp /clusterfs/cgroup* /etc/slurm-llnl
```

### Passo 02: Testando o Munge
- Para testar a chave Munge que foi copiada, inicie e ative-o com os seguintes comandos:
```shell
sudo systemctl enable munge
sudo systemctl start munge
```

- Acesse cada um dos _nodes_, realizando o comando abaixo para validar se a autenticação Munge, que foi configurada no _node01_ está funcionando corretamente
```shell
# conectando em um Raspberry
ssh pi@node01 munge -n | unmunge

# conectando em um Orange
ssh root@node01 munge -n | unmunge
```

- Caso a autenticação funcione, o resultado será parecido com o exemplo abaixo:
```
root@node01's password:
STATUS:           Success (0)
ENCODE_HOST:      node01 (192.168.0.20)
ENCODE_TIME:      2023-11-18 02:45:52 -0300 (1700286352)
DECODE_TIME:      2023-11-18 02:45:52 -0300 (1700286352)
TTL:              300
CIPHER:           aes128 (4)
MAC:              sha256 (5)
ZIP:              none (0)
UID:              root (0)
GID:              root (0)
LENGTH:           0
```
***Obs:*** Caso o processo não funcione, reinicie todos os _nodes_ e tente novamente

### Passo 03: Ativação do SLURM
- Ative o SLURM daemon:
```shell
sudo systemctl enable slurmd
sudo systemctl start slurmd
```

## Passo 07: Testando o SLURM
- No _node01_, digite o comando ``sinfo`` para verificar o status atual de cada _node_. O status desejado deve estar da seguinte forma:
```
PARTITION  AVAIL  TIMELIMIT  NODES  STATE NODELIST
mycluster*    up   infinite      3   idle node[02-04]
```
- Caso ocorra algum erro, tente atualizar o status de cada _node_ com o comando abaixo:
```shell
# substitua "node00" pelo nome do node desejado
scontrol update nodename=node00 state=idle
```

- A seguir, realize um teste para que retorne os nomes dos _nodes_ com o comando ``hostname``. Para isso, digite o comando abaixo no _node01_
```shell
srun --nodes=3 hostname
```
- O resultado será mostrado da seguinte forma:
```
node02
node03
node04
```

## Troubleshooting
- Visualizar se a pasta compartilhada existe: 
```shell
df -h
```

- Forçar a montagem da pasta compartilhada: 
```shell
mount -t nfs <nfs_server_ip>:<shared_folder_path> <mount_point>
```

- Forçar a alteração de status dos _nodes_:
```shell
# substitua "node00" pelo nome do node desejado
scontrol update nodename=node00 state=idle
```