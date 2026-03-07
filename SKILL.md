---
name: indonesia-gov-data
description: "Reference for 50 Indonesian government data sources, APIs, and MCP servers. Use when building apps that need Indonesian government data, scraping government websites, or connecting to Indonesian data MCP servers."
metadata:
  tags: "indonesia,government,api,data,scraping,mcp"
  updated: "2026-03"
---

# Indonesia Government Data Reference

## When to Use This Skill
- Building applications that query Indonesian government data
- Scraping Indonesian government websites
- Looking up API endpoints for BPS, BMKG, BPJPH, OJK, etc.
- Connecting to Indonesian data MCP servers (pasal.id)
- Cross-referencing company/entity data across multiple registries

## Quick Reference — All 50 Sources

### Tier 1: Open APIs (Ready to Consume)
| # | Source | API Endpoint | Auth | Key Data |
|---|--------|-------------|------|----------|
| 1 | data.go.id (SDI) | `data.go.id/api/3/action/` | None | 10K+ datasets (CKAN) |
| 2 | BPS Statistics | `webapi.bps.go.id/v1/api` | API Key (free) | GDP, CPI, population, trade |
| 3 | BMKG Weather | `data.bmkg.go.id/DataMKG/` | None | Weather, earthquakes, tsunami |
| 4 | IDX / BEI | `idx.co.id/api` (unofficial) | None | Stock prices, corporate data |
| 5 | pasal.id (MCP) | `pasal-mcp-server-production.up.railway.app/mcp` | None | 40K regulations, 937K articles |
| 6 | JDIH BPK | `api-jdih.perpusnas.go.id` | None | Legal documents (JSON) |
| 7 | Putusan MA | `putusan.mahkamahagung.go.id` | None | Court decisions |
| 8 | LPSE / INAPROC | `inaproc.lkpp.go.id` | None | Government procurement |
| 9 | APBN Kemenkeu | `data-apbn.kemenkeu.go.id` | None | State budget data |
| 10 | Bank Indonesia | `api-sandbox.bi.go.id` | OAuth2 | Exchange rates, BI Rate |
| 11 | BIG Geospatial | `map.big.go.id/wfs` | None | Admin boundaries, zoning (WFS) |
| 12 | BNPB Disaster | `inarisk.bnpb.go.id/api` | None | Disaster events, risk scores |

### Tier 2: Scrapeable Web (No formal API)
| # | Source | URL | Format | Key Data |
|---|--------|-----|--------|----------|
| 13 | BPJPH Halal | `cmsbl.halal.go.id` | JSON (POST) | 1.98M halal businesses |
| 14 | BPOM Products | `cekbpom.pom.go.id` | DataTables+CSRF | 242K food/drug/cosmetic registrations |
| 15 | AHU Company | `ahu.go.id` | HTML+CAPTCHA | All registered companies (PT, CV) |
| 16 | OSS / NIB | `oss.go.id` | HTML | Business ID (NIB) lookup |
| 17 | OJK Registry | `ojk.go.id` | HTML+XLS | Licensed financial entities |
| 18 | KPK e-LHKPN | `elhkpn.kpk.go.id` | HTML+PDF | Public officials' wealth declarations |
| 19 | Putusan MK | `mkri.id` | HTML+PDF | Constitutional court decisions |
| 20 | KSEI Statistics | `ksei.co.id` | PDF/XLSX | Securities investor stats |
| 21 | e-PPID | `ppid.*.go.id` | Per ministry | Public information requests |
| 22 | Pajak / DJP | `pajak.go.id` | Login required | NPWP verification |

### Tier 3: Regional Open Data (CKAN-based)
| # | Source | URL | Quality |
|---|--------|-----|---------|
| 23 | Satu Data Jakarta | `satudata.jakarta.go.id/api/3/action` | ⭐ Best regional |
| 24 | Open Data Jabar | `opendata.jabarprov.go.id/api` | ⭐ Good |
| 25 | Open Data Jatim | `opendata.jatimprov.go.id` | Good |
| 26 | Satu Data Surabaya | `satudata.surabaya.go.id` | Good |
| 27 | Open Data Bandung | `opendata.bandung.go.id/api` | Good |
| 28 | Open Data Bali | `data.baliprov.go.id` | Basic (CSV) |

### Tier 4-7: Ministry, Transparency, Financial, Civil Society
See individual docs in `apis/tier4-ministry/` through `apis/tier7-civil-society/`.

## MCP Server Setup

```bash
# Connect pasal.id to Claude
claude mcp add --transport http pasal-id https://pasal-mcp-server-production.up.railway.app/mcp
```

## Common Scraping Patterns

### IP Blocking Workaround
Most Indonesian gov sites block datacenter IPs. Use a Cloudflare Workers proxy.

### CSRF Token Handling (BPOM pattern)
```python
session = requests.Session()
page = session.get("https://cekbpom.pom.go.id/produk/pangan-olahan")
csrf = BeautifulSoup(page.text, "html.parser").find("meta", {"name": "csrf-token"})["content"]
```

### CKAN API Pattern (data.go.id, Jakarta, Jabar, etc.)
```python
resp = requests.get("https://data.go.id/api/3/action/package_search", params={"q": "keyword", "rows": 10})
```

### DataTables Pagination
```python
resp = session.post(url, data={"start": offset, "length": 100, "draw": page_num})
total = resp.json()["recordsTotal"]
```
