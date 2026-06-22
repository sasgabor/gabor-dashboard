# gabor-dashboard

Két élő, statikus HTML dashboard, GitHub Pages-en hosztolva. Mindkettő Notion API-ból tölt adatot egy Cloudflare Worker proxy-n keresztül.

🔗 **Élő linkek:**
- Gábor OS: https://sasgabor.github.io/gabor-dashboard/
- Rosas Logisztikai Kft.: https://sasgabor.github.io/gabor-dashboard/rosas.html

---

## 📁 Fájlok

| Fájl | Mit csinál | Utolsó frissítés |
|------|-----------|-------------------|
| `index.html` | Gábor OS – személyes önfejlesztési dashboard (Feelfit, Garmin, streak-ek, napirend) | 2026.06.06. |
| `rosas.html` | Rosas Logisztikai Kft. – belső céges dashboard (pénzügy, KPI projekt, marketing átvétel, EU AI Act/GDPR) | 2026.06.22. |
| `README.md` | Ez a fájl | 2026.06.22. |

---

## 🧍 Gábor OS (`index.html`)

Személyes egészség- és önfejlesztési dashboard.

**Tartalma:** napi streak-ek (cukor-, lisztmentes napok), Feelfit testösszetétel-adatok, Garmin élettani mutatók (pulzus, HRV, alvás, böjt), napi cél-üzenet, napirend.

**Adatforrás:** Notion *Egészség & Életmód* oldal, `parseCounters()` függvény olvassa ki a Számlálók szekciót. Ha a Notion API nem elérhető, a dashboard fallback módra vált beégetett, legutóbb ismert értékekkel – ezeket minden generálásnál frissíteni kell.

**Frissítési protokoll:** lásd a `gabor-dashboard` Skill-t (napi adatfeldolgozás után automatikusan frissítendő).

---

## 🌹 Rosas Logisztikai Kft. (`rosas.html`)

Céges belső dashboard a Rosas Logisztikai Kft. számára.

**Tartalma:**
- Pénzügyi KPI-k (árbevétel, üzemi/adózott eredmény, YoY Q1 összehasonlítás)
- 20%-os Nyereségnövekedés KPI Projekt státusza
- Marketing házon belülre hozása projekt (Facebook/LinkedIn/Ads átvétel)
- EU AI Act & GDPR megfelelési projekt státusza
- Projektek & Iniciatívák (élő Notion adatbázis-lekérdezés)

**Adatforrás:** Notion *Iniciatívák & Projektek* adatbázis (élő lekérdezés a böngészőben, Cloudflare Worker proxy-n keresztül), a többi szekció statikusan generált tartalom.

**Frissítési protokoll:** lásd a `rosas-dashboard` Skill-t (minden munkamenet végén automatikusan ellenőrzendő, kell-e frissítés).

---

## 🔧 Technikai architektúra (mindkét dashboardra)

```
Böngésző → Cloudflare Worker (notion-proxy) → Notion API → vissza
```

- **Cloudflare Worker:** `https://notion-proxy.sasgabor-sg.workers.dev`
- **Notion token:** csak a Cloudflare Worker Secrets-ben tárolva, sosem a HTML-ben
- ⚠️ A Notion integrations oldal megnyitása (Show gomb) regenerálja és érvényteleníti a tokent – csak akkor nyisd meg, ha valóban szükséges

---

## 📤 Feltöltési mód

Mindkét fájlt **manuálisan** kell feltölteni:
1. `github.com/sasgabor/gabor-dashboard`
2. `Add file → Upload files`
3. Fájl kiválasztása/húzása (a régi felülíródik)
4. `Commit changes`
5. Élő linken `Ctrl+Shift+R` a böngésző cache törléséhez

---

## 📝 Changelog

| Dátum | Fájl | Mi változott |
|-------|------|---------------|
| 2026.06.22. | rosas.html | EU AI Act & GDPR megfelelési kártya hozzáadva; verzió v4.2 |
| 2026.06.09. | rosas.html | Marketing átvétel projekt + 2026 Q1 pénzügyi adatok hozzáadva; verzió v4.0 |
| 2026.06.08. | rosas.html | KPI Projekt szekció hozzáadva (20%-os nyereségnövekedés) |
| 2026.06.06. | index.html | Fallback értékek frissítve (Feelfit + Garmin adatok) |
| – | README.md | Létrehozva (korábban csak cím szerepelt benne) |
