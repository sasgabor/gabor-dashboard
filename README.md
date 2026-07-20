# gabor-dashboard

Két élő, statikus HTML dashboard, GitHub Pages-en hosztolva. Mindkettő Notion API-ból tölt adatot egy Cloudflare Worker proxy-n keresztül.

🔗 **Élő linkek:**
- Gábor OS: <https://sasgabor.github.io/gabor-dashboard/>
- Rosas Logisztikai Kft.: <https://sasgabor.github.io/gabor-dashboard/rosas.html>

---

## 📁 Fájlok

| Fájl         | Mit csinál                                                                                               | Utolsó frissítés (kód) | Szinkron-komment frissült |
| ------------ | --------------------------------------------------------------------------------------------------------- | ---------------- | --- |
| `index.html` | Gábor OS – személyes önfejlesztési dashboard (Feelfit, Garmin, streak-ek, napirend)                       | 2026.06.23.      | 2026.07.01. (csak a komment szövege, kód nem változott) |
| `rosas.html` | Rosas Logisztikai Kft. – belső céges dashboard (pénzügy, KPI projekt, marketing átvétel, EU AI Act/GDPR)  | 2026.07.19.      | – |
| `README.md`  | Ez a fájl                                                                                                  | 2026.07.02.      | – |

⚠️ **2026.07.02-i javítás:** az `index.html` sorában korábban tévesen "2026.07.02." szerepelt "utolsó frissítés"-ként — ez saját elírás volt, nem valós adat. A fájl tényleges kódmódosítása 2026.06.23-i, csak a benne lévő szinkron-komment *szövege* frissült 07.01-én (rendszerprompt-verzió-szám). Ez most, egy élő fájl-megnyitással megerősítve, javítva.

---

## 🧍 Gábor OS (`index.html`)

Személyes egészség- és önfejlesztési dashboard.

**Tartalma:** napi streak-ek (cukor-, lisztmentes napok), Feelfit testösszetétel-adatok, Garmin élettani mutatók (pulzus, HRV, alvás, böjt), napi cél-üzenet, napirend.

**Adatforrás:** Notion *Egészség & Életmód* oldal, kizárólag a lap tetején lévő **"📌 Legfrissebb ismert adatok"** canonical táblából, `parseCanonicalTable()` + `mapCanonicalRows()` olvassa ki élőben (2026.06.23-tól — korábban szétszórt kulcsszó-kereséssel az egész lapon, ez törékeny volt). Ha a Notion API nem elérhető, a dashboard fallback módra vált beégetett, legutóbb ismert értékekkel, 📌-jelzéssel — ezt NEM kell minden adatfrissítésnél bumpolni, csak alkalmanként.

**Frissítési protokoll:** lásd a `gabor-dashboard` Skill-t. Napi adatfeldolgozás után NEM kell automatikusan újragenerálni — a dashboard élőben olvas a Notion canonical tábláról.

📝 **Rendszerprompt élő Notion-másolata** (2026.07.12. óta): a mindenkori Gábor OS rendszerprompt egy 1:1 másolata elérhető Notionban is, hogy Projekten kívüli beszélgetésben se kelljen manuálisan bemásolni — lásd a `gabor-session-close` Skill-t.

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

📝 **Rendszerprompt élő Notion-másolata** (2026.07.12. óta): a mindenkori Rosas rendszerprompt egy 1:1 másolata elérhető Notionban is, hogy Projekten kívüli beszélgetésben se kelljen manuálisan bemásolni — lásd a `rosas-session-close` Skill-t.

✅ **Lezárt tétel (2026.07.03., élőben megerősítve):** az árbevétel-KPI helyesen, kerekítve 550 M-et mutat (rosas-financials pontos alapadata: 549 540 ezer Ft). A jelszó `Rosas2026`, ez él, és sor-szintű (view-source) élő megerősítés is megtörtént (forráskód 6. sora: `var correct = "Rosas2026";`). Ez a bejegyzés korábban tévesen "még nem javított tételként" volt itt jelezve — ld. Rosas "⚖️ Döntési Napló" (Rosas Térkép oldal) és Rosas rendszerprompt v4.18.

---

## 🔧 Technikai architektúra (mindkét dashboardra)

```
Böngésző → Cloudflare Worker (notion-proxy) → Notion API → vissza
```

