<div align="center">

# 🌦️🍕 Weather Food Recommendation

### Pipeline de Engenharia de Dados + IA no Databricks que recomenda comida de acordo com o clima

![Databricks](https://img.shields.io/badge/Databricks-Free%20Edition-FF3621?style=for-the-badge&logo=databricks&logoColor=white)
![PySpark](https://img.shields.io/badge/Python-PySpark-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Delta Lake](https://img.shields.io/badge/Delta%20Lake-Medallion-00ADD8?style=for-the-badge)
![IA](https://img.shields.io/badge/IA-LLM%20%2B%20Genie%20Agent-8A2BE2?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Conclu%C3%ADdo-2ea44f?style=for-the-badge)
![Custo](https://img.shields.io/badge/Custo-R%24%200-2ea44f?style=for-the-badge)

</div>

---

## 🎯 O problema de negócio

Em dias de **chuva** ou de **calor intenso**, apps de delivery recomendam comidas e restaurantes diferentes, usando clima, horário e localização do usuário. É um caso clássico de **personalização com IA**: o contexto muda o tempo todo, e a recomendação precisa acompanhar.

Só que IA boa depende de **dado bom**. Um modelo ou um agente não consegue recomendar nada sem dados confiáveis, atualizados e organizados. Este projeto resolve as duas pontas:

- 🏗️ **Engenharia de Dados:** ingestão de duas APIs abertas, tratamento com qualidade de dados, camadas Bronze, Silver e Gold e orquestração diária e semanal
- 🤖 **IA sobre dados governados:** um LLM que explica cada recomendação dentro do pipeline e um agente em linguagem natural, em que qualquer pessoa pergunta "o que recomendar agora na Av. Paulista?" e recebe uma resposta baseada nas tabelas Gold, e não em achismo de modelo

> 🌧️ Chuva e frio → sopas, caldos, massas e comfort food
> ☀️ Calor intenso → açaí, sorvete, saladas e comidas leves

💡 **A ideia central:** o pipeline entrega o dado pronto e confiável, e a IA trabalha em cima dele. Assim a resposta é rastreável até a tabela de origem, o que reduz o risco de a IA inventar informação.

---

arquitetura.png

---

### 🧱 Camadas (Arquitetura Medalhão)

| Camada | O que guarda | Tabelas |
|---|---|---|
| 🥉 **Bronze** | Resposta bruta das APIs, sem alteração | JSONs em Volume (`weather/`, `restaurants/` e `places/`) |
| 🥈 **Silver** | Dados limpos, tipados, deduplicados e validados | `weather_hourly`, `restaurants`, `places` e as tabelas `_rejected` |
| 🥇 **Gold** | Tags de clima, scores, recomendações, explicações e cadastro | `weather_context`, `dim_afinidade_clima_comida`, `recommendation_scores`, `recommendations_today`, `recommendations_explained`, `dim_restaurantes` |

---

## 🛠️ Stack (100% gratuita)

| Tema | Ferramenta |
|---|---|
| ☁️ Plataforma | Databricks Free Edition (serverless) |
| 🌤️ API de clima | [Open-Meteo](https://open-meteo.com/) (sem chave, previsão e histórico) |
| 🍽️ API de restaurantes e locais | [OpenStreetMap via Overpass API](https://overpass-api.de/) (sem chave) |
| 💾 Armazenamento | Delta Lake + Unity Catalog Volumes |
| ⚙️ Orquestração | Databricks Jobs (job diário com 6 tasks e job semanal com 3 tasks) |
| 🔔 Monitoramento | Alertas por e-mail (falha e duração) |
| ✅ Qualidade de dados | PySpark + regras próprias + tabelas de rejeitados |
| ✍️ IA generativa | `ai_query` em batch com `databricks-meta-llama-3-3-70b-instruct` |
| 💬 IA conversacional | Genie Agent sobre as tabelas Gold, com a função `recomendar_por_local` como ferramenta |
| 🔀 Versionamento | Git + GitHub (Git folders do Databricks) |

---

## 🗂️ Notebooks

```
Weather-Food-Recommendation/
├── 📓 03_setup_ambiente              → schemas bronze/silver/gold + Volume
├── 📓 04_bronze_clima                → previsão (7 dias) e histórico (30 dias)
├── 📓 05_bronze_restaurantes         → restaurantes de São Paulo (Overpass)
├── 📓 06_silver_clima                → limpeza, Data Quality e MERGE
├── 📓 07_silver_restaurantes         → classificação de comida, Data Quality e MERGE
├── 📓 08_gold_afinidade              → nota de afinidade clima x categoria
├── 📓 09_gold_weather_context        → tag de clima de cada hora
├── 📓 10_gold_recommendation_scores  → score base por restaurante e clima
├── 📓 11_gold_recommendations_today  → recomendações da hora atual por zona + função recomendar(lat, lon)
├── 📓 12_gold_genie_metadata         → descrições das tabelas para o Genie
├── 📓 13_ai_teste_llm                → teste de acesso aos modelos de LLM do workspace
├── 📓 14_ai_recomendacoes_explicadas → explicações em português geradas por LLM (ai_query)
├── 📓 15_ai_funcao_recomendar        → função SQL recomendar_por_local, ferramenta do Genie
├── 📓 16_bronze_locais               → bairros e estações de São Paulo (Overpass)
├── 📓 17_silver_locais               → limpeza, Data Quality e MERGE dos locais
├── 📓 18_gold_locais                 → dim_restaurantes: cadastro completo de restaurantes
└── 📄 README.md
```

---

## 🤖 IA no projeto

A camada de IA fica **em cima** das tabelas Gold. Ela não substitui o pipeline, ela depende dele.

| Componente | O que faz | Status |
|---|---|---|
| 🧠 **Contexto para a IA** | Descrições de tabelas e colunas gravadas no Unity Catalog, para que a IA entenda o significado de cada campo | ✅ Pronto |
| ✍️ **LLM no pipeline (`ai_query`)** | Escreve, em uma frase, por que cada restaurante combina com o clima de agora. Roda como task do job diário | ✅ Pronto |
| 🧰 **Função de recomendação** | `recomendar_por_local(lat, lon, n)` registrada no Unity Catalog. Recebe a posição do usuário e devolve os restaurantes do clima de agora, pesando a distância | ✅ Pronto |
| 💬 **Genie Agent** | Chat em linguagem natural sobre as tabelas Gold e a função de recomendação: a pergunta vira SQL, roda e volta como tabela ou gráfico | ✅ Pronto |

### ✍️ Como o LLM entra no pipeline

O notebook `14` seleciona as melhores recomendações de `recommendations_today` (até 3 por zona, no máximo 200 linhas por execução, para respeitar a cota) e chama o LLM em batch com `ai_query`. O resultado vira a tabela `recommendations_explained`.

- 🔒 O prompt usa **só os dados da linha** (clima, restaurante, categoria, bairro e se está aberto) e proíbe inventar prato, nota, preço, tempo de entrega ou distância
- 🧯 A chamada usa `failOnError => false`, então uma linha com erro não derruba o lote
- 🛡️ **Guarda de qualidade:** respostas vazias, longas demais ou que citem nota, avaliação, preço ou entrega são descartadas
- 📝 Linhas descartadas recebem uma explicação por **template**. A coluna `fonte_explicacao` mostra `llm` ou `template`, e `llm_endpoint` registra o modelo usado
- 🧱 Se nenhuma resposta do LLM for aproveitada, o notebook falha e **não sobrescreve** a última versão boa da tabela
- 🔁 O modelo fica numa única variável (`LLM_ENDPOINT`), então trocar de modelo é mudar uma linha

### 🧰 Função `recomendar_por_local`

O notebook `15` registra uma função SQL no Unity Catalog. A regra de negócio fica num lugar só, governada, e a IA **chama a função em vez de inventar o cálculo**.

- 📍 Entrada: latitude, longitude e quantidade de resultados
- 📐 Distância pela fórmula de Haversine, só restaurantes a até 3 km
- 🧮 `score_final = score_v1 × exp(-distância_km / 1,5)`
- ⚠️ **Limite conhecido:** a função lê `recommendations_today`, que guarda só os melhores de cada zona (grade de ~1 km). Os candidatos são os melhores das zonas próximas, e não todos os restaurantes do raio

### 🔎 Exemplos de perguntas para o agente

- "Qual é o clima agora e quais são as 5 melhores recomendações?"
- "Quais são os restaurantes mais recomendados perto da Av. Paulista?"
- "Quantos restaurantes temos por categoria de comida?"
- "Em quantas horas dos próximos 7 dias vai chover?"
- "Quais categorias combinam mais com calor intenso?"

### 🛡️ Como o projeto reduz o risco de respostas erradas da IA

- 📚 As tabelas têm descrições claras, inclusive avisos de uso (por exemplo, qual tabela usar para contar restaurantes)
- 🔒 A IA consulta **tabelas governadas** no Unity Catalog, com o SQL visível para conferência
- 🧰 O cálculo de distância e score fica numa função, e não no prompt
- 🛡️ As explicações do LLM passam por uma guarda de qualidade e têm fallback por template
- ⚠️ A IA só sabe o que está nos dados: o projeto declara abertamente o que é real, o que é hipótese e o que é simulado
- 🕐 Se o dado mais recente não é de hoje, o agente avisa em vez de inventar o clima

---

## 🧮 Como a recomendação funciona

As APIs abertas dizem **onde** o restaurante está e **que tipo de comida** ele serve, mas não dizem se ele é bom. Por isso o score é construído pelo próprio pipeline:

1. 🌡️ **Clima da hora vira uma tag:** `chuvoso`, `frio`, `ameno` ou `calor_intenso` (chuva tem prioridade sobre calor e frio)
2. 🍜 **Cada restaurante vira uma categoria:** 13 categorias + `geral`, definidas por cuisine, nome ou amenity (nessa ordem)
3. 💞 **Afinidade clima x categoria:** nota de 0 a 1 definida pelo projeto
4. 🕐 **Aberto agora:** parser do `opening_hours` do OpenStreetMap (fechado sai, desconhecido tem peso menor)
5. 📍 **Proximidade:** a função `recomendar_por_local` pondera pela distância do usuário
6. ✍️ **Explicação:** um LLM escreve uma frase ligando o clima ao restaurante recomendado

```
score_base  = afinidade × confiança da classificação
score_v1    = score_base × fator de horário
score_final = score_v1 × proximidade
```

---

## ✅ Qualidade de dados e boas práticas

- 🔁 **Idempotência:** Silver com `MERGE`, então rodar de novo não duplica dados
- 🚫 **Rejeitados separados:** registros inválidos vão para tabelas `_rejected`, com o motivo
- 🧪 **Validações no código:** `assert` nos notebooks Gold fazem a task falhar se algo estiver errado
- 📊 **Resultado atual:** 5.006 restaurantes únicos na Silver, sem duplicados e com 17,9% em `geral` (sem pista de comida no nome ou na cozinha)
- 🛡️ **IA com guarda de qualidade:** respostas do LLM fora do padrão viram template, e a tabela final só é sobrescrita se o LLM responder
- 🔔 **Alertas:** e-mail em caso de falha ou de duração acima do esperado

---

## ⚙️ Orquestração

**Job `Weather_food_daily`**, com 6 tasks em sequência:

```
Ingest_clima → silver_clima → gold_clima → gold_scores → gold_recomendacoes_hoje → ai_explicacoes
```

**Job `Weather_food_restaurants_weekly`**, com 3 tasks em sequência:

```
bronze_locais → silver_locais → gold_locais
```

- 🕖 Agendamento por cron, com fuso `America/Sao_Paulo`
- 🔄 Retry na ingestão, que depende de API externa
- 🍽️ Restaurantes e locais mudam pouco, então ficam num job semanal separado. O job diário sempre lê a versão mais recente

---

## 🗺️ Roadmap

### ✅ Concluído
- [x] Repositório integrado ao Databricks (Git folder)
- [x] Bronze de clima, de restaurantes e de locais
- [x] Silver com Data Quality, tabelas de rejeitados e `MERGE`
- [x] Gold: tags de clima, afinidade, scores, recomendações do dia e cadastro de restaurantes
- [x] Job diário (6 tasks) e job semanal (3 tasks), com agendamento e alertas por e-mail
- [x] Descrições das tabelas no Unity Catalog para o Genie
- [x] 🤖 LLM no pipeline (`ai_query`) gerando explicações, como task do job diário
- [x] 🤖 Função `recomendar_por_local` registrada no Unity Catalog
- [x] 🤖 Genie Agent integrado às tabelas Gold e à função de recomendação

### 🔭 Melhorias futuras
- [ ] 📈 **Machine Learning avançado (score V2):** treinar um modelo (scikit-learn + MLflow) que gere um `score_v2` ao lado do `score_v1`, separando treino e teste por restaurante
- [ ] 🛒 Pedidos simulados ou feedback real como alvo de treino do V2, para ele ir além de reproduzir a regra da V1
- [ ] 🧪 Avaliação do agente com perguntas de teste e resposta esperada (benchmarks)
- [ ] Task `12_gold_genie_metadata` no job, para reaplicar as descrições após cada regravação
- [ ] Dashboard (AI/BI)
- [ ] Demo em vídeo curto
- [ ] Mais cidades além de São Paulo

---

## ⚠️ Transparência sobre os dados

| Dado | Origem |
|---|---|
| 🌡️ Clima | **Real** (Open-Meteo) |
| 🍴 Restaurantes | **Real** (OpenStreetMap) |
| 🚇 Bairros e estações | **Real** (OpenStreetMap) |
| 💞 Afinidade clima x comida | **Hipótese do projeto**, não é dado observado |
| ✍️ Explicações das recomendações | **Geradas por LLM** a partir dos dados da linha, com fallback por template |

O OpenStreetMap não traz nota nem popularidade. Por isso o score é baseado em regras, e não em avaliação de clientes ou pedidos reais. Horário de funcionamento, tipo de cozinha e bairro também podem estar ausentes, e a Silver trata esses casos. O campo `bairro` vem do OpenStreetMap e pode estar escrito de formas diferentes, por isso a recomendação trabalha com coordenadas e zonas de ~1 km.

---

## 🔒 Limites do Databricks Free Edition

- 🚫 Uso não comercial (projeto de portfólio)
- ⏱️ Cota de compute: se estourar, o processamento pausa até renovar
- 🌐 Acesso à internet restrito a domínios permitidos (Open-Meteo e Overpass funcionam)
- 🖥️ Um único SQL warehouse pequeno, então evito rodar o job e o Genie ao mesmo tempo
- ✍️ O LLM roda em lote com limite de 200 linhas por execução, para respeitar a cota
- 📍 Escopo atual: **uma cidade (São Paulo)**

---

## 🚀 Como reproduzir

1. 🍴 Faça um fork ou clone este repositório
2. ☁️ Crie uma conta no [Databricks Free Edition](https://www.databricks.com/learn/free-edition)
3. 🔗 Em **Settings → Linked accounts**, conecte o GitHub e instale o app no repositório
4. 📂 Em **Workspace → Create → Git folder**, clone o repositório
5. ▶️ Execute os notebooks na ordem numérica (03 a 18). Antes do `14`, rode o `13` e ajuste `LLM_ENDPOINT` para um modelo disponível no seu workspace
6. ⚙️ Crie o job diário com as 6 tasks e o job semanal com as 3 tasks, na ordem da seção de orquestração
7. 💬 Crie um Genie Agent apontando para as tabelas Gold e adicione a função `recomendar_por_local` como ferramenta. O agente não vai no Git folder, então precisa ser recriado no workspace

---

## 👤 Autor

**Felipe Banciella**
🔗 [github.com/banciella](https://github.com/banciella)

---

<div align="center">

⭐ Se curtiu o projeto, deixa uma estrela!

</div>
