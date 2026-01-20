# n8n Workflow — Meta Ads (Graph API) → Supabase (Campaign / Adset / Ad)

> Pipeline de dados de tráfego pago (Meta Ads) para Supabase, pronto para dashboard.

## Visão geral

Este workflow coleta dados de performance do **Meta Ads Insights** (Graph API) e salva no **Supabase** em uma tabela única contendo métricas diárias em três níveis:

- `campaign`
- `adset`
- `ad`

Cada linha representa **uma entidade (campanha OU adset OU ad)** em um determinado dia (`insight_date`).

---

## Objetivo

1. Buscar campanhas ativas do Meta Ads
2. Transformar campanhas em uma lista (1 item por campanha)
3. Para cada campanha:
   - buscar insights em nível de campanha
   - buscar insights em nível de adset
   - buscar insights em nível de ad
4. Normalizar tudo em um schema único
5. Unificar (merge append)
6. Inserir no Supabase

---

## Estrutura do fluxo (nodes)

1. **Manual Trigger** — When clicking “Execute workflow”
2. **default_data** — define intervalo de datas (`since`, `until`)
3. **Campanhas ativas** — lista campanhas ativas (Meta Graph API)
4. **Separa os dados3** — Split Out (campo `data`)
5. **Get Campaign** — Meta Insights (`level=campaign`)
6. **Normaliza campaign** — Code (JS)
7. **Get ad set** — Meta Insights (`level=adset`)
8. **Normaliza ad set** — Code (JS)
9. **Get ad** — Meta Insights (`level=ad`)
10. **Normaliza ad** — Code (JS)
11. **Merge** — append (3 inputs)
12. **Create a row** — Supabase insert

---

## Node: default_data

Define o range de datas usado no Meta Insights.

Exemplo:
```json
{
  "since": "2026-01-19",
  "until": "2026-01-19"
}
```

Usado em query params:
- `time_range[since] = {{ $items("default_data")[0].json.since }}`
- `time_range[until] = {{ $items("default_data")[0].json.until }}`

---

## Node: Separa os dados3 (Split Out)

Config:
- **Fields To Split Out:** `data`
- **Include:** `No Other Fields`

Resultado: 1 item por campanha, com campos como:
- `campaign_id`
- `campaign_name`
- `date_start`
- `date_stop`

---

## Node: Get Campaign (Meta Insights / Query Params)

### Query params
- `level` = `campaign`
- `fields` =
```
date_start,date_stop,campaign_id,campaign_name,adset_id,adset_name,ad_id,ad_name,spend,clicks,impressions,reach,cpc,ctr,cpm,actions,cost_per_action_type,objective,optimization_goal,frequency
```
- `limit` = `200`
- `time_range[since]` = `{{ $items("default_data")[0].json.since }}`
- `time_range[until]` = `{{ $items("default_data")[0].json.until }}`

📌 Não usar `filtering` aqui se o node já roda por campanha (via Split Out).

---

## Node: Merge

Modo: `append`

Inputs:
- Input 1: Normaliza campaign
- Input 2: Normaliza ad set
- Input 3: Normaliza ad

---

## Node: Create a row (Supabase)

Insere cada item normalizado como uma linha na tabela do Supabase.

Campos esperados no item:
- `source`, `insight_date`, `level`
- `campaign_id`, `campaign_name`
- `adset_id`, `adset_name`
- `ad_id`, `ad_name`
- métricas (spend, clicks, impressions, etc.)

---

## Erros comuns

### 1) Paired item data is unavailable
Causa: uso de `.item` em expressão sem pareamento válido.

Solução:
- usar `.first()`, `.last()` ou `$items('node')[0]`

### 2) (#100) param filtering must be an array
Causa: `filtering` enviado como string inválida.

Formato correto:
```json
[{"field":"campaign.id","operator":"EQUAL","value":"123"}]
```

---

## Resultado final

No Supabase, você terá uma tabela com linhas diárias por:
- campanha
- adset
- ad

Relacionamento via `campaign_id` permite drilldown no dashboard.
