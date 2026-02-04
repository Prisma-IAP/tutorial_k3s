# Gerando gráficos de distribuição e histogramas com a linguagem R

Nesta etapa iremos descrever como gerar gráficos de distribuição e histogramas utilizando a linguagem R.

## Passo 01: Instalando o R
- Acesse o _node01_ e faça a instalação de forma remota nos outros _nodes_ (indicando quantidade que você tiver disponível) através do seguinte comando:
```shell
sudo su -
srun --nodes=3 apt install r-base -y
```

- Ao término do processo, acesse os outros _nodes_ e digite o comando ```R --version``` para visualizar o resultado que será parecido com o exemplo abaixo:
```
root@node02:~# R --version
R version 3.4.4 (2018-03-15) -- "Someone to Lean On"
Copyright (C) 2018 The R Foundation for Statistical Computing
Platform: arm-unknown-linux-gnueabihf (32-bit)

R is free software and comes with ABSOLUTELY NO WARRANTY.
You are welcome to redistribute it under the terms of the
GNU General Public License versions 2 or 3.
For more information about these matters see
http://www.gnu.org/licenses/.
```

## Passo 03: Criando o algoritmo em R para teste
- Crie um arquivo em */clusterfs/normal/generate.R* com as seguintes informações:
```r
arg = commandArgs(TRUE)
samples = rep(NA, 100000)
for (i in 1:100000){ samples[i] = mean(rexp(40, 0.2)) }
jpeg(paste('plots/', arg, '.jpg', sep=""))
hist(samples, main="", prob=T, color="darkred")
lines(density(samples), col="darkblue", lwd=3)
dev.off()
```

- A seguir, crie uma nova pasta chamada *plots* e execute o comando a abaixo em um dos *nodes*:
```shell
mkdir plots
R --vanilla -f generate.R --args "plot1"
```

- Ao acessar a pasta *plots* será possível ver o resultado no arquivo *plot1.jpg*

## Passo 03: Criando um script batch com código em R
- Crie um novo arquivo */clusterfs/normal/submit.sh* com as seguintes informações:
```shell
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --partition=<partition name>
cd $SLURM_SUBMIT_DIR
mkdir plots
R --vanilla -f generate.R --args "plot$SLURM_ARRAY_TASK_ID"
```

- Acesse o *node01* e rode o job criado no passo anterior. Para isso, siga os passos abaixo:
```shell
cd /clusterfs/normal
sbatch --array=[1-50] submit.sh
```

- Após a inicialização do job, execute o comando ```squeue```. O retorno será conforme abaixo:
```
    JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
11_[9-50] mycluster submit.s     root PD       0:00      1 (Resources)
     11_1 mycluster submit.s     root  R       0:03      1 node02
     11_2 mycluster submit.s     root  R       0:03      1 node02
     11_3 mycluster submit.s     root  R       0:03      1 node02
     11_4 mycluster submit.s     root  R       0:03      1 node02
     11_5 mycluster submit.s     root  R       0:03      1 node03
     11_6 mycluster submit.s     root  R       0:03      1 node03
     11_7 mycluster submit.s     root  R       0:03      1 node03
     11_8 mycluster submit.s     root  R       0:03      1 node03
```

- Por fim, acesso o diretório */clusterfs/normal/plots* para visualizar os plots que foram criados