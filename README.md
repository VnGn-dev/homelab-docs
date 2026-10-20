# Homelab

Servidor doméstico para estudos e portfólio.

## Hardware
- CPU: Intel i5-9400F
- RAM: 16 GB
- GPU: NVIDIA GTX 1650 (4 GB)
- Armazenamento: 1 SSD (sistema) + 1 SSD (Docker/projetos) + 2 HDs (dados e backup)
- Rede: Wi-Fi (Realtek RTL8821AE) + Ethernet

## Software
- Ubuntu Server 26.04 LTS
- Docker
- Tailscale (acesso remoto privado, SSH por chave)
- code-server (VS Code no navegador)
- Ollama (IA local, em andamento)

## Serviços
Veja [services/](services/README.md): cada serviço tem seu `docker-compose.yml` e um `.env.example`.

## Documentação
- [ ] Wi-Fi com Netplan
- [ ] Discos e fstab
- [ ] Tailscale e SSH
- [ ] Docker e code-server
