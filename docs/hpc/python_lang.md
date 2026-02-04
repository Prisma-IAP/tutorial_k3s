# Instalação Python

## Passo 01: Instalando dependências
- Instale as bibliotecas do Python no _node01_, seguindo o comando abaixo:
```shell
sudo apt install -y build-essential python-dev python-setuptools python-pip python-smbus libncursesw5-dev libgdbm-dev libc6-dev zlib1g-dev libsqlite3-dev tk-dev libssl-dev openssl libffi-dev
```

## Passo 02: Configuração e build do Python
- Realize o download e descompacte o Python na versão indicada abaixo:
```sh
cd /clusterfs && mkdir build && cd build
wget https://www.python.org/ftp/python/3.7.3/Python-3.7.3.tgz
tar xvzf Python-3.7.3.tgz
... tar output ...
cd Python-3.7.3
```

- Configure o Python no _node01_ conforme os passos abaixo:
```shell
mkdir /clusterfs/usr        # directory Python will install to
$ cd /clusterfs/build/Python-3.7.3
$ srun --nodelist=node01 bash  # configure will be run on node01
node01$ ./configure --enable-optimizations --prefix=/clusterfs/usr --with-ensurepip=install
```

- Crie um novo script com o nome *sub_build_python.sh* com as instruções abaixo:
```shell
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=4
#SBATCH --nodelist=node1
cd $SLURM_SUBMIT_DIR
make -j4
```

- A seguir execute acesse a pasta com a versão do Python e execute o script utilizando os recursos do _SLURM_: 
```shell
cd /clusterfs/build/Python-3.7.3
sbatch sub_build_python.sh
```

- Para visualizar os logs da instalação, utilize o comando abaixo:
```shell
tail -f slurm-xxx.out # xxx é o número do JOBID
```

## Passo 03: Instalação do Python
- Crie um novo arquivo com o nome *sub_install_python.sh* no diretório */clusterfs/build/Python-3.7.3* com as seguintes instruções:
```shell
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --nodelist=node01
cd $SLURM_SUBMIT_DIR
make install
```

- Execute o novo script seguindo os passos abaixo:
```shell
sudo su -
cd /clusterfs/build/Python-3.7.3
sbatch sub_install_python.sh
```

- Também é possível visualizar os logs da instalação utilizando o comando abaixo:
```shell
tail -f slurm-xxx.out # xxx é o número do JOBID
```

## Passo 04: Testes 
- Realize um teste para verificar se o Python foi devidamente instalado no dispositivo compartilhado com o seguinte comando:
```shell
srun --nodes=3 /clusterfs/usr/bin/python3 -c "print('Hello')"
# Hello
# Hello
# Hello
```

- Por fim, verifique se o *pip* está acessível com o comando abaixo:
```shell
srun --nodes=1 /clusterfs/usr/bin/pip3 --version
# pip 19.0.3 from /clusterfs/usr/lib/python3.7/site-packages/pip (python 3.7)
```