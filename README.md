# 🇮🇩 Indonesia Government APIs & Data Sources

A comprehensive reference for Indonesian government open data portals, APIs, and data sources — with practical Python examples, scraping patterns, and gotchas learned from production use.

> **Why this exists:** Indonesian government APIs are poorly documented, frequently change without notice, and have quirks not covered in official docs. This repo captures real-world knowledge from building production applications against these data sources.

## Data Sources

### Finance & Economy
| Source | Agency | Docs | API? |
|--------|--------|------|------|
| [OJK Legality Check](apis/ojk/) | Otoritas Jasa Keuangan | Licensed & illegal financial entities | ❌ Scraping |
| [Bank Indonesia](apis/bank-indonesia/) | Bank Indonesia | Exchange rates, interest rates, money supply | ✅ REST API |
| [IDX Stock Data](apis/idx/) | Indonesia Stock Exchange | Stock prices, corporate actions | ⚠️ Unofficial |
| [BPS Statistics](apis/bps/) | Badan Pusat Statistik | National statistics (GDP, CPI, trade, etc.) | ✅ REST API |
| [BAPPEBTI](apis/bappebti/) | Badan Pengawas Perdagangan Berjangka Komoditi | Commodities futures brokers | ❌ HTML scraping |

### Food Safety & Halal
| Source | Agency | Docs | API? |
|--------|--------|------|------|
| [BPJPH Halal](apis/bpjph/) | Badan Penyelenggara Jaminan Produk Halal | Halal certification database (~1.98M businesses) | ✅ JSON API |
| [BPOM](apis/bpom/) | Badan Pengawas Obat dan Makanan | Registered food, drugs, cosmetics | ⚠️ DataTables |

### Government Services
| Source | Agency | Docs | API? |
|--------|--------|------|------|
| [data.go.id](apis/data-go-id/) | National Open Data Portal | 10,000+ datasets (CKAN-based) | ✅ CKAN API |
| [LAPOR](apis/lapor/) | National Public Complaint System | Complaint tracking | ❌ No API |

### Tax & Business
| Source | Agency | Docs | API? |
|--------|--------|------|------|
| [NTA Invoice Registry](apis/nta/) | National Tax Agency (JP equivalent for reference) | Invoice number validation | ✅ REST API (requires app ID) |

## Quick Start

```python
# Example: Search BPJPH halal database
import requests

resp = requests.post(
    "https://cmsbl.halal.go.id/api/search/data_penyelia",
    json={"length": 20, "start": 0, "nama_penyelia": "A"},
    headers={"Content-Type": "application/json"}
)
businesses = resp.json()["data"]
print(f"Found {len(businesses)} businesses")
```

## Common Patterns & Gotchas

### 🚫 IP Blocking
Most Indonesian government sites block datacenter IPs (AWS, GCP, DO). Solutions:
1. **Cloudflare Workers proxy** — route requests through CF edge (recommended)
2. **Residential proxy** — reliable but costs money
3. **Rate limiting** — 2-5s delays between requests

### 📄 Data Formats
Indonesian government sites predominantly use:
- **Excel (.xlsx)** — most common download format
- **PDF** — especially for financial/legal data
- **HTML tables** — DataTables jQuery plugin is ubiquitous
- **JSON APIs** — rare but exist (BPJPH, BPS, BI)

### 🔄 Pagination
Common pagination patterns:
- DataTables: `start` + `length` params, `recordsTotal` in response
- CKAN: `offset` + `limit` params
- Custom: varies wildly

### 🔐 Authentication
- Most public data: no auth needed
- CSRF tokens: BPOM requires session cookies + CSRF
- API keys: BPS requires free registration
- OAuth: rare (BI API for some endpoints)

## Project Structure

```
apis/
├── bank-indonesia/     # Exchange rates, BI Rate, statistics
├── bappebti/           # Commodities futures brokers
├── bpjph/              # Halal certification database
├── bpom/               # Food, drug, cosmetics registry
├── bps/                # National statistics (BPS)
├── data-go-id/         # National open data portal (CKAN)
├── idx/                # Stock exchange data
├── lapor/              # Public complaint system
├── nta/                # Tax invoice validation
└── ojk/                # Financial entity legality
examples/
├── search_halal.py     # Search halal-certified businesses
├── check_ojk.py        # Check if a financial entity is legal
├── bi_exchange.py      # Get Bank Indonesia exchange rates
└── bps_inflation.py    # Get inflation data from BPS
```

## Contributing

Know an Indonesian government API that's not listed? Found a gotcha? PRs welcome!

## Disclaimer

This project documents publicly available government data sources for educational and research purposes. It is not affiliated with any Indonesian government agency. Always respect rate limits and terms of service.

## License

MIT
