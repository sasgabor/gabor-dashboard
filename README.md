# gabor-dashboard

Két élő, statikus HTML dashboard, GitHub Pages-en hosztolva. Mindkettő Notion API-ból tölt adatot egy Cloudflare Worker proxy-n keresztül.

🔗 **Élő linkek:**
- Gábor OS: <https://sasgabor.github.io/gabor-dashboard/>
- Rosas Logisztikai Kft.: <https://sasgabor.github.io/gabor-dashboard/rosas.html>

---

## 📁 Fájlok

| Fájl         | Mit csinál                                                                                               | Utolsó frissítés |
| ------------ | --------------------------------------------------------------------------------------------------------- | ---------------- |
| `index.html` | Gábor OS – személyes önfejlesztési dashboard (Feelfit, Garmin, streak-ek, napirend)                       | 2026.07.02.      |
| `rosas.html` | Rosas Logisztikai Kft. – belső céges dashboard (pénzügy, KPI projekt, marketing átvétel, EU AI Act/GDPR)  | 2026.06.22.      |
| `README.md`  | Ez a fájl                                                                                                  | 2026.07.02.      |

---

## 🧍 Gábor OS (`index.html`)

Személyes egészség- és önfejlesztési dashboard.

**Tartalma:** napi streak-ek (cukor-, lisztmentes napok), Feelfit testösszetétel-adatok, Garmin élettani mutatók (pulzus, HRV, alvás, böjt), napi cél-üzenet, napirend.

**Adatforrás:** Notion *Egészség & Életmód* oldal, kizárólag a lap tetején lévő **"📌 Legfrissebb ismert adatok"** canonical táblából, `parseCanonicalTable()` + `mapCanonicalRows()` olvassa ki élőben (2026.06.23-tól — korábban szétszórt kulcsszó-kereséssel az egész lapon, ez törékeny volt). Ha a Notion API nem elérhető, a dashboard fallback módra vált beégetett, legutóbb ismert értékekkel, 📌-jelzéssel — ezt NEM kell minden adatfrissítésnél bumpolni, csak alkalmanként.

**Frissítési protokoll:** lásd a `gabor-dashboard` Skill-t. Napi adatfeldolgozás után NEM kell automatikusan újragenerálni — a dashboard élőben olvas a Notion canonical tábláról.

---

## 🌹 Rosas Logisztikai Kft. (`rosas.html`)

Céges belső dashboard a Rosas Logisztikai Kft. számára.

**Tartalma:**
- Pénzügyi KPI-k (árbevétel, üzemi/adózott eredmény, YoY Q1 összehasonlítás)
- 20%-os Nyereségnövekedés KPI Projekt státusza
- Marketing házon belülre hozása projekt (Facebook/LinkedIn/Ads átvétel)
- EU AI Act & GDPR megfelelési projekt státusza
- Projektek & Iniciatívák (jelenleg kézzel karbantartott statikus lista)

**Adatforrás:** A Projektek & Iniciatívák szekció jelenleg **kézzel karbantartott statikus lista** (az élő Notion-lekérdezés a Cloudflare Worker hiánya miatt nem működött – ld. Changelog). A többi szekció statikusan generált tartalom.

**Frissítési protokoll:** lásd a `rosas-dashboard` Skill-t (minden munkamenet végén automatikusan ellenőrzendő, kell-e frissítés).

---

## 🔧 Technikai architektúra (mindkét dashboardra)

```
Böngésző → Cloudflare Worker (notion-proxy) → Notion API → vissza
```

- **Cloudflare Worker:** `https://notion-proxy.sasgabor-sg.workers.dev`
- **Notion token:** csak a Cloudflare Worker Secrets-ben tárolva, sosem a HTML-ben *(2026.06.23-tól: korábban az `index.html`-ben is szerepelt egy kliens-oldali NOTION_TOKEN konstans nyílt szövegben, ami egy publikus repóban bárki számára olvasható volt — eltávolítva, ld. Changelog. A Worker a kimenő Notion-hívásnál mindig a saját env.NOTION_TOKEN secret-jét használja, a kliens által küldött fejléket figyelmen kívül hagyja.)*
- ✅ **Token-kezelés (javítva, 2026.07.02.):** a Notion integrations oldalon a "Show" (szem) és "Copy" ikon **korlátlanul, következmény nélkül** használható – ezek csak megjelenítik/másolják a tokent, NEM regenerálják. **Kizárólag** a középső, kör-nyíl "Refresh" ikon regenerálja és érvényteleníti azonnal a régit – ezt csak szándékos token-csere esetén szabad megnyomni.
  > ⚠️ Ez a sor korábban (2026.06.23–2026.07.02. között) tévesen azt állította, hogy minden "Show" kattintás regenerálja a tokent. A hiba a rendszerpromptban és a `gabor-dashboard` skillben már 2026.06.29-én javítva lett, de ide — több korábbi állítás ellenére, hogy "feltöltve és checksum-ellenőrizve" — a mai napig nem lett ténylegesen átvezetve. Most van csak ténylegesen javítva ebben a fájlban.
