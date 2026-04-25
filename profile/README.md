# microquery — [microquery.dev](https://microquery.dev)

**Pay-per-query SQL for AI agents. Billed in USDC on Base.**

Agents need data. They can't hold a credit card or sign up for a
subscription. Microquery gives them a wallet-native way to pay: deposit
USDC into an on-chain escrow, send a SQL query over HTTP, pay for bytes
scanned. No browser. No SSO. No human in the loop.

Supports [x402](https://x402.org) — agents that speak the HTTP 402 payment
protocol can query without pre-registration. One header, one round trip.

```http
POST /query?database=nvd HTTP/1.1
Authorization: EIP712 <base64-signed>

SELECT id, cvss_v3_score FROM cve
WHERE cvss_v3_score > 9.0
ORDER BY cvss_v3_score DESC LIMIT 10
```

Cost: **$0.15 / TB scanned**. A CVE lookup (~1 GB) costs $0.0002. A
full on-chain scan (1 TB) costs $0.15. Each query includes a signed
spending cap — the agent controls exactly how much it can be charged.

## MCP — Claude Desktop & Cursor

For AI assistants running inside an MCP host, install the
[microquery-mcp](https://github.com/microqueryhq/microquery-mcp) server.
Claude handles the SQL; you just ask the question. Zero config: auto-registers
on first query, no API key or wallet required.

```json
{
  "mcpServers": {
    "microquery": {
      "command": "python3",
      "args": ["/path/to/microquery_mcp.py"]
    }
  }
}
```

## Datasets

| Dataset | Size | database · table |
| ------- | ---- | ---------------- |
| *Blockchain — Ethereum* | | |
| ETH Transactions | 136 GB | `eth` · `transactions` |
| ETH Transfers | 70 GB | `eth` · `transfers` |
| ETH DEX Swaps | 17 GB | `eth` · `dex_swaps` |
| ETH Contracts | 10 GB | `eth` · `contracts` |
| ETH MEV relay | 610 MB | `eth` · `mev` |
| ETH Blocks | 800 MB | `eth` · `blocks` |
| ETH Lending | 500 MB | `eth` · `lending` |
| ETH LP Events | 400 MB | `eth` · `lp_events` |
| *Blockchain — Bitcoin* | | |
| BTC Outputs | 115 GB | `btc` · `outputs` |
| BTC Blocks | 6.6 GB | `btc` · `blocks` |
| *Blockchain — Base L2* | | |
| Base Transactions | — | `base` · `transactions` |
| Base Transfers | — | `base` · `transfers` |
| Base Blocks | — | `base` · `blocks` |
| *DeFi* | | |
| DeFi TVL history | 365 MB | `defi` · `tvl` |
| DeFi Protocols | 2 MB | `defi` · `protocols` |
| *Biomedical* | | |
| PubMed / MEDLINE | 138 GB | `pubmed` · `baseline` |
| FDA FAERS | 6.5 GB | `fda` · `faers` |
| ClinVar | 2.0 GB | `clinvar` · `variants` |
| ClinicalTrials | 700 MB | `clinicaltrials` · `studies` |
| FDA Orange Book | 5 MB | `fda` · `orangebook` |
| *Genomics* | | |
| GWAS Associations | — | `gwas` · `associations` |
| ClinPGx Clinical Variants | — | `clinpgx` · `clinical_variants` |
| ClinPGx Drug Labels | — | `clinpgx` · `drug_labels` |
| ClinPGx Guidelines | — | `clinpgx` · `guidelines` |
| ClinPGx Relationships | — | `clinpgx` · `relationships` |
| Illumina GDA PGx | — | `illumina` · `gda_pgx` |
| Illumina GSA PGx | — | `illumina` · `gsa_pgx` |
| *Financial / Government* | | |
| FEC Campaign Finance | 35 GB | `fec` · `contributions` |
| SEC EDGAR | 6.2 GB | `sec` · `edgar` |
| FRED Economic Series | 136 MB | `fred` · `series` |
| OFAC / BIS Sanctions | 8 MB | `sanctions` · `entities` |
| FHFA House Prices | — | `fhfa` · `hpi` |
| NADAC Drug Prices | — | `nadac` · `prices` |
| World Bank Commodities | — | `worldbank` · `commodity_prices` |
| *Security / Dev* | | |
| NVD / CVE | 700 MB | `nvd` · `cve` |
| OSV Advisories | 400 MB | `osv` · `advisories` |
| MalwareBazaar | — | `malwarebazaar` · `samples` |
| End-of-Life | 1 MB | `eol` · `cycles` |
| *Research* | | |
| arXiv | 3.6 GB | `arxiv` · `papers` |
| Open Food Facts | — | `openfoodfacts` · `products` |

Call `GET /v1/databases` for the live schema including field names and
partition keys.

## Get started

```bash
# Register — returns an api_key and $0.10 trial credit
curl -s -X POST https://microquery.dev/v1/register \
  -H "Content-Type: application/json" \
  -d '{"name": "my-agent"}' | jq .

# Run a query
curl -s \
  -H "Authorization: Bearer $API_KEY" \
  "https://microquery.dev/query?database=nvd" \
  --data-raw "SELECT id, cvss_v3_score FROM cve
              WHERE cvss_v3_score > 9.0 LIMIT 5"
```

Response headers tell the agent exactly what happened:
`X-Microquery-Cost-MicroUSDC`, `X-Microquery-Balance-MicroUSDC`,
`X-Microquery-Bytes-Scanned`.

## Repositories

| Repo | Description |
| ---- | ----------- |
| [microquery-mcp](https://github.com/microqueryhq/microquery-mcp) | MCP server for Claude Desktop, Cursor, and any MCP host — auto-registers, auto-tops-up, no config required |
| [microquery-agent](https://github.com/microqueryhq/microquery-agent) | Autonomous agent — registers, deposits USDC via EIP-2612 permit, discovers datasets, runs queries, monitors balance |
| [microquery-agent-x402](https://github.com/microqueryhq/microquery-agent-x402) | x402 variant — no registration needed, pays via EIP-3009 `TransferWithAuthorization`; supports per-query challenge-response or upfront deposit mode |

## Links

[docs](https://microquery.dev/docs) ·
[dataset schemas](https://microquery.dev/datasets) ·
[FAQ](https://microquery.dev/faq) ·
[MCP server](https://github.com/microqueryhq/microquery-mcp)

---

*Each query is a fast, cheap scan. Chain them. Use your context window
as the join buffer.*
