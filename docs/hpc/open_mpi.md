# OpenMPI

## Passo 01: Instalação do OpenMPI
- Acesse o _node01_ e realize a instalação do ***OpenMPI*** e suas dependências em cada *node*. Para isso, utilize os seguintes comandos:
```shell
sudo su -
srun --nodes=3 apt install openmpi-bin openmpi-common libopenmpi3 libopenmpi-dev -y
```

- A seguir, realize o primeiro teste criando o arquivo */clusterfs/hello_mpi.c* com o seguinte conteúdo:
```c
#include <stdio.h>
#include <mpi.h>
int main(int argc, char** argv) {
    int node;
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &node);
    printf("Hello World from Node %d!\n", node);
    MPI_Finalize();
}
```

- Em seguida é necessário compilar o código em C para que seja possível rodar em cada *node* a partir do arquivo ***a.out***. Para isso, realize as seguintes etapas:
```shell
login1$ srun --pty bash
node1$ cd /clusterfs
node1$ mpicc hello_mpi.c
node1$ ls
a.out* hello_mpi.c
node1$ exit
```

- Crie um script com o nome */clusterfs/sub_mpi.sh* para que seja possível rodar no cluster. Utilize o conteúdo a seguir:
```shell
#!/bin/bash
cd $SLURM_SUBMIT_DIR
# Print the node that starts the process
echo "Master node: $(hostname)"
# Run our program using OpenMPI.
# OpenMPI will automatically discover resources from SLURM.
mpirun --allow-run-as-root a.out
```

- Por fim, execute o job seguindo os seguintes comandos:
```shell
cd /clusterfs
sbatch --nodes=3 --ntasks-per-node=2 sub_mpi.sh
```

- O resultado final deve ser parecido com o informado abaixo:
```
Master node: node01
Hello World from Node 0!
Hello World from Node 1!
Hello World from Node 2!
Hello World from Node 3!
Hello World from Node 4!
Hello World from Node 5!
```