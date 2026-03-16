# LPSE / INAPROC — Government Procurement Data

**Agency:** LKPP (National Procurement Policy Agency)
**Central Portal:** https://spse.inaproc.id (directory — Next.js, migrated 2026)
**Legacy LPSE Network:** https://lpse.*.go.id → most CNAME to ars.inaproc.id now
**API type:** ⚠️ Structured HTML scraping + SPSE JSON endpoints

## Migration Status (March 2026)

LKPP is migrating from individual `lpse.*.go.id` domains to `inaproc.id`:

| Domain | Status | Notes |
|--------|--------|-------|
| lpse.lkpp.go.id | ❌ DNS dead | Old central portal |
| lpse.pu.go.id | ❌ DNS dead | Ministry of Public Works |
| lpse.kominfo.go.id | ❌ DNS dead | Ministry of Communications |
| lpse.kemenkeu.go.id | ⚠️ CNAME → ars.inaproc.id | CF challenge (403) |
| lpse.kemkes.go.id | ⚠️ CNAME → ars.inaproc.id | CF challenge (403) |
| lpse.jakarta.go.id | ✅ Still active | DKI Jakarta |
| lpse.kemenag.go.id | ✅ Still active | Ministry of Religion |
| inaproc.id | ⚠️ CF Turnstile | Root domain challenges all requests |
| spse.inaproc.id | ✅ Portal (Next.js) | Directory only, no tender API |
| ars.inaproc.id | 🔒 Pomerium auth | Internal admin portal |

## SPSE JSON API (per-portal)

Individual portals still expose standard SPSE endpoints when accessible:

```python
import httpx

PORTALS = [
    "https://lpse.jakarta.go.id/eproc4",
    "https://lpse.kemenag.go.id/eproc4",
]

async def search_vendors(portal_base: str, keyword: str):
    """Standard SPSE vendor search — works on any /eproc4 portal."""
    async with httpx.AsyncClient(timeout=15) as client:
        resp = await client.get(
            f"{portal_base}/dt/rekanan",
            params={"term": keyword, "draw": "1", "start": "0", "length": "10"},
        )
        return resp.json()

async def search_tenders(portal_base: str, keyword: str):
    """Standard SPSE tender search."""
    async with httpx.AsyncClient(timeout=15) as client:
        resp = await client.get(
            f"{portal_base}/dt/tender",
            params={"term": keyword, "draw": "1", "start": "0", "length": "10"},
        )
        return resp.json()
```

## SPSE Response Format

### Vendor (`/dt/rekanan`)
```json
{
  "data": [
    {
      "kodeRekanan": "R-001",
      "namaRekanan": "PT CONTOH SEJAHTERA",
      "npwp": "01.234.567.8-012.000",
      "alamat": "Jl. Sudirman No. 1",
      "kota": "Jakarta",
      "statusAktif": true
    }
  ]
}
```

### Tender (`/dt/tender`)
```json
{
  "data": [
    {
      "kode": "T-001",
      "namaPaket": "Pengadaan Komputer",
      "namaSatker": "Dinas Pendidikan DKI Jakarta",
      "nilaiPagu": 1000000000,
      "tahapTender": "Pengumuman"
    }
  ]
}
```

## Gotchas

1. **SPSE software is identical across portals** — write scraper once, run everywhere
2. **Many portals migrating to inaproc.id** — individual domains becoming CNAMEs
3. **Cloudflare Turnstile** on inaproc.id — `cf-mitigated: challenge` header, requires browser/Playwright
4. **Jakarta proxy required** — geo-blocked portals need Indonesian IP
5. **spse.inaproc.id is a directory only** — lists LPSE instances, does NOT provide tender search API
6. **ars.inaproc.id requires Pomerium SSO** — not publicly accessible
7. **Pagination** — `start` is 0-indexed, `length` for page size
8. **DNS is unreliable** — check resolution before scraping, portals go down without notice

## civic-stack SDK

```python
from civic_stack.lpse import fetch, search, search_tenders

# Searches across all reachable portals
result = await fetch("PT Garuda Indonesia", proxy_url="socks5h://127.0.0.1:1080")
tenders = await search_tenders("konstruksi", proxy_url="socks5h://127.0.0.1:1080")
```
