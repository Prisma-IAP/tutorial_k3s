# Testes Python e MPI

## Passo 01: Instalação de bibliotecas Python
- Crie um novo arquivo em */clusterfs/calc-pi/sub_install_pip.sh* e adicione as seguintes informações:
```sh
#!/bin/bash
#SBATCH --nodes=1
#SBATCH --ntasks-per-node=1
/clusterfs/usr/bin/pip3 install numpy mpi4py
```

- Execute o job como adminstrador para que seja possível realizar as modificações na versão Python instalada:
```sh
cd /clusterfs/calc-pi
sudo su
sbatch sub_install_pip.sh
```

- aaa
```python
from mpi4py import MPI
from math   import pi as PI
from numpy  import array

def comp_pi(n, myrank=0, nprocs=1):
    h = 1.0 / n
    s = 0.0
    for i in range(myrank + 1, n + 1, nprocs):
        x = h * (i - 0.5)
        s += 4.0 / (1.0 + x**2)
    return s * h

def prn_pi(pi, PI):
    message = "pi is approximately %.16f, error is %.16f"
    print  (message % (pi, abs(pi - PI)))

comm = MPI.COMM_WORLD
nprocs = comm.Get_size()
myrank = comm.Get_rank()

n    = array(0, dtype=int)
pi   = array(0, dtype=float)
mypi = array(0, dtype=float)

if myrank == 0:
    _n = 20 # Enter the number of intervals
    n.fill(_n)
comm.Bcast([n, MPI.INT], root=0)
_mypi = comp_pi(n, myrank, nprocs)
mypi.fill(_mypi)
comm.Reduce([mypi, MPI.DOUBLE], [pi, MPI.DOUBLE],
            op=MPI.SUM, root=0)
if myrank == 0:
    prn_pi(pi, PI)
```