- **Cloudflare Worker:** `https://notion-proxy.sasgabor-sg.workers.dev`
- **Notion token:** csak a Cloudflare Worker Secrets-ben tárolva, sosem a HTML-ben *(2026.06.23-tól: korábban az `index.html`-ben is szerepelt egy kliens-oldali NOTION_TOKEN konstans nyílt szövegben, ami egy publikus repóban bárki számára olvasható volt — eltávolítva, ld. Changelog. A Worker a kimenő Notion-hívásnál mindig a saját env.NOTION_TOKEN secret-jét használja, a kliens által küldött fejléket figyelmen kívül hagyja.)*
- ✅ **Token-kezelés (javítva, 2026.07.02.):** a Notion integrations oldalon a "Show" (szem) és "Copy" ikon **korlátlanul, következmény nélkül** használható – ezek csak megjelenítik/másolják a tokent, NEM regenerálják. **Kizárólag** a középső, kör-nyíl "Refresh" ikon regenerálja és érvényteleníti azonnal a régit – ezt csak szándékos token-csere esetén szabad megnyomni.
- ⚠️ **Nyitott proxy:** a Worker jelenleg nem ellenőrzi, ki hívja – bárki, aki ismeri a Worker URL-t, közvetlenül tud Notion API-hívásokat indítani rajta keresztül. Reális javítás: a Worker szigorítása egy konkrét GET-lekérdezésre. Nem sürgős, nyitott kérdés.

---

## 📤 Feltöltési mód

Mindkét fájlt **manuálisan** kell feltölteni:
1. `github.com/sasgabor/gabor-dashboard`
2. `Add file → Upload files`
3. Fájl kiválasztása/húzása (a régi felülíródik)
4. `Commit changes`
5. Élő linken `Ctrl+Shift+R` a böngésző cache törléséhez

⚠️ **2026.07.02-i tanulság:** egy fájl frissítését csak akkor tekintsd késznek, ha közvetlenül megnyitottad és a tartalmat szó szerint összevetetted — egy korábbi "feltöltve, checksummal megerősítve" állítás egyszer tévesnek bizonyult.

---

## 📝 Changelog

### Gábor OS (`index.html`)

| Dátum       | Mi változott                                                                                                                                                                                                                                                                                                                                                                                                      |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 2026.07.19. | **v4.4** — Pótolva a `renderFigyelem()`-ből hiányzó napi alkohol-küszöb (bármely nap > 40 g). A `gabor-os` skill három küszöböt ír elő (>40 g/nap, >100 g/7 nap, 0 alkoholmentes nap); a v4.3 csak az utóbbi kettőt implementálta, így egy 45 g-os nap figyelmeztetés nélkül elment. |
| 2026.07.18. | **v4.3** — 🍷 `Alkohol` oszlop (a trend-tábla 14., utolsó oszlopa) + `renderAllapot()` (böjt-streak és alkoholmentes streak egymás mellett) + `renderFigyelem()`: küszöb-alapú figyelmeztető sáv, ami akkor szólal meg, ha a mutatók a saját céljaitól rossz irányba mennek. Gábor kérése: a rendszer ne csak udvariasan jelöljön, hanem szóljon rá. |
| 2026.07.18. | **v4.2** — 🕐 `Böjt` oszlop (a trend-tábla 13. oszlopa) + 12h+ böjt streak. Bővítési szabály rögzítve: új oszlop MINDIG a `COLS` tömb végére, sosem közé (különben a parser csendben rossz adatot olvas). A fejléc szinkron-kommentjéből törölve az elavult „Jelenlegi rendszerprompt verzió" mező. |
| 2026.07.17. | **v4.1** — Adatforrás-váltás: a napi idősort (7 napos trend) a dashboard mostantól **kizárólag** a `📊 Napi mérések – Trend napló (standard)` Notion-oldalról olvassa (`parseTrend()` + `TREND_PAGE_ID`), NEM a canonical táblából. A 2026.06.23-i `parseCanonicalTable()` / `mapCanonicalRows()` architektúra **kivezetve** — az alábbi 06.23-i sor innentől történeti. |
| 2026.07.14. | Kódváltozás: az EU AI Act & GDPR kártya frissítve (AI Compliance Dossier v1.1 + AI-folyamatleltár v1.3 elkészült; státusz: ügyvédnő válaszára várunk); új "Elvégzett fejlesztések" kártya a 07.14-i dosszié+leltár elküldéséről; fejléc dátuma 07.14. |
| 2026.07.12. | Nincs kódváltozás — a mindenkori rendszerprompt elérhetővé vált egy élő Notion-oldalon is (Projekten kívüli hozzáféréshez), lásd `gabor-session-close` Skill.                                                                                                                                                                                                                                                     |
| 2026.07.01. | Csak a fájl fejlécében lévő szinkron-komment szövege frissült (rendszerprompt-verzió-szám); a kód nem változott. 2026.07.02-én élő fájl-megnyitással megerősítve.                                                                                                                                                                                                                                               |
| 2026.06.23. | Architektúra-átalakítás: egyetlen, a Notion canonical táblára célzott parser (`parseCanonicalTable`/`mapCanonicalRows`), egységes `renderDashboard()`. Biztonsági javítás: eltávolítva a kliens-oldali NOTION_TOKEN konstans.                                                                                                                                                                                    |
| 2026.06.18. | FALLBACK adatok frissítve.                                                                                                                                                                                                                                                                                                                                                                                        |
| 2026.06.06. | Fallback értékek frissítve.                                                                                                                                                                                                                                                                                                                                                                                       |

