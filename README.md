# Automated-Pairs-Trading-Screener-B3-Brazilian-Stock-Exchange-

n8n · brapi API · JavaScript · Statistical Arbitrage

A data-automation tool that screens pairs of correlated Brazilian stocks and flags statistical divergences that may signal a mean-reversion opportunity. Built to support swing-trade analysis, not to place trades or predict the market.

What it does

It automates the full screening pipeline: it pulls historical prices for pairs of stocks in the same sector, measures how far each pair has drifted from its normal relationship, and outputs a ranked, readable report.

For each pair, it computes the price ratio over a rolling window, its mean and standard deviation, and the current z-score (how many standard deviations the pair is from normal). When a pair stretches beyond a threshold (2 std), it's flagged: which stock is "expensive", which is "cheap", and the direction of a potential mean-reversion trade. It also classifies signal quality — a moderate divergence tends to revert, while an extreme one more likely means the correlation itself broke, so those are marked to avoid.

The interesting engineering problem

The free tier of the price API (brapi) is rate-limited. Fetching every pair fresh on every run hits that limit fast. I solved it with a local memory layer: each run reads a CSV of everything fetched so far, merges in the new candles without duplicating dates, and writes it back. Over time the tool builds its own local price history and leans less on live calls — a simple cache that turns a hard API limit into a non-issue.

How it works (pipeline)
Trigger → builds the list of stock pairs to check (grouped by sector)
Fetch → pulls candles from brapi, one ticker at a time, with a pause between calls to stay under the rate limit
Brain (Code node) → aligns the two price series by date, computes the ratio, z-score and signal quality for each pair
Excel export → a ranked report: pair, sector, z-score, signal, which to buy/sell, prices, distance %
Memory branch → reads local CSV → merges new candles → saves back, so history accumulates across runs
Tech stack
n8n for orchestration
brapi REST API for B3 market data
JavaScript (Code nodes) for the statistics: ratio, rolling mean/std, z-score
CSV/Excel for local persistence and the output report
A note on scope

This is a screening and data-organization tool, not financial advice and not an automated trader. It highlights statistical divergences for a human to review. Markets are noisy and correlations break; a flagged pair is a starting point for analysis, never a recommendation to trade.

⚠️ The brapi API token was replaced with a placeholder (COLE_SEU_TOKEN_AQUI). Add your own token to run it.

------------------------------------------------------------------------------------------------------------------------------------------

# Ferramenta Automatizada de Triagem de Pairs Trading (B3 - Bolsa de Valores Brasileira)

n8n · API brapi · JavaScript · Arbitragem Estatística

Uma ferramenta de automação de dados que realiza a triagem de pares de ações brasileiras correlacionadas e identifica divergências estatísticas que podem sinalizar uma oportunidade de reversão à média. Desenvolvida para apoiar a análise de *swing trade*, e não para executar operações ou prever o mercado.

O que ela faz

Automatiza todo o fluxo de triagem: coleta preços históricos de pares de ações do mesmo setor, mede o grau de desvio de cada par em relação à sua relação habitual e gera um relatório organizado e de fácil leitura.

Para cada par, calcula a razão de preços em uma janela móvel, a média e o desvio padrão dessa razão, além do *z-score* atual (quantos desvios padrão o par está em relação à normalidade). Quando um par ultrapassa um determinado limite (2 desvios padrão), ele é sinalizado: identifica-se qual ação está "cara", qual está "barata" e a direção de uma possível operação de reversão à média. A ferramenta também classifica a qualidade do sinal — uma divergência moderada tende a reverter, enquanto uma extrema sugere que a correlação foi rompida, sendo, portanto, marcada para evitar a operação.

O desafio de engenharia interessante

O plano gratuito da API de preços (brapi) possui limites de taxa de requisição (*rate limits*). Buscar dados de todos os pares a cada execução esgota rapidamente esse limite. Resolvi isso implementando uma camada de memória local: a cada execução, a ferramenta lê um arquivo CSV com todos os dados já coletados, incorpora os novos dados (*candles*) sem duplicar datas e salva o arquivo atualizado. Com o tempo, a ferramenta constrói seu próprio histórico local de preços e depende menos de chamadas em tempo real — um cache simples que transforma uma limitação rígida da API em algo irrelevante. Como funciona (pipeline)
Gatilho (Trigger) → gera a lista de pares de ações a serem verificados (agrupados por setor)
Busca (Fetch) → obtém dados de velas (candles) da brapi, um ticker por vez, com uma pausa entre as requisições para respeitar o limite de taxa (rate limit)
Cérebro (Nó de código) → alinha as duas séries de preços por data e calcula a razão (ratio), o z-score e a qualidade do sinal para cada par
Exportação para Excel → um relatório classificado contendo: par, setor, z-score, sinal, indicação de compra/venda, preços e distância percentual
Ramificação de memória → lê o CSV local → mescla novos dados de velas → salva novamente, permitindo o acúmulo do histórico entre as execuções
Stack tecnológica
n8n para orquestração
API REST da brapi para dados de mercado da B3
JavaScript (nós de código) para cálculos estatísticos: razão, média móvel/desvio padrão, z-score
CSV/Excel para persistência local e relatório de saída
Observação sobre o escopo

Esta é uma ferramenta de triagem e organização de dados; não constitui aconselhamento financeiro nem um sistema de negociação automatizada. Ela destaca divergências estatísticas para análise humana. Os mercados são ruidosos e as correlações podem se romper; um par sinalizado serve como ponto de partida para análise, nunca como uma recomendação direta de negociação.

⚠️ O token da API brapi foi substituído por um marcador (COLE_SEU_TOKEN_AQUI). Insira seu próprio token para executar o fluxo.
