Meta Ads → Supabase → Dashboard (Lovable)

Este repositório contém tudo o que é necessário para montar um pipeline completo de dados de tráfego pago do Meta Ads (Facebook/Instagram), armazenar os dados no Supabase e gerar um dashboard automatizado no Lovable usando os dados do banco.

⸻

📦 O que existe neste repositório

1) ⚙️ Workflow do n8n (extração de dados da Meta)

<img width="1395" height="676" alt="Captura de Tela 2026-01-20 às 20 41 08" src="https://github.com/user-attachments/assets/24cb490f-784e-45e0-a8c4-20aa2e7d985b" />


Arquivo: workflow.json

Workflow no n8n que consulta a Meta Graph API / Insights e extrai métricas diárias em três níveis:
	•	Campaign
	•	Adset
	•	Ad

Esses dados são normalizados em um formato único e enviados para o Supabase.

📌 Objetivo: criar uma base estruturada e consistente para dashboards e análises.

⸻

2) 🗄️ SQL para gerar a tabela no Supabase

Arquivo: supabase.sql

Script SQL pronto para rodar no Supabase e criar a tabela que recebe os dados do workflow.

O schema foi pensado para:
	•	aceitar dados null (quando não se aplica)
	•	armazenar métricas diárias por nível (campaign | adset | ad)
	•	permitir drilldown e relacionamento via campaign_id
	•	facilitar exportação para BI e dashboards

⸻

3) 📊 PRD + Prompt para gerar Dashboard no Lovable

<img width="1369" height="647" alt="Captura de Tela 2026-01-20 às 20 42 24" src="https://github.com/user-attachments/assets/c41dd510-9122-4fae-b942-db75fd27ee82" />
<img width="1369" height="647" alt="Captura de Tela 2026-01-20 às 20 42 46" src="https://github.com/user-attachments/assets/d6c16699-d6cb-4375-80c9-b293ebd03ead" />
<img width="1369" height="647" alt="Captura de Tela 2026-01-20 às 20 43 00" src="https://github.com/user-attachments/assets/9b887688-1f34-4012-aa4a-a1bb3e73943c" />


Arquivos sugeridos:
	•	prd_dashboard.md (especificação do dashboard)
	•	prompt_lovable.md (prompt pronto para colar no Lovable)

Este material descreve:
	•	quais gráficos e KPIs o dashboard deve ter
	•	quais filtros precisam existir (data, campanha, adset, ad)
	•	como organizar páginas e visualizações
	•	como conectar o dashboard aos dados do Supabase

📌 Objetivo: acelerar a criação do dashboard com IA sem precisar desenhar tudo do zero.

⸻

🧱 Arquitetura (resumo)

Meta Ads (Graph API) → n8n → Supabase → Lovable Dashboard

O pipeline gera uma tabela única com linhas diárias para cada entidade (campanha, adset e ad), permitindo análises como:
	•	custo por lead (CPL)
	•	custo por mensagem (CTWA)
	•	performance por criativo (ads)
	•	performance por segmentação (adsets)
	•	performance consolidada por campanha

⸻

🗂️ Estrutura recomendada do repositório

/
├── README.md
├── workflow.json
├── supabase.sql
├── prd_dashboard.md
├── prompt_lovable.md
└── docs/
    └── images/
        ├── workflow.png
        ├── get-campaign.png
        └── supabase-table.png


⸻

🚀 Como usar

1) Criar a tabela no Supabase
	1.	Abra o painel do Supabase
	2.	Vá em SQL Editor
	3.	Cole o conteúdo do arquivo supabase.sql
	4.	Execute

⸻

2) Importar o workflow no n8n
	1.	Abra o n8n
	2.	Vá em Workflows → Import
	3.	Selecione workflow.json
	4.	Configure credenciais:
	•	Meta Graph API
	•	Supabase (API / URL)

⸻

3) Rodar o workflow

Execute manualmente ou agende via Cron no n8n.

O resultado será a inserção de linhas na tabela do Supabase com métricas diárias.

⸻

4) Criar o dashboard no Lovable
	1.	Abra o Lovable
	2.	Use o arquivo prd_dashboard.md como especificação
	3.	Cole o conteúdo do prompt_lovable.md
	4.	Conecte no Supabase usando as tabelas/queries recomendadas

⸻

📌 Observações importantes
	•	O workflow foi projetado para evitar duplicações e gerar 1 linha por entidade por dia
	•	O campo level define o nível do insight (campaign, adset, ad)
	•	O relacionamento principal do dataset é via campaign_id

⸻

📄 Licença

Uso livre para projetos internos. Ajuste conforme necessário.
