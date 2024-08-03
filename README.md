# Minecraft Java Edition 1.21 with Docker
https://www.minecraft.net/en-us/article/minecraft-java-edition-1-21
https://zenn.dev/mesgory/articles/e3dc962bd38dfd

## VMへの接続

```
chmod 600 /path/to/ssh/key
ssh -i ./path/to/ssh/key opc@<your vm public ip>
```

## OCI VM へのインストール

### Docker, Docker Compose
ref: https://www.atlantic.net/dedicated-server-hosting/how-to-install-docker-and-docker-compose-on-oracle-linux/

```
### Docker

sudo -i
dnf update -y

dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo
dnf install docker-ce -y

systemctl start docker
systemctl enable docker
systemctl status docker

docker info
docker run hello-world
docker images | grep hello-world

docker ps -a
docker rm <CONTAINER ID>
docker rmi hello-world
```

```
### Docker Compose

# During Super User

export PATH=$PATH:/usr/local/bin
echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
source ~/.bashrc

dnf install -y curl

uname -m
curl -L "https://github.com/docker/compose/releases/download/v2.21.0/docker-compose-linux-$(uname -m)" -o /usr/local/bin/docker-compose

curl -L https://github.com/docker/compose/releases/download/v2.21.0/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

### Minecraft

Oracle Linux9 を使用する際は volumesの "- /etc/timezone:/etc/timezone:ro" を削除する

```
services:
  vanilla:
    container_name: mc_vanilla
    image: itzg/minecraft-server
    ports:
      - 25565:25565
    tty: true         # ターミナルの割り当て（`-t`オプション）
    stdin_open: true  # 標準入出力ストリーム（`-i`オプション）
    environment:
      ENABLE_ROLLING_LOGS: "TRUE"
      JVM_OPTS: "-XX:MaxRAMPercentage=75"
      TYPE: "VANILLA"
      EULA: "TRUE"
      VERSION: "1.21"
      MOTD: "The World of Vanilla Server"
      MAX_PLAYERS: 5
      MAX_WORLD_SIZE: 10000
      ENABLE_COMMAND_BLOCK: "TRUE"
      SNOOPER_ENABLED: "FALSE"
      VIEW_DISTANCE: 12
      MODE: "SURVIVAL"
      PVP: "FALSE"
      STOP_SERVER_ANNOUNCE_DELAY: 20
      GUI: "FALSE"
    volumes:
      - ./vanilla:/data
      - /etc/timezone:/etc/timezone:ro
    restart: unless-stopped
  vanilla-backups:
    container_name: mc_vanilla_backups
    image: itzg/mc-backup
    environment:
      BACKUP_NAME: "vanilla"
      BACKUP_INTERVAL: "12h"
      PRUNE_BACKUPS_DAYS: 5
      PAUSE_IF_NO_PLAYERS: "true"
      PLAYERS_ONLINE_CHECK_INTERVAL: "5m"
      INITIAL_DELAY: "2m"
      RCON_HOST: vanilla
    depends_on:
      - vanilla
    volumes:
      - ./vanilla:/data:ro
      - ./backups:/backups
      - /etc/timezone:/etc/timezone:ro
    restart: unless-stopped
    network_mode: "service:vanilla"
```

-usage

```
docker-compose up -d
```