### Rosas (`rosas.html`)

| Dátum       | Mi változott                                                                                                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 2026.07.19. | „Projektek & Iniciatívák" szekció kézi szinkron: felkerült az **RPOS – Rosas Folyamatoptimalizálási Rendszer** kártya a 🔄 Folyamatban oszlopba (anonimizált megfogalmazás, ügyfélnév és ár nélkül); a szekció láblécének dátuma 2026.07.19-re állítva. |
| 2026.07.12. | Nincs kódváltozás — a mindenkori rendszerprompt elérhetővé vált egy élő Notion-oldalon is (Projekten kívüli hozzáféréshez), lásd `rosas-session-close` Skill. |
| 2026.07.12. | Kódváltozás: az EU AI Act kártya AI Literacy-sora frissítve (valós, folyamatban lévő oktatási státuszra); a deprecated "Rendszerprompt verzió" mezők (fejléc-komment, alcím, footer) ténylegesen törölve a fájlból. |
| 2026.07.03. | Nincs kódváltozás — csak dokumentációs javítás: a README ezen fájl-táblázatban lévő "árbevétel-KPI 549 M / jelszó nincs megerősítve" megjegyzése elavult volt; élő, forráskód-szintű ellenőrzés megerősítette, hogy a `rosas.html` már 550 M-et és a helyes jelszót tartalmazza. |
| 2026.06.22. | Projektek & Iniciatívák szekció statikus listára váltva; EU AI Act & GDPR kártya hozzáadva; v4.2                                                                              |
| 2026.06.09. | Marketing átvétel projekt + 2026 Q1 pénzügyi adatok; v4.0                                                                                                                      |
| 2026.06.08. | KPI Projekt szekció hozzáadva                                                                                                                                                  |

### README.md

| Dátum       | Mi változott                                                                                                                                                                                                                                                                                            |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 2026.07.12. | Fájlok táblázat: `rosas.html` "utolsó frissítés (kód)" dátuma 07.12-re javítva. Mindkét szekcióhoz (Gábor OS, Rosas) megjegyzés + changelog-sor a rendszerprompt élő Notion-másolatáról. Rosas changelog: AI Literacy-sor és deprecated verziómezők törlésének rögzítése.                        |
| 2026.07.03. | Rosas szakasz "Ismert, még nem javított tétel" figyelmeztetése lezártra frissítve — élő, forráskód-szintű ellenőrzés (Rosas oldalról) megerősítette, hogy a `rosas.html` már 550 M árbevétel-KPI-t és a helyes `Rosas2026` jelszót tartalmazza. Ld. Rosas "⚖️ Döntési Napló".                        |
| 2026.07.02. | Fájlok táblázat: index.html "utolsó frissítés" dátuma javítva (téves 07.02 → helyes 06.23, kód/komment külön oszlopban). Rosas árbevétel/jelszó ismert-tétel megjegyzés hozzáadva.                                                                                                                    |
| 2026.07.02. | A "Show gomb regenerál" mítosz ténylegesen javítva és élőben megerősítve.                                                                                                                                                                                                                                |
| 2026.06.23. | Token elhelyezés javítva, nyitott proxy kockázat jelezve, changelog két blokkra bontva.                                                                                                                                                                                                                 |
| –           | Létrehozva                                                                                                                                                                                                                                                                                              |
