# 🐧 Preparar o Linux Mint

Passo a passo para deixar o desktop pronto. Faça uma vez só.

## 1. Ferramentas básicas

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl git python3-venv python3-pip postgresql-client
```

O `postgresql-client` instala só o cliente `psql`, para você se conectar ao banco. O servidor vai rodar no Docker.

## 2. Docker Engine (repositório oficial)

O Linux Mint é baseado no Ubuntu. Use o **codinome do Ubuntu** (`UBUNTU_CODENAME`), e não o do Mint, senão o repositório não é encontrado.

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$UBUNTU_CODENAME") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Referência oficial: https://docs.docker.com/engine/install/ubuntu/

## 3. Usar o Docker sem `sudo`

```bash
sudo usermod -aG docker $USER
```

Depois, **saia da sessão e entre de novo**, ou reinicie.

## 4. Testar

```bash
docker run hello-world
docker compose version
psql --version
git --version
python3 --version
```

Todos devem responder sem erro.

## 5. Clonar o projeto

```bash
mkdir -p ~/dev && cd ~/dev
git clone https://github.com/Bolapipa/fleet-ops-db.git
cd fleet-ops-db
```

Configure a sua identidade neste repositório:

```bash
git config user.name "Arthur Delacorte"
git config user.email "delacortearthur@gmail.com"
```

## 6. Opcional, mas recomendado

- **DBeaver Community** (Central de Programas do Mint): cliente visual para ver as tabelas e rodar consultas.
- **VS Code** com as extensões *PostgreSQL* e *Python*.
- **GitHub CLI** (`sudo apt install gh`) para trabalhar com as issues pelo terminal: `gh issue list`.

✅ Tudo funcionando? Vá para a **Fase 0** em [ROADMAP.md](ROADMAP.md).
