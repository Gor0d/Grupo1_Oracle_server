# Atividade Grupo 1 - Kafka

## Instalação do Linux 

Utilize a última versão estável disponibilizada do Oracle Linux:
[OracleLinux-R8-U6-x86_64-dvd.iso](https://yum.oracle.com/oracle-linux-isos.html)

### Particionamento do disco - LVMs

Na instalação do SO Oracle Linux, em INSTALLATION DESTINATION, selecionar o LOCAL STANDARD DISKS, e em STORAGE CONFIGURATION selecionar a opção CUSTOM. Clique em DONE.

Surgirá uma nova tela MANUAL PARTITIONING. Inclua 6 repartições:
/boot xfs
/swap swap
/home xfs
/var xfs
/tmp xfs
/  xfs

## No terminal

### Rede da máquina virtual em CIDR/24

Instale o pacote net-tools:
sudo yum install net-tools 

Verifique a máscara de rede (255.255.255.0):
if config -a

### Configurando Hostname

Edite o arquivo /etc/hosts:
sudo vi /etc/hosts

Reinicie a máquina:
reboot

### DNS com o nome kafkadockerlab

Edite o arquivo /etc/hosts:
sudo vi /etc/hosts

Inclua uma nova linha com o ip da máquina virtual e o nome "kafkadockerlab":
10.0.2.15  kafkadockerlab

Reinicie a máquina:
reboot

### IP Fixo

Verifique o nome da sua placa de rede, o seu IP e a sua mascara de rede:
ifconfig -a 

Edite o arquivo ifcfg-nome_da_sua_placa:
sudo vi /etc/sysconfig/network-scripts/ifcfg-enp0s3

Adicione as seguintes linhas:
IPADDR= 10.0.2.15
NETMASK=255.255.255.0
GATEWAY=10.0.2.2 
BROADCATS=10.0.2.255
DNS=8.8.8.8

Altere a linha do BOOTPROTO, substituindo o "dhcp" para "none".

### Configurando SSH

Instale o software SSH:
sudo yum install openssh-server

Ative o SSH para iniciá-lo automaticamente junto ao sistema:
sudo systemctl enable sshd

### Bloquando o acesso SSH para o root

Edite o arquivo sshd_config:
sudo vim /etc/ssh/sshd_config

Altere a linha "PermitRootLogin", substituindo o "yes" por "no".

## Conectando o Windows com a VM via SSH

### Na VM

Nas configurações de REDE da VirtualBox, deixe conectado a NAT. Clique em REDIRECIONAMENTO DE REDES e adicione a linha:
Rule 1  TCP  192.168.3.17  2222  10.0.2.15  22

### No Windows

Edite como administrador o arquivo:
c:\windows\system32\drivers\etc\hosts

Adicione a linha "IP_máquina_física -p 2222 DNS DNS.localdomain":
192.168.3.17 -p 2222 kafkadockerlab kafkadockerlab.localdomain.

Para conectar via SSH, vá no Prompt de Comando do Windows e execute:
ssh usuário@kakfkadockerlab -p 2222

### Filesystem /var/lib/docker - 10GB e ext4

Com a máquina desligada, vá em ARMAZENAMENTO na virtualbox, e adicione um segundo Disco Rígido(HD) de 12GB.

No terminal, verifique se o segundo HD foi criado:
lsblk

Com o SDB criado, o particione:
sudo fdisk /dev/sdb

Digite:
n-nova partição
p-partição primária
1- número da partição
ENTER
ENTER
w-salvar

Depois de particionado o disco, crie um sistema de arquivos:
sudo mkfs -t ext4 /dev/sdb1

Crie um diretório onde será montado o SBD1:
sudo mkdir /var/lib/docker

Edite o arquivo fstab para montar o SDB1 automaticamente dentro do diretório criado:
sudo vi /etc/fstab

Adicione a seguinte linha:
/dev/sdb1 /var/lib/Docker ext4 defaults 0 0 

### Instalando o Docker

Instale o pacote yum-utils:
sudo yum install yum-utills

Adicione o repositório do Docker:
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

Instale os pacotes do Docker:
sudo yum install docker-ce docker-ce-cli

Inicie os serviços do Docker:
sudo systemctl start docker

Habilite o serviço do Docker:
sudo systemctl enable docker

Verifique o status do Docker:
sudo systemctl status docker

Baixe e execute a imagem do Docker da biblioteca online do Docker:
sudo docker run hello-word

### Instalando Docker Compose

Instale o comando utilitário curl através do comando:

dnf install -y curl

Realize o download da última versão do Docker Compose através do caminho:

curl -L https://github.com/docker/compose/releases/download/v2.5.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose

Execute permissão para o Docker Compose com o comando

chmod +x /usr/local/bin/docker-compose

### Instalando imagem Kafka e configurando tópico

Para que o kafka funcione é necessário instalar duas imagens, do zookeeper e a do kafka. Para não ter ser necessário realizar o download de forma manual, siga os seguintes passos:
Abra o arquivo docker-compose.yml e copie o código abaixo:

version: '3'

services:
zookeeper:
  image: wurstmeister/zookeeper
  container_name: zookeeper
  ports:
    - "2181:2181"
kafka:
  image: wurstmeister/kafka
  container_name: kafka
  ports:
    - "9092:9092"
environment:
KAFKA_ADVERTISED_HOST_NAME: kafkadockerlab
KAFKA_ZOOKEEPER_CONNECT: zookeeper:2181

Para subir as imagens e gerar os containers, execute o comando:
docker-compose -f docker-compose.yml up -d

Para verificar se ambos os container estão rodando digite o comando:
docker ps

Com zookeeper e kafka funcionando, digite o comando abaixo para "startar" o Kafka no shell:
docker exec -it kafka /bin/sh (substitua o nome "kafka" pelo nome do container caso tenha definido diferente no seu docker-compose.yml)

Todos os scripts do kafka ficam alocados em /opt/kafka_<versão>/bin

Digite o comando a seguir para criar um tópico:
kafka-topics.sh --create --zookeeper zookeeper: 2181 --replication-factor 1 --partitions 1 --topic <nome do tópico>

E tópico será criado. Para listar todos os tópicos criados dentro do Kafka utilizar o comando:
kafka-topics.sh --list --zookeeper zookeeper:2181
