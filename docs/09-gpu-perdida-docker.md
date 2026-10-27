# Containers perdendo acesso à placa de vídeo

Problema que apareceu alguns dias depois de configurar a GPU: o Jellyfin e o
Ollama pararam de funcionar sem eu ter mexido em nada.

---

## Problema: filme carregando infinito e IA travada

Fui mostrar um filme no Jellyfin para um amigo e ele ficou carregando sem
parar. No mesmo dia, o Open WebUI tinha travado.

### Diagnóstico

1. A placa estava livre no servidor (nada usando a memória):
   ```bash
   nvidia-smi
   ```
2. Mas, de dentro do container, a placa não aparecia:
   ```bash
   docker exec jellyfin nvidia-smi
   # Failed to initialize NVML: Unknown Error
   ```

Ou seja: o driver funcionava no servidor, mas os containers tinham ficado
"cegos". O Jellyfin tentava converter o vídeo na GPU, não conseguia, e
travava. O Ollama, que usa a mesma placa, também.

### Causa
Problema conhecido da NVIDIA com Docker. Algumas atualizações automáticas do
Ubuntu recarregam o systemd (`systemctl daemon-reload`). Com o modo padrão do
Docker (cgroup driver `systemd`), esse recarregamento apaga as permissões que
deixavam os containers acessarem a placa.

### Solução

**Na hora:** recriar os containers. Filmes, configurações e modelos não se
perdem, porque ficam em pastas do servidor.
```bash
cd ~/homelab-docs/services/jellyfin && docker compose up -d --force-recreate
cd ~/homelab-docs/services/ollama && docker compose up -d --force-recreate
```

**Definitiva:** trocar o cgroup driver do Docker para `cgroupfs`, que mantém
as permissões quando o systemd recarrega.

### Comandos usados

```bash
# Cópia de segurança da configuração do Docker
sudo cp /etc/docker/daemon.json /etc/docker/daemon.json.bakk

# Adicionar a opção sem risco de quebrar o JSON
sudo python3 -c 'import json; p="/etc/docker/daemon.json"; d=json.load(open(p)); d["exec-opts"]=["native.cgroupdriver=cgroupfs"]; json.dump(d, open(p,"w"), indent=4)'

# Aplicar e conferir
sudo systemctl restart docker
docker info | grep -i "cgroup driver"     # Cgroup Driver: cgroupfs
```

Como ficou o `/etc/docker/daemon.json`:
```json
{
    "data-root": "/srv/ssd/docker",
    "runtimes": {
        "nvidia": {
            "args": [],
            "path": "nvidia-container-runtime"
        }
    },
    "exec-opts": [
        "native.cgroupdriver=cgroupfs"
    ]
}
```

### Prova de que resolveu
Simulei a causa e conferi que o container continua vendo a placa:
```bash
sudo systemctl daemon-reload
docker exec jellyfin nvidia-smi     # a tabela da GTX 1650 aparece
```

---

## Aprendizados
- Quando algo funciona no servidor mas não no container, comparar os dois
  lados (`nvidia-smi` fora e `docker exec ... nvidia-smi` dentro) mostra onde
  está o problema.
- Uma correção só está confirmada quando eu consigo **reproduzir a causa** e
  ver que o problema não volta.
- Antes de editar um arquivo de configuração do sistema, fazer cópia de
  segurança. Para JSON, editar com uma ferramenta (Python) é mais seguro do que
  à mão.
