# gabor-dashboard

Két élő, statikus HTML dashboard, GitHub Pages-en hosztolva. Mindkettő Notion API-ból tölt adatot egy Cloudflare Worker proxy-n keresztül.

🔗 **Élő linkek:**
- Gábor OS: https://sasgabor.github.io/gabor-dashboard/
- Rosas Logisztikai Kft.: https://sasgabor.github.io/gabor-dashboard/rosas.html

---

## 📁 Fájlok

| Fájl | Mit csinál | Utolsó frissítés |
|------|------------|-------------------|
| `index.html` | Gábor OS – személyes önfejlesztési dashboard (Feelfit, Garmin, streak-ek, napirend) | 2026.06.23. |
| `rosas.html` | Rosas Logisztikai Kft. – belső céges dashboard (pénzügy, KPI projekt, marketing átvétel, EU AI Act/GDPR) | 2026.07.02. |
| `README.md` | Ez a fájl | 2026.07.02. |

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
- Marketing házon belülre hozása projekt (Facebook/LinkedIn/Ads átvétel) + Golden Brothers meeting eredménye
- Sales Autopilot – 200+ fős lista átvitele (önálló szekció)
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
- ✅ A Notion integrations oldalon a "Show" és "Copy" ikon korlátlanul, következmény nélkül használható. **Kizárólag** a középső, kör-nyíl "Refresh" ikon regenerálja és érvényteleníti a tokent – csak szándékos token-csere esetén nyomd meg. *(2026.06.29-i javítás: a korábbi állítás, miszerint minden "Show" kattintás érvénytelenít, téves volt — élő Notion-dokumentáció alapján ellenőrizve, lásd rendszerprompt v3.11.)*
- ⚠️ **Nyitott proxy:** a Worker jelenleg nem ellenőrzi, ki hívja – bárki, aki ismeri a Worker URL-t, közvetlenül tud Notion API-hívásokat indítani rajta keresztül (olvasás ÉS írás is, mert a Worker minden HTTP metódust továbbenged). Mivel az URL nyilvánosan elérhető (a dashboard forráskódjában is szerepel), ez egy nyitott kapu a teljes Notion workspace-hez. Érdemes megfontolni egy megosztott titkos fejlék (pl. egyéni `X-Dashboard-Key` header) hozzáadását a Workerhez, amit csak a dashboard ismer és a Worker ellenőriz, mielőtt továbbítja a kérést.
- ⚠️ **Rosas (`rosas.html`) jelenleg NEM ezt az architektúrát használja** – teljesen statikus, kézzel karbantartott HTML, élő Notion-Worker kapcsolat nélkül. Ez a szakasz a tervezett/jövőbeli állapotot írja le a Rosas dashboardra nézve.

---

## 📤 Feltöltési mód

Mindkét fájlt **manuálisan** kell feltölteni:
1. `github.com/sasgabor/gabor-dashboard`
2. `Add file → Upload files`
3. Fájl kiválasztása/húzása (a régi felülíródik)
4. `Commit changes`
5. Élő linken `Ctrl+Shift+R` a böngésző cache törléséhez

⚠️ A GitHub online szerkesztőbe (Edit) copy-paste-elt HTML-ben az ékezetes karakterek és emoji-k encoding-hibát szenvedhetnek, ami a CSS teljes összeomlását okozhatja. Kizárólag az `Add file → Upload files` módszer megbízható.

---

## 📝 Changelog

> ⚠️ 2026.06.23-tól két külön blokkban (Gábor OS / Rosas) — ne keverd időrendben egy közös táblázatba, mert úgy nehéz észrevenni, ha az egyik projekt changelog-ja elmaradt.

### Gábor OS (`index.html`)

| Dátum | Mi változott |
|-------|---------------|
| 2026.06.23. | Architektúra-átalakítás: a régi, szétszórt kulcsszó-kereséses parser (`parseCounters`/`parseHealthData`) helyett egyetlen, a Notion canonical táblára (📌 Legfrissebb ismert adatok) célzott parser (`parseCanonicalTable`/`mapCanonicalRows`), egységes `renderDashboard()` élő és fallback esetre egyaránt (nincs többé külön, szétcsúszható kódág). `getPageBlocks()` egy szintet lemegy a gyerek-blokkokba is. |
| 2026.06.23. | Biztonsági javítás: eltávolítva a kliens-oldali NOTION_TOKEN konstans (nyílt szövegben szerepelt egy publikus repóban, funkcionálisan nem is volt szükséges). Hozzáadva: 📌-jelzés akkor is, ha a fő Notion-lekérdezés sikeres, de egy konkrét mező (streak, Feelfit vagy Garmin dátum) mégis fallback-re esett vissza – korábban ez csendben történt, élő adatnak álcázva. |
| 2026.06.18. (utólag rögzítve) | FALLBACK adatok frissítve a 06.17–06.18-i Feelfit/Garmin/számláló adatokra – ez a frissítés korábban nem került be ide, csak a fájlba. |
| 2026.06.06. | Fallback értékek frissítve (Feelfit + Garmin adatok) |

### Rosas (`rosas.html`)

