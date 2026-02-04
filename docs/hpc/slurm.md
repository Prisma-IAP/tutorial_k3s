# Comandos básicos - SLURM  

- ***sinfo***: disponibiliza informações relacionadas ao cluster
```
$ sinfo
PARTITION   AVAIL   TIMELIMIT   NODES   STATE   NODELIST
mycluster*     up    infinite       3    idle   node[01–03]
```

- ***scontrol***: com este comando é possível visualizar o status atual do job que está sendo executado, além de atualizar o status dos nodes caso estejam em algum status diferente de *idle*
```shell
# atualiza o status dos nós que compõem o cluster
scontrol update nodename=node[02-03] state=idle

# disponibiliza o status atual do job
scontrol show job <JOBID>
```

- ***srun***: é utilizado para executar um comando diretamente em quantos _nodes_/núcleos você desejar
```
$ srun --nodes=3 hostname
node01
node02
node03
```

- Também é possível executar as tarefas em um único _node_, dessa forma o resultado será diferente do exemplo acima
```
$ srun --ntasks=3 hostname
node02
node02
node02
```

- Outro parâmetro importante é o _ntasks_, com ele é possível indicar a quantidade de tarefas por _node_
```
$ srun --nodes=2 --ntasks-per-node=3 hostname
node02
node02
node02
node03
node03
node03
```

- ***squeue***: é utilizado para visualizar os jobs agendados
```
$ squeue
JOBID PARTITION  NAME     USER ST    TIME  NODES NODELIST(REASON)
  609 mycluster  24.sub.s pi   R     10:16     1 node02
```

- ***scancel***: é utilizado para cancelar um job em execução
```shell
# scancel JOBID
scancel 609
```

- ***sbatch***: é utilizado para executar um script batch

- Para exemplificar, crie um arquivo _/clusterfs/helloworld.sh_ e adicione as seguintes informações (não esqueça de substituir o _partition name_ pelo nome do da sua PARTITION, ex: ***mycluster***):
```shell
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
#SBATCH --partition=<partition name>
cd $SLURM_SUBMIT_DIR
echo "Hello, World!" > helloworld.txt
```

- Por fim, basta digitar o comando a seguir para que o job seja executado:
```shell
sbatch ./helloworld.sh
```

- Não irá aparecer nenhuma mensagem na tela, no entanto, será possível visualizar que um novo arquivo foi criado _/clusterfs/helloworld.txt_

- Em caso de erros, um arquivo com o nome _slurm-XXX.out_, onde XXX é o código do job que foi executado