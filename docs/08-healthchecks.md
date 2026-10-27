# Aviso quando o servidor cai (Healthchecks.io)

Monitoramento externo: se o servidor desligar, travar ou perder a internet,
recebo um e-mail no celular.

---

## Problema: como saber que o servidor caiu se ele mesmo caiu?

### Causa
Um monitor rodando no próprio servidor (como o Uptime Kuma) desliga junto com
ele e não consegue avisar ninguém. E um serviço de fora também não consegue
testar o servidor, porque ele só aceita conexões pelo Tailscale.

### Solução
Inverter a lógica: **o servidor avisa que está vivo**. A cada 5 minutos ele
acessa um endereço do Healthchecks.io. Se o aviso parar de chegar, o
Healthchecks manda um e-mail.

```
servidor ──"estou vivo" a cada 5 min──> Healthchecks.io
                                              │
                         parou de chegar? ────┴──> e-mail no celular
```

Configuração no Healthchecks.io (plano gratuito):
- Check `homelab`
- **Period:** 5 minutos (de quanto em quanto tempo o sinal chega)
- **Grace Time:** 10 minutos (tolerância, para não dar alarme falso se a
  internet piscar)
- Alerta por e-mail (já vem configurado com o e-mail da conta)

Escolhi e-mail em vez de Telegram porque não preciso instalar nenhum app no
celular.

### Comandos usados

Teste manual (o check fica verde no painel):
```bash
curl -fsS -m 10 --retry 5 -o /dev/null https://hc-ping.com/<meu-codigo> && echo "Sinal enviado!"
```

Agendamento com o cron (`crontab -e`):
```
*/5 * * * * curl -fsS -m 10 --retry 5 -o /dev/null https://hc-ping.com/<meu-codigo>
```

| Parte | O que faz |
|---|---|
| `*/5 * * * *` | a cada 5 minutos, todo dia |
| `-fsS` | quieto se der certo, mostra erro se falhar |
| `-m 10` | desiste depois de 10 segundos |
| `--retry 5` | tenta até 5 vezes |
| `-o /dev/null` | descarta a resposta do site |

Teste do alerta sem derrubar o servidor: **Integrations → Test!** no
Healthchecks.

---

## Aprendizados
- O endereço de ping é um segredo: quem tiver ele consegue fingir que o
  servidor está vivo. Ele fica só no `crontab` do servidor, nunca no Git.
- `crontab -l` mostra os agendamentos, `crontab -e` edita.
- Para ver todos os agendamentos de outro jeito: o cron é o "despertador" do
  Linux, e cada linha é um comando com hora marcada.
