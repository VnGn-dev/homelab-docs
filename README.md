# Homelab

Servidor doméstico que eu montei e mantenho para estudar infraestrutura,
Docker e desenvolvimento back-end. Tudo roda em containers, é acessado de
qualquer lugar por VPN (Tailscale) e é documentado aqui: a configuração de
cada serviço e os problemas que encontrei no caminho, com causa e solução.

```
 Notebook / celular ──┐
                      │  Tailscale (VPN privada, sem abrir portas no modem)
                      ▼
              ┌───────────────────────────────┐
              │  homelab (Ubuntu Server)      │
              │                               │
              │  Docker                       │
              │   ├─ AdGuard Home    DNS      │
              │   ├─ Jellyfin        mídia    │──► GTX 1650 (transcodificação)
              │   ├─ Portainer       gestão   │
              │   └─ code-server     dev      │
              └───────────────────────────────┘
```

## Hardware
- CPU: Intel i5-9400F
- RAM: 16 GB
- GPU: NVIDIA GTX 1650 (4 GB), usada pelos containers via NVIDIA Container Toolkit
- Rede: Ethernet (Wi-Fi Realtek RTL8821AE disponível)

| Disco | Montado em | Tamanho | Uso |
|---|---|---|---|
| SSD | `/` | 116 GB | sistema |
| SSD | `/srv/ssd` | 220 GB | projetos e dados do Docker |
| HD | `/srv/dados` | 458 GB | filmes e séries |
| HD | `/srv/backup` | 458 GB | backups |

## Software
- Ubuntu Server 26.04 LTS
- Docker e Docker Compose
- Tailscale (acesso remoto privado e DNS da rede)
- Driver NVIDIA 595 + NVIDIA Container Toolkit

## Serviços

Cada serviço tem sua pasta em [services/](services/README.md), com
`docker-compose.yml`, `.env.example` e, quando precisa, um passo a passo.

| Serviço | Para que serve | Porta |
|---|---|---|
| [AdGuard Home](services/adguard/) | Bloqueia propaganda e rastreadores em todos os aparelhos | 8080 |
| [Jellyfin](services/jellyfin/) | Filmes e séries, com transcodificação na GPU | 8096 |
| [Portainer](services/portainer/) | Painel para gerenciar os containers | 9443 |
| [code-server](services/code-server/) | VS Code no navegador para programar de qualquer lugar | 8443 |

### Decisões de segurança
- **Nenhuma porta aberta no modem.** Todo acesso externo passa pelo Tailscale.
- **Portas presas ao IP do Tailscale** (`${BIND_IP}:porta:porta`). O Docker
  ignora o `ufw`, então publicar só `porta:porta` deixaria o serviço aberto
  para toda a rede local.
- **Senhas fora do Git.** Cada serviço tem um `.env` (ignorado) e um
  `.env.example` (versionado, sem valores reais).
- **Chave SSH** para o servidor enviar ao GitHub, sem senha.

## Como recriar o servidor

```bash
git clone git@github.com:VnGn-dev/homelab-docs.git
cd homelab-docs/services/<servico>
cp .env.example .env   # preencher os valores
docker compose up -d
```

## Documentação

Formato de cada documento: **Problema → Causa → Solução → Comandos usados**.

- [ ] [Wi-Fi com Netplan](docs/01-wifi-netplan.md)
- [ ] [Discos e fstab](docs/02-discos-fstab.md)
- [ ] [Tailscale e SSH](docs/03-tailscale-ssh.md)
- [ ] [Docker e code-server](docs/04-docker-code-server.md)
- [x] [AdGuard Home](docs/05-adguard.md)
- [x] [Jellyfin e GPU no Docker](docs/06-jellyfin.md)
- [x] [GitHub com chave SSH](docs/07-github-ssh.md)

## Próximos passos
- [ ] Uptime Kuma: alertas no celular quando um serviço cair
- [ ] Backup automático dos dados para `/srv/backup` (Restic)
- [ ] Ollama + Open WebUI: IA local na GPU
- [ ] Sistema de gestão de equipamentos médicos (Python, FastAPI e PostgreSQL)
      rodando aqui, como projeto de desenvolvimento back-end