- ⚠️ **Nyitott proxy:** a Worker jelenleg nem ellenőrzi, ki hívja – bárki, aki ismeri a Worker URL-t, közvetlenül tud Notion API-hívásokat indítani rajta keresztül (olvasás ÉS írás is, mert a Worker minden HTTP metódust továbbenged). Mivel az URL nyilvánosan elérhető (a dashboard forráskódjában is szerepel), ez egy nyitott kapu a teljes Notion workspace-hez. Érdemes megfontolni egy megosztott titkos fejléc (pl. egyéni `X-Dashboard-Key` header) hozzáadását a Workerhez, amit csak a dashboard ismer és a Worker ellenőriz, mielőtt továbbítja a kérést. (Ez a kockázat még nyitott, nem sürgős.)

---

## 📤 Feltöltési mód

Mindkét fájlt **manuálisan** kell feltölteni:
1. `github.com/sasgabor/gabor-dashboard`
2. `Add file → Upload files`
3. Fájl kiválasztása/húzása (a régi felülíródik)
4. `Commit changes`
5. Élő linken `Ctrl+Shift+R` a böngésző cache törléséhez

⚠️ **Fontos, 2026.07.02-i tanulság:** egy korábbi munkamenet kétszer is (v3.15, v3.16) azt állította, hogy ez a feltöltés megtörtént és élő checksum-összevetéssel meg lett erősítve — élőben megnyitva viszont kiderült, hogy a régi, hibás tartalom volt élesben egészen mostanáig. **Ne tekintsd egy fájl frissítését késznek, amíg a `github.com/sasgabor/gabor-dashboard` oldalt közvetlenül meg nem nyitottad és a tartalmat szó szerint össze nem vetetted.**

---

## 📝 Changelog

> ⚠️ Két külön blokkban (Gábor OS / Rosas) — ne keverd időrendben egy közös táblázatba, mert úgy nehéz észrevenni, ha az egyik projekt changelog-ja elmaradt.

### Gábor OS (`index.html`)

| Dátum       | Mi változott                                                                                                                                                                                                                                                                                                                                                                                                      |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 2026.06.23. | Architektúra-átalakítás: a régi, szétszórt kulcsszó-kereséses parser (`parseCounters`/`parseHealthData`) helyett egyetlen, a Notion canonical táblára (📌 Legfrissebb ismert adatok) célzott parser (`parseCanonicalTable`/`mapCanonicalRows`), egységes `renderDashboard()` élő és fallback esetre egyaránt. `getPageBlocks()` egy szintet lemegy a gyerek-blokkokba is.                                        |
| 2026.06.23. | Biztonsági javítás: eltávolítva a kliens-oldali NOTION_TOKEN konstans. Hozzáadva: 📌-jelzés akkor is, ha egy konkrét mező fallback-re esett vissza.                                                                                                                                                                                                                                                              |
| 2026.06.18. | FALLBACK adatok frissítve a 06.17–06.18-i Feelfit/Garmin/számláló adatokra.                                                                                                                                                                                                                                                                                                                                       |
| 2026.06.06. | Fallback értékek frissítve (Feelfit + Garmin adatok)                                                                                                                                                                                                                                                                                                                                                              |

### Rosas (`rosas.html`)

| Dátum       | Mi változott                                                                                                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 2026.06.22. | Projektek & Iniciatívák szekció statikus listára váltva (a Cloudflare Worker élő Notion-kapcsolat még nem épült meg) |
| 2026.06.22. | EU AI Act & GDPR megfelelési kártya hozzáadva; verzió v4.2                                                                                                                     |
| 2026.06.09. | Marketing átvétel projekt + 2026 Q1 pénzügyi adatok hozzáadva; verzió v4.0                                                                                                     |
| 2026.06.08. | KPI Projekt szekció hozzáadva (20%-os nyereségnövekedés)                                                                                                                       |

### README.md

| Dátum       | Mi változott                                                                                                                                                                                                                                                                                            |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2026.07.02. | A "Show gomb regenerál" téves mítosz **ténylegesen** javítva (korábbi, 06.29–07.01-i "javítva" állítások hamisnak bizonyultak — a régi szöveg élőben egészen mostanáig fennmaradt). "Fájlok" táblázat dátumai frissítve. Figyelmeztetés hozzáadva: fájlfrissítést csak élő, közvetlen ellenőrzés után szabad késznek tekinteni. |
| 2026.06.23. | Frissítve az `index.html` "Utolsó frissítés" dátuma; javítva a Notion token elhelyezéséről szóló állítás; jelzés a nyitott Worker-proxy kockázatáról; changelog két külön blokkra bontva.                                                                                                            |
| –           | Létrehozva                                                                                                                                                                                                                                                                                              |
