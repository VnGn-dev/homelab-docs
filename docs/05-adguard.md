# AdGuard Home

Bloqueador de propaganda e rastreadores por DNS, rodando em Docker e usado por
todos os meus aparelhos através do Tailscale, sem precisar mexer no modem
(não tenho acesso a ele).

- Painel: `http://<IP-Tailscale>:8080`
- Configuração: `services/adguard/`

---

## Problema 1: sem acesso ao modem para configurar o DNS da casa

### Causa
O jeito comum é trocar o DNS no modem, mas o modem é da operadora e só o
técnico tem acesso.

### Solução
Usar o próprio Tailscale como "distribuidor" do DNS. No painel do Tailscale,
coloquei o IP do servidor como nameserver global e ativei
**Override DNS servers**. Todo aparelho com Tailscale ligado passa a usar o
AdGuard, em casa ou na rua.

### Comandos usados
Nenhum. Configurado em `login.tailscale.com/admin/dns`:
- Nameservers → Add nameserver → Custom → IP do servidor no Tailscale
- Override DNS servers: ativado

---

## Problema 2: porta 53 já usada pelo Ubuntu

### Causa
O Ubuntu já tem um serviço de DNS próprio (`systemd-resolved`) usando a
porta 53. Publicar a porta como `53:53` (em todas as interfaces) dá conflito.

### Solução
Publicar a porta só no IP do Tailscale. O `systemd-resolved` escuta em
`127.0.0.53`, então os dois não se encontram e não precisei desligar nada.

### Comandos usados
No `docker-compose.yml`:
```yaml
ports:
  - "${BIND_IP}:53:53/tcp"
  - "${BIND_IP}:53:53/udp"
```

---

## Problema 3: servidor perguntando o DNS para ele mesmo

### Causa
Com o Tailscale distribuindo o AdGuard como DNS, o próprio servidor também
passaria a usar ele, e ficaria num ciclo.

### Solução
Dizer para o Tailscale do servidor não aceitar o DNS distribuído.

### Comandos usados
```bash
sudo tailscale set --accept-dns=false
```

---

## Problema 4: containers não sobem quando o servidor reinicia

### Causa
Os serviços escutam no IP do Tailscale. Na inicialização, o Docker pode ligar
antes de o Tailscale ter o IP, e os containers falham com
`cannot assign requested address`.

### Solução
Permitir que o sistema reserve um IP que ainda não existe.

### Comandos usados
```bash
echo "net.ipv4.ip_nonlocal_bind=1" | sudo tee /etc/sysctl.d/99-tailscale-bind.conf
sudo sysctl --system
```

---

## Problema 5: propaganda do YouTube continua aparecendo

### Causa
Limitação de todo bloqueador por DNS: o YouTube entrega os anúncios pelo
mesmo endereço dos vídeos. Bloquear um bloqueia o outro.

### Solução
Para YouTube, usar bloqueador no navegador (uBlock Origin, uBlock Origin Lite
ou AdGuard for Safari). O AdGuard do servidor continua bloqueando o resto.

### Comandos usados
Nenhum.

---

## Aprendizados
- O IP que aparece no assistente do AdGuard (`172.x.x.x`) é o de dentro do
  container. O que vale para mim é o IP do Tailscale.
- Confirmar qual IP é de qual máquina: `tailscale ip -4` no servidor. Eu
  estava usando o IP do Mac por engano.
- Se a internet de um aparelho parar, provavelmente o servidor caiu:
  desligar o Tailscale no aparelho resolve na hora.
