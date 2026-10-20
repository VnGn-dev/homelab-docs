# GitHub com chave SSH no servidor

Repositório clonado no servidor em `~/homelab-docs`, enviando para o GitHub
sem senha, usando chave SSH.

---

## Problema: `git push` pedindo usuário e senha

### Causa
O repositório foi clonado por HTTPS. O GitHub não aceita mais a senha da
conta pelo terminal.

### Solução
Criar uma chave SSH no servidor, cadastrar a parte pública no GitHub e trocar
o endereço do repositório de HTTPS para SSH.

- O arquivo que termina em `.pub` é a parte pública: pode ser compartilhado.
- O arquivo sem `.pub` (`id_ed25519`) é a parte secreta: nunca sai do
  servidor.

### Comandos usados
```bash
# Identidade do Git (a mesma dos commits anteriores: git log -1)
git config --global user.name "VnGn"
git config --global user.email "vinicius.goncalves.dev@gmail.com"

# Criar a chave (Enter em todas as perguntas)
ssh-keygen -t ed25519 -C "vinicius.goncalves.dev@gmail.com"

# Mostrar a parte pública para copiar
cat ~/.ssh/id_ed25519.pub
```
No GitHub: `github.com/settings/keys` → New SSH key → título `homelab` →
colar a chave.

```bash
# Testar (responder "yes" na primeira vez)
ssh -T git@github.com

# Trocar o repositório para SSH e enviar
git remote set-url origin git@github.com:VnGn-dev/homelab-docs.git
git push
```

---

## Rotina para salvar mudanças

```bash
git status                  # o que mudou
git add --dry-run .         # simula o que entraria (conferir que não tem .env)
git add .
git commit -m "o que mudou"
git push
```

## Aprendizados
- Antes de adicionar, conferir com `git add --dry-run .` que nenhum `.env`
  (com IPs e senhas) vai junto. O `.gitignore` protege, mas vale olhar.
