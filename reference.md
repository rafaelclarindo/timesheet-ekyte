# API de apontamento

Descoberto no app ao abrir **Apontar horas** numa task. `GET /api/ticket-phases` só lista etapas. Quem grava é o POST abaixo.

Sessão: login em `https://app.ekyte.com/#/login` com `EKYTE_LOGIN` e `EKYTE_PASSWORD`. O header `Authorization: Bearer` sai da próxima XHR autenticada. Guardar o bearer só no `.env`, chave `EKYTE_BEARER`. Se a API responder 401, repetir o login e trocar o bearer.

## Leitura

- Task: `GET https://api.ekyte.com/api/v2/companies/{companyId}/ctc-tasks/{taskId}`
- Último apontamento do dia: `GET https://api.ekyte.com/api/companies/{companyId}/time-trackings/last-time-tracking?date={YYYY-MM-DD}T08:00:00`
- Etapas (não grava): `GET https://api.ekyte.com/api/ticket-phases`

`{companyId}` = `EKYTE_COMPANY_ID` da sessão. `{taskId}` sai do link da coluna A.

## Gravação

`POST https://api.ekyte.com/api/companies/{companyId}/workspaces/{workspaceId}/time-trackings`

`Content-Type: application/json`

`workspaceId`, `phaseId` e o objeto da task vêm do GET da task. `typeTimeTracking` 2 = manual. `effort` é minutos inteiros. `type` 1 = tarefa.

```json
{
  "typeTimeTracking": 2,
  "startDate": "YYYY-MM-DDTHH:mm:00",
  "startDateTime": "HH:mm",
  "endDate": "YYYY-MM-DDTHH:mm:00",
  "endDateTime": "HH:mm",
  "effort": 0,
  "comment": "",
  "workspaceId": 0,
  "type": 1,
  "ctcTaskId": 0,
  "ctcTask": {},
  "phaseId": 0,
  "phase": {},
  "ticket": {},
  "ticketPhase": {},
  "planning": {},
  "plan": {},
  "manualTime": "HH:mm"
}
```

`ctcTask` e `phase` são os objetos do GET, no mesmo formato que o app envia ao apontar. `startDate` começa no fim do último apontamento do dia. `endDate` = início + `effort` minutos.

Resposta de sucesso: HTTP 200 e o corpo é o id numérico do apontamento.
