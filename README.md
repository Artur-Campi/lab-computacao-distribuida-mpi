# Laboratório de Computação Distribuída

Docker + Ubuntu + OpenMPI. Prof. Alcides / Prof. Mario.

## Como abrir

Abrir este repositório em um Codespace (botão Code > Codespaces > Create codespace on main).
O Codespace já sobe com Docker, OpenMPI e figlet instalados.

## Parte 1 - MPI local

```bash
mpirun --version

mpirun -np 4 hostname
mpirun -np 2 hostname
mpirun -np 1 hostname

nproc
lscpu

mpirun --oversubscribe -np 4 hostname

mpirun --oversubscribe -np 4 bash -c 'echo "Olá! Eu sou o processo $$"; sleep $((RANDOM % 5 + 1)); echo "Processo $$ terminou"'
mpirun --oversubscribe -np 4 bash -c 'echo "Olá! Eu sou o processo $$ e comecei a corrida."; sleep 0.$((RANDOM % 9 + 1)); echo "--> O processo $$ terminou!"'
mpirun --oversubscribe -np 5 date
mpirun --oversubscribe -np 4 bash -c 'echo "Olá do rank $OMPI_COMM_WORLD_RANK de $OMPI_COMM_WORLD_SIZE"'
```

## Parte 2 - Construindo o cluster

```bash
chmod +x start_cluster.sh
./start_cluster.sh
docker ps
```

## Parte 3 - SSH sem senha

```bash
chmod +x setup_ssh.sh
./setup_ssh.sh
docker compose exec -u mpiuser master ssh -o StrictHostKeyChecking=no worker1 hostname
```

## Parte 4 - Execução distribuída

```bash
docker compose exec master bash
su - mpiuser

cat > hosts << 'FIM'
master slots=2
worker1 slots=2
worker2 slots=2
worker3 slots=2
FIM

echo -e "Host worker1 worker2 worker3\n  StrictHostKeyChecking no" >> ~/.ssh/config
chmod 600 ~/.ssh/config

mpirun --hostfile hosts -np 8 hostname
mpirun --hostfile hosts -np 8 bash -c 'echo "Rank $OMPI_COMM_WORLD_RANK rodando em $(hostname)"'
mpirun --oversubscribe -np 30 --hostfile hosts hostname
```

## Exercício 1 - Figlet distribuído

```bash
mpirun --hostfile hosts -np 6 bash -c 'P=("Olá" "Mack," "sou" "o" "Artur" "Campi");
  B=$(figlet -C utf8 -f small "${P[$OMPI_COMM_WORLD_RANK]}");
  printf "[rank %s em %s]\n%s\n" "$OMPI_COMM_WORLD_RANK" "$(hostname)" "$B"'
```

## Exercício 2 - Cores e threads

```bash
nproc
lscpu
mpirun --hostfile hosts -np 8 hostname
mpirun --hostfile hosts -np 9 hostname
mpirun --oversubscribe --hostfile hosts -np 16 hostname
```

## Exercício 3 - Alfabeto

```bash
mpirun --oversubscribe --hostfile hosts -np 26 bash -c '
  L=$(printf "\\$(printf %o $((65+OMPI_COMM_WORLD_RANK)))");
  B=$(figlet -f small "$L");
  printf "rank %02d | host %s | letra %s\n%s\n" "$OMPI_COMM_WORLD_RANK" "$(hostname)" "$L" "$B"'
```

## Se o Codespace reiniciar

```bash
docker compose up -d
docker compose exec master service ssh start
```
