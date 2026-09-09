# Pairs Trading Screener, B3 (Brazilian Stock Exchange)
### n8n · brapi API · JavaScript · Statistical Arbitrage

I built this to stop eyeballing dozens of stock charts by hand looking for pairs
that had drifted apart. It pulls the prices on its own, does the statistics, and
hands me a ranked list of which pairs are stretched and worth a look. It's a
screening tool to support my own swing-trade analysis, not an automated trader
and not a prediction of the market.

🇧🇷 **Versão em português mais abaixo** ⬇️ · [Ir para o português](#-triagem-de-pairs-trading--b3-bolsa-brasileira)

<img width="1236" height="460" alt="arquitetura-n8n-trade-finanças" src="https://github.com/user-attachments/assets/881fd79e-0bd3-41de-ad0f-7af3be4d1118" />


---

## The problem

Pairs trading looks for two correlated stocks (usually same sector) that
normally move together. When they drift apart, the bet is that they'll converge
again. Watching that by hand across dozens of pairs every day is tedious and
easy to get wrong. I wanted the boring part (fetch, align, measure) done for me,
leaving only the decision to me.

## What it does

For each pair it pulls the price history, lines the two series up by date and
computes the price ratio over a rolling window. It then measures the current
**z-score** (how many standard deviations the pair sits from its own normal) and
flags anything past 2 standard deviations: which stock looks expensive, which
looks cheap, and the size of the gap. It also rates the signal, because not
every divergence is worth taking.

| Workflow file | What it is | Trigger | Status |
|---|---|---|---|
| [`workflows/pairs-trading-correlacao.json`](workflows/pairs-trading-correlacao.json) | Pairs screener + local memory | Manual | ✅ Working |
| [`workflows/funil-ativos-b3.json`](workflows/funil-ativos-b3.json) | Same engine, trend-based parameters | Manual | ✅ Working |

## The interesting engineering problem

The free tier of the price API (brapi) is rate-limited, so fetching every pair
fresh on every run hits the ceiling fast. I solved it with a **local memory
layer**: each run reads a CSV of everything fetched so far, merges in the new
candles without duplicating dates, and writes it back. Over time the tool builds
its own local price history and depends less on live calls. A plain cache that
turns a hard API limit into a non-issue.

## How it works

1. **Trigger** builds the list of stock pairs to check, grouped by sector
2. **Fetch** pulls candles from brapi one ticker at a time, with a pause between calls to stay under the rate limit
3. **Brain (Code node)** aligns the two price series by date and computes the ratio, rolling mean/std, z-score and signal quality
4. **Excel export** produces a ranked report
5. **Memory branch** reads the local CSV, merges the new candles and saves it back, so history accumulates across runs

<img width="1364" height="460" alt="excel-n8n-trade-correlação" src="https://github.com/user-attachments/assets/858c2964-78a4-423c-abd6-27cb66bd3ab7" />


The second variant scores single assets on a trend/convergence basis instead of pairs:

<img width="1351" height="466" alt="excel-n8n-trade" src="https://github.com/user-attachments/assets/e7fd41bd-6498-491e-bd4d-a141597ee52d" />


## Signal quality

A signal alone isn't enough, so each flagged pair is rated by how far it has
drifted. A **moderate** gap tends to revert (the useful case). An **extreme**
gap more often means the correlation itself broke, so those are marked to avoid
rather than to trade. This keeps the output honest instead of just spitting out
every outlier.

## Tech stack

- **n8n** for orchestration
- **brapi** REST API for B3 market data
- **JavaScript** (Code nodes) for the statistics: ratio, rolling mean/std, z-score
- **CSV / Excel** for local persistence and the output report

## A note on scope

This is a screening and data-organization tool. It is not financial advice and
not an automated trader. It highlights statistical divergences for a human to
review. Markets are noisy and correlations break, so a flagged pair is a
starting point for analysis, never a recommendation to trade.

## Running it yourself

> ⚠️ The brapi API token was replaced with a placeholder
> (`COLE_SEU_TOKEN_AQUI`). Add your own token to run it.

1. In n8n, use **Import from File** and pick a workflow from `workflows/`.
2. Paste your own brapi token where the placeholder is, in the Code node.
3. Adjust the pair list and thresholds if you want. Run it and read the Excel.

---
---

# 🇧🇷 Triagem de Pairs Trading, B3 (Bolsa Brasileira)
### n8n · API brapi · JavaScript · Arbitragem Estatística

Fiz esse sistema para parar de olhar dezenas de gráficos de ações na mão
procurando pares que se distanciaram. Ele busca os preços sozinho, faz a
estatística e me entrega uma lista ordenada de quais pares estão esticados e
valem uma olhada. É uma ferramenta de triagem pra apoiar a minha própria análise
de swing trade, não um robô que opera nem uma previsão do mercado.

<img width="1236" height="460" alt="arquitetura-n8n-trade-finanças" src="https://github.com/user-attachments/assets/43427280-0774-4c83-9a1a-3e6364903882" />


## O problema

Pairs trading procura duas ações correlacionadas (geralmente do mesmo setor) que
costumam andar juntas. Quando elas se distanciam, a aposta é que voltem a se
juntar. Acompanhar isso na mão em dezenas de pares todo dia é cansativo e fácil
de errar. Eu queria a parte chata (buscar, alinhar, medir) feita automaticamente,
sobrando só a decisão pra mim.

## O que ele faz

Pra cada par ele busca o histórico de preços, alinha as duas séries por data e
calcula a razão de preços numa janela móvel. Depois mede o **z-score** atual
(quantos desvios-padrão o par está do próprio normal) e sinaliza tudo que passa
de 2 desvios: qual ação está cara, qual está barata, e o tamanho da distância.
Também classifica a qualidade do sinal, porque nem toda divergência vale a pena.

| Arquivo do workflow | O que é | Gatilho | Status |
|---|---|---|---|
| [`workflows/pairs-trading-correlacao.json`](workflows/pairs-trading-correlacao.json) | Triagem de pares + memória local | Manual | ✅ Funcionando |
| [`workflows/funil-ativos-b3.json`](workflows/funil-ativos-b3.json) | Mesmo motor, parâmetros por tendência | Manual | ✅ Funcionando |

## O problema de engenharia interessante

O plano grátis da API de preços (brapi) tem limite de requisições, então buscar
todo par do zero a cada rodada estoura o limite rápido. Resolvi com uma **camada
de memória local**: cada rodada lê um CSV com tudo que já foi buscado, junta os
candles novos sem duplicar datas e salva de volta. Com o tempo, a ferramenta
monta o próprio histórico de preços e depende menos das chamadas ao vivo. Um
cache simples que transforma uma limitação rigida da API em algo irrelevante.

## Como funciona

1. **Gatilho** monta a lista de pares a checar, agrupados por setor
2. **Busca** puxa os candles da brapi um ticker por vez, com pausa entre as chamadas pra respeitar o limite
3. **Cérebro (nó Code)** alinha as duas séries por data e calcula razão, média/desvio móvel, z-score e qualidade do sinal
4. **Exporta Excel** gera um relatório ordenado
5. **Ramo de memória** lê o CSV local, junta os candles novos e salva de volta, acumulando histórico entre as rodadas

<img width="1364" height="460" alt="excel-n8n-trade-correlação" src="https://github.com/user-attachments/assets/a143e0a1-95e4-497c-9189-144e4d9ab48d" />


A segunda variação pontua ativos individuais por tendência/convergência, em vez de pares:

<img width="1351" height="466" alt="excel-n8n-trade" src="https://github.com/user-attachments/assets/3babc18c-dfc8-4c54-83b6-4c3d74d592ad" />


## Qualidade do sinal

Um sinal sozinho não basta, então cada par sinalizado é classificado pela
distância que percorreu. Uma distância **moderada** tende a reverter (o caso
útil). Uma distância **enorme** geralmente significa que a própria correlação
quebrou, então esses são marcados pra evitar, não pra operar. Isso mantém a
saída honesta, em vez de "cuspir" todo outlier.

## Stack

- **n8n** pra orquestração
- **API REST brapi** pros dados da B3
- **JavaScript** (nós Code) pra estatística: razão, média/desvio móvel, z-score
- **CSV / Excel** pra persistência local e o relatório de saída

## Sobre o escopo

Isso é uma ferramenta de triagem e organização de dados. Não é conselho
financeiro nem robô que opera sozinho. Ela destaca divergências estatísticas pra
um humano avaliar. O mercado é ruidoso e correlações quebram, então um par
sinalizado é ponto de partida pra análise, nunca recomendação de operar.

## Como rodar

> ⚠️ O token da API brapi foi trocado por um placeholder
> (`COLE_SEU_TOKEN_AQUI`). Coloque o seu pra rodar.

1. No n8n, use **Import from File** e escolha um workflow em `workflows/`.
2. Cole o seu token da brapi no lugar do placeholder, dentro do nó Code.
3. Ajuste a lista de pares e os limites se quiser. Rode e leia o Excel.

