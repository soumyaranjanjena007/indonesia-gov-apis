# LPSE / INAPROC — Government Procurement Data

**Agency:** LKPP (National Procurement Policy Agency)
**Central Portal:** https://inaproc.id
**LPSE Network:** https://lpse.*.go.id (689 individual portals)
**SiRUP (Plans):** https://sirup.lkpp.go.id
**API type:** ⚠️ Limited central API + structured HTML scraping

## Overview

All Indonesian government procurement runs through LPSE (Layanan Pengadaan Secara Elektronik) portals. 689 regional portals all run identical SPSE software — one scraper works on all of them. INAPROC aggregates centrally.

## INAPROC Central Search

```python
import requests

resp = requests.get("https://inaproc.id/api/tender", params={
    "keyword": "konstruksi gedung",
    "status": "aktif",
    "page": 1,
    "limit": 20,
})
tenders = resp.json()
```

## LPSE Portal Scraping (works on all 689 portals)

All portals run identical SPSE software — same HTML structure everywhere:

```python
from bs4 import BeautifulSoup
import requests

def search_lpse(base_url, keyword, page=0):
    resp = requests.get(f"{base_url}/lelang/list", params={
        "namaPaket": keyword,
        "statusLelang": "selesai",
        "page": page,
        "rpp": 50,
    }, timeout=30)
    soup = BeautifulSoup(resp.text, "html.parser")
    rows = soup.select("table.list-tender tbody tr")
    results = []
    for row in rows:
        cells = row.find_all("td")
        if len(cells) >= 4:
            results.append({
                "name": cells[1].text.strip(),
                "agency": cells[2].text.strip(),
                "value": cells[3].text.strip(),
            })
    return results

# Works on any LPSE portal
data = search_lpse("https://lpse.jakarta.go.id", "jalan")
```

## Major LPSE Portals

| Portal | URL |
|--------|-----|
| Central (LKPP) | https://lpse.lkpp.go.id |
| Jakarta | https://lpse.jakarta.go.id |
| Jawa Barat | https://lpse.jabarprov.go.id |
| Kemenkes | https://lpse.kemkes.go.id |
| Kemen PUPR | https://lpse.pu.go.id |

## SiRUP — Annual Procurement Plans

```python
resp = requests.get("https://sirup.lkpp.go.id/sirup/ro/rekappaketpenyedia/satker", params={
    "tahun": 2025,
    "idSatker": "123456",
})
```

## Gotchas

1. **SPSE software is identical across portals** — write scraper once, run everywhere
2. **INAPROC central API is limited** — not all fields available centrally
3. **Pagination** — `page` is 0-indexed, use `rpp` for page size
4. **Vendor history lookup** — useful for due diligence on contractor past wins
5. **Some portals block datacenter IPs** — residential proxies may be needed
6. **SiRUP = plans only** — actual contracts are in LPSE
7. **DNS unreliable** — many LPSE portals (including lpse.lkpp.go.id) had DNS resolution failures from both Indonesian and non-Indonesian IPs as of March 2026
