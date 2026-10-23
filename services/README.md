# Serviços

Cada pasta é um serviço rodando em Docker Compose. Todos são acessados pelo
Tailscale, sem abrir porta no modem.

| Serviço | Para que serve | Endereço (via Tailscale) |
|---|---|---|
| code-server | VS Code no navegador | `https://<IP-Tailscale>:8443` |
| portainer | Gerenciar containers | `https://<IP-Tailscale>:9443` |
| adguard | Bloquear propaganda (DNS) | `http://<IP-Tailscale>:8080` |
| jellyfin | Filmes e séries | `http://<IP-Tailscale>:8096` |
| ollama | IA local (Open WebUI) | `http://<IP-Tailscale>:3001` |

Cada pasta tem um README com o passo a passo específico do serviço.

## Como subir um serviço

```bash
cd services/<nome-do-servico>
cp .env.example .env   # preencha os valores
docker compose up -d
```

## Configuração necessária (uma vez só)

Os serviços escutam no IP do Tailscale. Quando o servidor liga, o Docker às
vezes inicia **antes** do Tailscale ter o IP, e os containers falham com o erro
`cannot assign requested address`. Para evitar isso, permita que o sistema
reserve um IP que ainda não existe:

```bash
echo "net.ipv4.ip_nonlocal_bind=1" | sudo tee /etc/sysctl.d/99-tailscale-bind.conf
sudo sysctl --system
```

## Por que as portas usam o IP do Tailscale

O Docker cria regras de firewall próprias e **ignora o `ufw`**. Se a porta for
publicada como `8443:8443`, ela fica aberta para toda a rede. Usando
`${BIND_IP}:8443:8443` com o IP do Tailscale, o serviço só responde pela VPN.

## Regras

- O `.env` (com senhas) nunca vai para o Git; só o `.env.example`.
- Dados dos containers ficam fora do Git (veja o `.gitignore`).
