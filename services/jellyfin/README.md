# Jellyfin

Sua própria "Netflix": filmes e séries guardados no HD, assistidos no celular,
notebook ou TV. A GTX 1650 converte os vídeos em tempo real quando o aparelho
não suporta o formato original (isso se chama **transcodificação**).

## Passo a passo

### 1. Organizar as pastas no HD

O Jellyfin reconhece capa, sinopse e episódios sozinho se os nomes seguirem
este padrão:

```
/mnt/hd/midia/
├── filmes/
│   └── Nome do Filme (2010)/
│       └── Nome do Filme (2010).mkv
└── series/
    └── Nome da Série/
        └── Season 01/
            └── Nome da Série S01E01.mkv
```

### 2. Preparar a GPU (uma vez só, serve também para o Ollama)

```bash
# Driver NVIDIA
sudo ubuntu-drivers install
sudo reboot

# Depois de reiniciar, conferir se a placa aparece:
nvidia-smi
```

Depois, instale o **NVIDIA Container Toolkit**, que deixa o Docker usar a GPU.
Siga a seção "Installing with Apt" da documentação oficial:
https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html

E por fim:

```bash
sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# Teste: deve mostrar a GTX 1650
docker run --rm --gpus all ubuntu nvidia-smi
```

### 3. Subir o container

```bash
cd services/jellyfin
cp .env.example .env     # ajuste MEDIA_DIR e BIND_IP
mkdir -p config cache   # cria as pastas antes, como seu usuário
docker compose up -d
```

Abra `http://<IP-Tailscale>:8096` e siga o assistente: crie o usuário e
adicione as bibliotecas apontando para `/media/filmes` e `/media/series`.

### 4. Ligar a GPU dentro do Jellyfin

**Painel → Reprodução → Transcodificação**:

- **Aceleração por hardware:** NVIDIA NVENC
- Marque a decodificação para **H264**, **HEVC**, **MPEG2**, **VC1** e **VP9**
- Salve

Para conferir: assista algo no celular com qualidade baixa forçada e rode
`nvidia-smi` no servidor. O processo do Jellyfin deve aparecer usando a placa.

## Assistir

- **Celular e notebook:** app oficial do Jellyfin, endereço
  `http://<IP-Tailscale>:8096` (com o Tailscale ligado).
- **TV:** se a TV não tem Tailscale, é preciso liberar o acesso pela rede de
  casa. Fica para uma próxima etapa.

## Se der problema

- **Container reiniciando ("Restarting (139)"):** as pastas `config` e `cache`
  foram criadas pelo Docker como root. Corrija com
  `sudo chown -R $USER:$USER config cache` e rode `docker compose restart`.
- **"could not select device driver nvidia":** o container toolkit não está
  instalado ou o Docker não foi reiniciado (passo 2).
- **Biblioteca vazia:** confira se `MEDIA_DIR` está certo e se seu usuário
  consegue ler a pasta: `ls /mnt/hd/midia`.
- **Vídeo travando:** veja em **Painel → Atividade** se está transcodificando.
  Se estiver e a GPU não aparece no `nvidia-smi`, revise o passo 4.