| Dátum | Mi változott |
|-------|---------------|
| 2026.07.02. (3. kör – v4.17) | A v4.16-ban nyitva hagyott utolsó kérdés (Facebook Ads Manager "GB-Csali 2026 – 58 letöltő, 520 Ft/letöltés" riport-száma) Gábor megerősítésével lezárult: ténylegesen 200+ csali letöltő van, mindkét korábbi "58"-as szám (CRM-es és Facebook-kampányos is) elavult volt. A letöltésenkénti költség (520 Ft) jelenleg nem releváns, egy külön statisztikai vizsgálat tárgya lesz később – törölve mindenhonnan (rendszerprompt Marketing szakasz, Notion Marketing oldal Aktív kampányok táblája, `rosas.html` verzió-jelzés v4.17-re frissítve). |
| 2026.07.02. (2. kör) | A v4.16-os csere után végzett ellenőrző-audit talált egy live hibát: az előző kör sed-parancsa lemaradt a footer sorról (a footer külön sorban végződött "v4.14"-gyel, a fejléc/alcím viszont már helyesen v4.16-ot mutatott) – javítva. Emellett: a fenti "Fájlok" táblázat "Utolsó frissítés" oszlopa a rosas.html/README.md sorokban még 2026.06.30-at mutatott a tényleges 2026.07.02. helyett – javítva (ugyanaz a hiba-mintázat, mint amit korábban az index.html táblázat-soránál is találtunk). A `rosas-session-close` skill FALLBACK objektumában talált `csali_letoltok: 58` érték is elavult volt (a 200+ korrekció óta) – javítva. |
| 2026.07.02. | Konzisztencia-audit (a Gábor OS projektből átvitt önellenőrzési útmutató alapján) 3 tételt talált és zárt le: (1) a `rosas.html` és e README még v4.14-et mutatott a tényleges v4.16 helyett — javítva; (2) a "Csali letöltők – 58 kontakt" szám tévesnek bizonyult (a valós szám 200+, az alvállalkozói egyeztetésből kiderülve) — a `rosas-crm` skillben és a Notion Marketing oldalon is javítva, a rendszerpromptban minden előfordulás frissítve; (3) az EU AI Act szakaszban pontosítva, hogy az AI Literacy oktatási anyag már elkészült (Word, 2026.06.30.), csak a tényleges oktatás megtartása van hátra. |
| 2026.06.30. | Konzisztencia-audit alapján frissítve: rendszerprompt-verzió jelzés v4.10 → **v4.14** (fejléc + footer); 2025-ös árbevétel KPI-kártya kerekítése 549 M → 550 M javítva (pontos érték: 549,54 M Ft); új **Sales Autopilot – 200+ fős lista átvitele** szekció felvéve (Top prioritás sáv, Következő lépések, Projektek & Iniciatívák/Folyamatban oszlop is kiegészítve); **Golden Brothers 2026.06.30-i meeting eredménye** bekerült a Marketing kártyába (LinkedIn elengedve, Facebook + AI Chatbot + Kvíz folytatódik max 3 hónap átmenettel). |
| 2026.06.22. | Projektek & Iniciatívák szekció statikus listára váltva (a Cloudflare Worker élő Notion-kapcsolat még nem épült meg, ezért "Nincs adat" jelent meg minden oszlopban – javítva) |
| 2026.06.22. | EU AI Act & GDPR megfelelési kártya hozzáadva; verzió v4.2 |
| 2026.06.09. | Marketing átvétel projekt + 2026 Q1 pénzügyi adatok hozzáadva; verzió v4.0 |
| 2026.06.08. | KPI Projekt szekció hozzáadva (20%-os nyereségnövekedés) |

### README.md

| Dátum | Mi változott |
|-------|---------------|
| 2026.07.01. | A Rosas-projekt hibakatalógusa alapján végzett kereszt-ellenőrzés talált egy élő hibát: a Technikai architektúra szakasz tévesen állította, hogy minden "Show" kattintás a Notion integrations oldalon regenerálja a tokent – ez már 2026.06.29-én (rendszerprompt v3.11) javítva lett élőben, de ide nem lett átvezetve. Most javítva. |
| 2026.06.30. | Frissítve a `rosas.html` "Utolsó frissítés" dátuma és a Rosas changelog a 2026.06.30-i dashboard-frissítéssel összhangba hozva (Sales Autopilot szekció, kerekítés-javítás, rendszerprompt-verzió, Golden Brothers meeting). Technikai architektúra szakasz kiegészítve egy jelzéssel, hogy a Rosas dashboard jelenleg NEM a Cloudflare Worker-es élő architektúrát használja, hanem teljesen statikus. Feltöltési mód szakasz kiegészítve a GitHub online szerkesztő encoding-hiba figyelmeztetésével. |
| 2026.06.23. | Frissítve az `index.html` "Utolsó frissítés" dátuma (a táblázat hónapokig elmaradt a valóságtól); javítva a Notion token elhelyezéséről szóló (akkor már nem igaz) állítás; jelzés a nyitott Worker-proxy kockázatáról; changelog két külön blokkra bontva (Gábor OS / Rosas). |
| – | Létrehozva (korábban csak cím szerepelt benne) |
