# Ollama + Open WebUI

IA local: os modelos rodam na GTX 1650 do servidor, sem mandar nada para a
internet.

- **Ollama:** baixa e roda os modelos. Tem uma API (porta 11434) que outros
  programas podem usar.
- **Open WebUI:** interface parecida com o ChatGPT, no navegador.

```
Navegador ──> Open WebUI (3001) ──> Ollama (11434) ──> GTX 1650
Projeto Python ─────────────────────┘
```

## Pré-requisito

GPU configurada para o Docker (driver NVIDIA + NVIDIA Container Toolkit).
Veja o passo 2 do [README do Jellyfin](../jellyfin/README.md).

## Passo a passo

### 1. Criar as pastas e o .env

```bash
cd services/ollama
sudo mkdir -p /srv/ssd/ollama /srv/ssd/open-webui
cp .env.example .env     # coloque o IP do Tailscale
```

### 2. Subir

```bash
docker compose up -d
```

A imagem do Open WebUI é grande (alguns GB), a primeira vez demora.

### 3. Baixar um modelo

A GTX 1650 tem 4 GB de memória, então os modelos precisam ser pequenos
(até uns 4 bilhões de parâmetros, o "3b" ou "4b" no nome).

```bash
docker exec ollama ollama pull llama3.2:3b
```

Outras opções boas: veja em https://ollama.com/library modelos com
**3b** ou **4b** no nome. Modelo maior que a memória da placa roda, mas
fica lento, porque parte vai para o processador.

### 4. Criar a conta no Open WebUI

Abra `http://<IP-Tailscale>:3001`. **A primeira conta criada vira
administradora**, então crie logo depois de subir.

### 5. Conferir se está usando a GPU

Faça uma pergunta no Open WebUI e, ao mesmo tempo, no servidor:

```bash
nvidia-smi
```

O processo `ollama` deve aparecer usando a memória da placa.

## Comandos úteis

```bash
docker exec ollama ollama list          # modelos baixados
docker exec ollama ollama ps            # modelo carregado e se está na GPU
docker exec ollama ollama rm <modelo>   # apagar um modelo
```

## Se der problema

- **Resposta muito lenta:** rode `docker exec ollama ollama ps`. Se a coluna
  PROCESSOR mostrar CPU em vez de 100% GPU, o modelo é grande demais para a
  placa. Use um menor.
- **Open WebUI sem modelos na lista:** nenhum modelo foi baixado ainda
  (passo 3), ou recarregue a página.
