# Jellyfin

Servidor de filmes e séries, com a GTX 1650 convertendo os vídeos
(transcodificação) quando o aparelho não suporta o formato original.

- Endereço: `http://<IP-Tailscale>:8096`
- Mídia: `/srv/dados/midia` (no container: `/media`)
- Configuração: `services/jellyfin/`

---

## Problema 1: Docker não conseguia usar a placa de vídeo

### Causa
O driver da NVIDIA já estava instalado (`nvidia-smi` funcionava), mas faltava
o **NVIDIA Container Toolkit**, que é a ponte entre o Docker e a GPU.
`nvidia-ctk --version` dava `command not found`.

### Solução
Adicionar o repositório da NVIDIA, instalar o toolkit e configurar o Docker.

### Comandos usados
```bash
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt update
sudo apt install -y nvidia-container-toolkit
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# Teste: a tabela da GTX 1650 aparece de dentro de um container
docker run --rm --gpus all ubuntu nvidia-smi
```

---

## Problema 2: erro genérico no assistente de instalação

"Ocorreu um erro ao processar o requisito. Por favor, tente novamente mais
tarde."

### Causa
Nos logs apareciam `forbidden` e `Token is required`. O container tinha
reiniciado no meio do assistente e o navegador ficou com uma versão antiga da
página.

### Solução
Como ainda não tinha nada configurado, apaguei a configuração e comecei do
zero, abrindo a página numa janela privada.

### Comandos usados
```bash
docker compose logs --tail 30     # ver o motivo real do erro
docker compose down
sudo rm -rf config cache
docker compose up -d
```

---

## Problema 3: container reiniciando sem parar ("Restarting (139)")

### Causa
Ao apagar as pastas `config` e `cache`, o Docker criou de novo **como root**.
O Jellyfin roda como meu usuário (1000) e não conseguia escrever nelas.

### Solução
Passar as pastas para o meu usuário. Para não acontecer de novo, criar as
pastas antes de subir o container.

### Comandos usados
```bash
docker compose down
sudo chown -R vinigon:vinigon config cache
docker compose up -d

# Prevenção (antes do primeiro "up"):
mkdir -p config cache
```

---

## Problema 4: filme copiado não aparecia no Jellyfin

### Causa
O arquivo estava certo no servidor e o container enxergava ele, mas a
biblioteca ainda não tinha sido escaneada.

### Solução
Conferir os dois lados e escanear a biblioteca.

### Comandos usados
```bash
ls -laR /srv/dados/midia/filmes                 # visto pelo servidor
docker exec jellyfin ls -laR /media/filmes      # visto pelo container
```
No Jellyfin: Painel → Bibliotecas → ⋮ no card Filmes → Escanear biblioteca.

---

## Problema 5: "No such file or directory" ao copiar o filme

### Causa
Rodei o `scp` no terminal do **servidor**, que procurou o arquivo na pasta
Downloads dele. O filme estava no **Mac**.

### Solução
O `scp` roda no computador **onde o arquivo está**. Conferir o começo da linha
do terminal: `vinigon@homelab` é o servidor, `teste@MacBook...` é o Mac.

### Comandos usados
No Mac:
```bash
scp ~/Downloads/Nome*.mkv vinigon@<IP-Tailscale>:/srv/dados/midia/filmes/
ssh vinigon@<IP-Tailscale> 'cd /srv/dados/midia/filmes && mkdir -p "Nome do Filme (Ano)" && mv Nome*.mkv "Nome do Filme (Ano)/"'
```

---

## Aprendizados
- Pastas criadas automaticamente pelo Docker nascem como root.
- Mensagem de erro genérica na tela: o motivo real está em
  `docker compose logs`.
- Caminho no Jellyfin é o de dentro do container (`/media/...`), não o do
  servidor.
- Padrão de nomes que o Jellyfin reconhece:
  `filmes/Nome do Filme (Ano)/Nome do Filme (Ano).mkv`
- Transferência entre Mac e servidor é rápida porque o Tailscale conecta os
  dois direto pela rede de casa.
