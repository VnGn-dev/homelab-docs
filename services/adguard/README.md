# AdGuard Home

Bloqueia propaganda e rastreadores em todos os aparelhos que usam o Tailscale,
em casa ou na rua. Não precisa de acesso ao modem.

## Como funciona

Todo site que você abre começa com uma pergunta de DNS ("qual o IP de
google.com?"). O AdGuard responde essas perguntas e, quando o endereço é de
propaganda, responde "não existe". O anúncio simplesmente não carrega.

```
Celular ──(Tailscale)──> AdGuard no servidor ──> internet
                              │
                              └─ anúncio? bloqueia
```

## Passo a passo

### 1. Subir o container

```bash
cd services/adguard
cp .env.example .env     # coloque seu IP do Tailscale
docker compose up -d
```

### 2. Assistente de instalação

Abra `http://<IP-Tailscale>:3000` no navegador:

- **Interface web:** escolha "Todas as interfaces", porta **80**
  (por fora ela aparece como 8080, por causa do compose).
- **Servidor DNS:** "Todas as interfaces", porta **53**.
- Crie usuário e senha.

Depois disso, o painel fica em `http://<IP-Tailscale>:8080`.

### 3. O servidor não pode usar o próprio AdGuard pelo Tailscale

Senão ele fica num ciclo (ele pergunta para ele mesmo). Rode **no servidor**:

```bash
sudo tailscale set --accept-dns=false
```

### 4. Mandar os aparelhos usarem o AdGuard

No painel do Tailscale (login.tailscale.com) → **DNS**:

1. Em **Nameservers** → **Add nameserver** → **Custom** → coloque o IP do
   Tailscale do servidor.
2. Ative **Override DNS servers**.

Pronto: todo aparelho com Tailscale ligado passa a usar o AdGuard.

### 5. Testar

No celular, com o Tailscale ligado, abra um site cheio de propaganda. No painel
do AdGuard, a tela inicial mostra as consultas bloqueadas subindo.

## Se der problema

- **Internet parou nos aparelhos:** o servidor caiu ou o container parou.
  Desligue o Tailscale no aparelho para voltar ao normal e veja
  `docker compose logs`.
- **Algum app ou site parou de funcionar:** no painel, vá em
  **Log de consultas**, ache o domínio bloqueado e clique em **Desbloquear**.
- **Erro "address already in use" na porta 53:** só acontece se alguém trocar
  o compose para `53:53` sem o IP. Volte para `${BIND_IP}:53:53`.

## Listas de bloqueio recomendadas

Em **Filtros → Listas de bloqueio DNS → Adicionar lista**:

- AdGuard DNS filter (já vem ativa)
- HaGeZi's Pro Blocklist
