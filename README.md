# Automated-Pairs-Trading-Screener-B3-Brazilian-Stock-Exchange-
Sistema de automação que faz triagem estatística de pares de ações da B3 para apoiar decisões de swing trade.
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
