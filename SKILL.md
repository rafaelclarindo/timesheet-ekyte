---
name: timesheet-ekyte
description: >-
  Sincroniza as tasks da Thainá Gomes no Ekyte com a planilha de timesheet,
  uma aba nova por dia, atualizando o link a cada 2 horas, e lança o
  apontamento quando pedirem para subir o timesheet. Use quando pedirem
  timesheet, apontamento de horas, planilha da Thainá, subir timesheet,
  sincronizar tasks do Ekyte, ou onboarding do .env e do token MCP do Ekyte.
---

# Timesheet Ekyte — Thainá Gomes

Planilha: [Invictus Thainá Gomes - Timesheet](https://docs.google.com/spreadsheets/d/1V_aTfL9Df7iQx3GI1OMK1tQW-cAbWWaA_fxNJBxACNY/edit).

Contrato da API de gravação: [reference.md](reference.md).

## Segredos

Login, senha, executor e token MCP não entram nesta skill, no status, em commit nem em exemplo. Em computador novo, pedir e gravar só em `projects/ekyte-invictus/.env` (gitignored). Não reaproveitar credencial de outro usuário, de chat antigo ou de `.mcp.json` alheio.

## Onboarding

Rodar antes de sync ou lançamento se faltar qualquer chave:

1. `EKYTE_LOGIN` — e-mail de quem executa as tasks desta planilha (Thainá).
2. `EKYTE_PASSWORD`
3. `EKYTE_MCP_TOKEN` — token privado do MCP `https://api.ekyte.com/mcp`. Gravar no `.env`. No `.mcp.json` local, o servidor `ekyte` usa essa URL com o token. Não ecoar o token na resposta.
4. Entrar em `https://app.ekyte.com/#/login`. Ler `companyId` da sessão e o id do usuário logado. Gravar `EKYTE_COMPANY_ID` e `EKYTE_EXECUTOR_ID`.
5. Confirmar que o executor é a pessoa da planilha. Se o login for de outra pessoa, parar. Não escrever as tasks dela na planilha da Thainá.

Chaves vazias: `projects/ekyte-invictus/.env.example`.

## Aba do dia

Uma aba nova por dia. Nome: `DD/MM/AAAA` (ex.: `07/10/2026`).

Cabeçalho, igual à planilha:

| A | B | C |
|---|---|---|
| Task do Projeto | Horas | Minutos |

- A: link `https://app.ekyte.com/#/tasks/list/{id}/edit`
- B: horas, preenchidas pela Thainá. Não sobrescrever.
- C: minutos, preenchidos pela Thainá. Não sobrescrever.

Se a aba de hoje não existir, criar só com o cabeçalho e os links. Se existir, atualizar.

## Sync — a cada 2 horas

Tasks: executor = `EKYTE_EXECUTOR_ID`, situação em andamento, prazo de etapa até o dia da aba. Fonte: MCP `ekyte`, `list_tasks`. Se o filtro não vier na tool, filtrar o retorno.

- Link novo: primeira linha vazia da aba de hoje.
- Link que já está na aba: manter B e C.
- Link que saiu da fila e B e C estão vazios: apagar a linha.
- Link que saiu da fila e B ou C tem valor: manter a linha.

Na primeira sync do dia nesta sessão, repetir só este sync a cada 2 horas. O lançamento não entra nesse ciclo.

## Subir o timesheet

Só quando pedirem para subir, lançar ou apontar o timesheet. Nunca no ciclo de 2 horas.

1. Sync da aba de hoje.
2. Ler a aba de hoje. Para cada linha com hora ou minuto maior que zero: `effort = horas * 60 + minutos`. Hora vazia vale 0. Minuto vazio vale 0.
3. Encadear os horários a partir do fim do último apontamento do dia (`last-time-tracking` em [reference.md](reference.md)). Sem sobreposição.
4. Gravar cada linha com o POST de [reference.md](reference.md). Sucesso = HTTP 200 e id numérico.
5. Se uma linha falhar, parar. Dizer a linha, o link e o erro. Não marcar as outras como feitas.
6. Responder com link, minutos e id de cada apontamento gravado.

Não lançar linha zerada. Não lançar de novo a mesma task no mesmo dia com o mesmo `effort` se o apontamento desse dia já existir.
