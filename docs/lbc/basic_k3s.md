# Primeiros passos - Cluster com K3s

Neste tutorial introdutório serão apresentados os passos iniciais de utilização do K3s para a criação de um ambiente clusterizado em *Single-Board Computers (SBC)* como *Raspberry Pi*, *Orange Pi*, entre outros. Para maiores informações sobre o K3s, basta acessar a [documentação oficial](https://docs.k3s.io/).

Durante este tutorial serão abordados dois tipos de máquinas: 

- **Servers** - são as máquinas onde o K3s será instalado e terão as configurações principais com os manifestos yaml, contendo todas as informações para o deploy de aplicações, podendo ser sites com páginas estáticas, até projetos maiores envolvendo banco de dados.

- **Agents ou Workers** - são máquinas consideradas como nós, onde irão adicionar de forma complementar novas instâncias dos serviços configurados nos *Servers*. Ex.: em casos de serviços como *Load-Balancer*, quando maior a quantidade de máquinas, maior a disponibilidade de escalonamento e acesso a aplicação 

## Passo 01: Preparação de redes
- Neste passo, é de suma importância que os IPs de cada computador seja estático. Para isso, basta seguir os passos de configurações de rede [neste repositório](https://github.com/Prisma-IAP/basic_network_configs)

## Passo 02: Instalando o K3s em uma máquina Server
- Após acessar o terminal, sendo de forma remota, por ssh ou com o *SBC* conectado em um monitor, digite o comando abaixo:
```shell
curl -sfL https://get.k3s.io | sh -
```

- Caso opte por remover algum recurso complementar que é adicionado durante a instalação, como o *Traefik*, o comando pode ser realizado dessa forma:
```shell
curl -sfL https://get.k3s.io | sh -s - --disable=traefik
```

- Depois de realizar a instalação do K3s, aguarde aproximadamente 30 segundos para visualizar as informações do server com o comando:
```shell
kubectl get nodes
```

- Para verificar todos os pods disponíveis, digite o comando a seguir:
```shell
kubectl get pods --all-namespaces
```

- Para listar todas os status gerais do cluster, basta digitar o comando abaixo:
```shell
kubectl get all --all-namespaces
```

- Após realizar do status atual do server, obtenha o token que será utilizado durante a instalação do K3s nas máquinas workers. Para isso, digite o comando a seguir e copiei a resposta obtida: 
```shell
cat /var/lib/rancher/k3s/server/node-token
```

- Por fim, obtenha o ip do server com o comando ``ifconfig`` e anote as informações destas últimas duas etapas para o próximo passo.

## Passo 03: Instalando o K3s em uma máquina Worker
- Após obter o token do K3s e o ip do server no passo anterior, edite o comando abaixo, onde se encontram as variáveis ***server_ip*** e ***server_token*** para realizar a instalação do K3s na quantidade de workers que desejar
```shell
curl -sfL https://get.k3s.io | K3S_URL=https://server_ip:6443 K3S_TOKEN=server_token sh -
```

- No server, utilize o comando a seguir para visualizar a atualização dos node: 
```shell
watch -n 1 kubectl get nodes
```

- **Importante**: Para configurar um worker no Raspberry Pi, será necessário incluir as informações ``cgroup_memory=1 cgroup_enable=memory`` ao final do arquivo ``/boot/cmdline.txt`` caso ainda não existam.

## Passo 04 (Opcional): Desinstalando o K3s
- Em uma máquina server, utilize o comando: 
```shell
/usr/local/bin/k3s-uninstall.sh
```

- Em cada máquina worker, utilize o comando: 
```shell
/usr/local/bin/k3s-agent-uninstall.sh
```

## Passo 05 (Opcional): Solucionando problemas
- Caso seja necessário parar o server, use o comando: 
```shell
/usr/local/bin/k3s-killall.sh
```

- Para reiniciar o server: 
```shell
sudo systemctl restart k3s
```

- Para reiniciar o agent: 
```shell
sudo systemctl restart k3s-agent
```

## Passo 06 (Opcional): OpenLens Dashboard
O **OpenLens** é uma ferramenta de grande importância para garantir maior visibilidade, estatísticas em tempo real, estado atual dos serviços que estão funcionando no cluster, além de facilitar na solução de problemas.

- Em um computador conectado na mesma rede onde o cluster está sendo configurado, faça o [download](https://github.com/MuhammedKalkan/OpenLens/releases) do **OpenLens** para o sistema operacional desejado e realize a instalação

- Após realizar a instalação do OpenLens, selecione a opção *Add Cluster*

- Acesse o server através do prompt de comando e digite o comando abaixo: 
```shell
kubectl config view --minify --raw
```

- Copie os dados que foram exibidos no terminal e cole no OpenLens

- Altere o IP ***127.0.0.1:6443*** para o IP do **server** onde o K3s foi instalado