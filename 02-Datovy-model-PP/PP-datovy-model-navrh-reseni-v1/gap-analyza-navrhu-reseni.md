# Gap analýza: návrh řešení vs. závěry se HEN

**Účel:** Porovnání `datovy-model-navrh-reseni-123372122.md` (návrh řešení PP, stav ER v1) s `hen-nove-entity-analyza.md` (závěry s dodavatelem HEN ze StudiaRealizovatelnosti.docx + Obec Košariská.xlsx).
**Datum:** 2026-04-01
**Revize:** 2026-04-07 — po zapracování v2 datového modelu (`datovy-model-navrh-reseni-123372122.md`, verze v2)
**Výstup:** seznam mezer a jejich aktuální stav; otevřené body k potvrzení.

---

## Přehled nalezených mezer

| # | Oblast | Typ | Závažnost | Stav po revizi v2 |
|---|--------|-----|-----------|-------------------|
| 1 | Zóna položky | Chybějící entita | Vysoká | ✅ Zapracováno — sekce 15 DM v2 |
| 2 | Región | Chybějící entita | Střední | ✅ Zapracováno — sekce 16 DM v2 |
| 3 | Dni vývozu — atributy | Nekompletní DDL | Vysoká | ✅ Zapracováno — 8 atributů + `vyvoz` místo `typ_dne` |
| 4 | Rozvrh — frekvence a parametry | Nekompletní DDL | Vysoká | ✅ Zapracováno — `frekvence` + nullable parametry (AR-03) |
| 5 | Zóna — atributy a adresní model | Nesprávný přístup | Vysoká | ✅ Zapracováno — geometrie odebrána z DS (AR-02) |
| 6 | External ID (číslo nonsubjektu) na RPO | Chybějící atribut | Vysoká | ✅ Zapracováno — strategie `external_id` pro všechny entity |
| 7 | Okruh — chybějící atributy | Nekompletní DDL | Střední | ✅ Zapracováno — `poznamka`, `referencia`, `external_id` |
| 8 | Integrační model — sada API služeb | Chybějící sekce | Vysoká | ⚠️ Mimo scope DM — řeší samostatná integrační analýza |
| 9 | Okruh-položky vs. RPO_Okruh_Rozvrh | Nesoulad modelů | Vysoká | ✅ Rozhodnuto — zachován PP model `RPO_Okruh_Rozvrh` (AR-01) |
| 10 | Vazba Zóna → Région a Útvar | Chybějící vazba | Střední | ✅ Zapracováno — `region_id` přidán na Zónu |
| 11 | `vozidlo_id` na Okruhu | Otevřená otázka | Nízká | ✅ Zdůvodněno — záměrné rozšíření PP pro plánování DV |

---

## Detailní popis

### 1. Chybějící entita: Zóna položky

**Stav v návrhu:** Zóna je modelována jako entita s atributem `geometrie` (geodatový typ nebo GeoJSON). Entita `Zóna položky` v návrhu neexistuje.

**Co říkají závěry HEN:** Zóna se NEDEFINUJE geometrií — obsahuje **hierarchickou adresní definici** (Okres → Obec → Časť obce → Mestská časť → Ulica → Číslo domu od/do). Každá položka zóny zužuje rozsah přes vyšší úrovně, konkrétní adresné body se dopočítávají.

**Co dopracovat:**
- Přidat entitu `zona_polozka` (nebo `zone_item`) s atributy:
  - `id` (PK)
  - `zona_id` (FK → `zone`)
  - `poradí` / `riadok` (číslo řádku)
  - `okres_id` (FK nebo text)
  - `obec_id` (FK nebo text)
  - `cast_obce` (text, výběr z adresných bodů)
  - `mestska_cast` (text)
  - `ulica_id` (FK nebo text)
  - `cislo_od` (int)
  - `cislo_do` (int)
- Rozhodnout, zda `geometrie` zůstane jako **odvozená / computed hodnota** (vizualizace na mapě), nebo zda se odstraní.

**✅ Stav po revizi v2:** Entita `zona_polozka` přidána v sekci 15 DM v2. Atributy odpovídají HEN modelu (`poradi`, `okres`, `obec`, `cast_obce`, `mestska_cast`, `ulica`, `cislo_od`, `cislo_do`). Gap uzavřen.

---

### 2. Chybějící entita: Región

**Stav v návrhu:** Entita `Región` zcela chybí.

**Co říkají závěry HEN:** Jednoduchý textový číselník (`Referencia`, `Název`), na který odkazuje hlavička Zóny. Slouží jako kategorizace zón.

**Co dopracovat:**
- Přidat entitu `region` s atributy:
  - `id` (PK)
  - `kod` / `referencia` (nvarchar, business identifikátor z HEN)
  - `nazev` (nvarchar)
  - `external_id` (nvarchar, ID z HEN pro synchronizaci)
- Přidat FK `region_id` na entitě `zone`.

**✅ Stav po revizi v2:** Entita `region` přidána v sekci 16 DM v2 s atributy `id`, `external_id`, `nazev`. FK `region_id` přidán na entitě `zone`. Gap uzavřen.

---

### 3. Nekompletní DDL: Dni vývozu (Kalendar)

**Stav v návrhu:** Entita `Kalendar` má atributy: `id`, `rozvrh_id`, `datum`, `typ_dne`.

**Co říkají závěry HEN:** Entita Dni vývozu (items Rozvrhu vývozov) obsahuje dalších 8 atributů viditelných v reálných datech (Obec Košariská.xlsx):

| Atribut HEN | Typ | Chybí v návrhu |
|---|---|---|
| `Deň v týždni` | text (Pondelok, Utorok...) | ✗ |
| `Týždeň v mesiaci` | číslo | ✗ |
| `Štvrťrok` | číslo | ✗ |
| `Týždeň` | číslo (číslo týdne v roce) | ✗ |
| `Typ týždňa` | text (Párny/Nepárny) | ✗ |
| `Víkend` | boolean | ✗ |
| `Sviatok` | boolean | ✗ |
| `Posledný v mesiaci` | boolean | ✗ |
| `Poznámka` | text | ✗ |
| `Vývoz` (ÁNO/NIE) | boolean | `typ_dne` jen částečně pokrývá |

**Co dopracovat:**
- Rozšířit DDL tabulky `kalendar` o výše uvedené atributy.
- Přehodnotit roli atributu `typ_dne` — v HEN je klíčovým atributem `vyvoz` (boolean), nikoliv obecný „typ dne". Potvrdit, zda `typ_dne` slouží k něčemu jinému, nebo zda jde o aliasování.
- Potvrdit, zda jsou atributy jako `tyden_v_mesici`, `stvrteleti`, `typ_tydne` v PP počítány aplikačně z `datum`, nebo přenášeny z HEN jako data.

**✅ Stav po revizi v2:** Entita `kalendar` v sekci 4 DM v2 rozšířena o všechny chybějící atributy (`den_v_tydnu`, `tyden_v_mesici`, `tyden`, `typ_tydne`, `stvrteleti`, `vikend`, `svatek`, `posledni_v_mesici`, `poznamka`). Atribut `vyvoz bit` nahradil `typ_dne`. **Otevřený bod:** Potvrdit, zda jsou odvozené atributy přenášeny z HEN jako data nebo počítány aplikačně v PP.

---

### 4. Nekompletní DDL: Rozvrh — frekvence a parametry

**Stav v návrhu:** Entita `Rozvrh` má atributy: `id`, `nazev`, `provozovna_id`, `platnost_od`, `platnost_do`.

**Co říkají závěry HEN:** Rozvrh vývozov má atribut `Frekvencia` (kombobox: Denne / Týždenně / Vlastné), přičemž každý typ frekvence má vlastní parametry:
- **Denní** — startovací datum, počet opakování, interval (každý N-tý den)
- **Týdenní** — vybrané dny v týdnu + typ týdne (párný/nepárný)
- **Vlastní** — manuální zadání počtu dnů/měsíc

**Co dopracovat:**
- Přidat atribut `frekvence` (enum/nvarchar: `DENNI` / `TYDENNI` / `VLASTNI`).
- Rozhodnout, jak modelovat parametry frekvence:
  - Varianta A: Sada nullable sloupců na `rozvrh` (denormalizovaný přístup vhodný pro jednoduché dotazy)
  - Varianta B: Samostatná tabulka `rozvrh_parametry` s typovým zápisem
- Atribut `Útvar` (DV → Stredisko) z HEN je v návrhu jako `provozovna_id`. Potvrdit, zda je sémantická shoda správná, nebo zda Útvar má jiný rozsah než provozovna.
- Názov rozvrhu v HEN je `Kompozitní` (sestavován automaticky z roku/týdne/dnů/počtu dnů) — potvrdit, zda se v PP ukládá jako odvozený string nebo zda existuje uživatelsky editovatelný `nazev`.

**✅ Stav po revizi v2:** Entita `rozvrh` v sekci 3 DM v2 rozšířena o atribut `frekvence nvarchar(20) not null` (hodnoty: DENNI/TYDENNI/VLASTNI) a nullable parametrové sloupce (`start_datum`, `interval_dni`, `pocet_opakovani`, `dny_v_tydnu`, `typ_tydne`, `pocet_dni_mesic`) — dle AR-03 Varianta A. Gap uzavřen. **Otevřený bod:** Potvrdit, zda `provozovna_id` v PP odpovídá HEN konceptu `Útvar/Stredisko`.

---

### 5. Nesoulad v modelu Zóny — geometrie vs. adresní definice

**Stav v návrhu:** Zóna má atribut `geometrie geometry(Polygon)` (geodatový typ), sloužící k prostorovému vymezení.

**Co říkají závěry HEN:** Zóna v HEN **neobsahuje geometrii** — je definována hierarchicky jako adresní podmínka (Okres/Obec/Ulica/č.domu). Konkrétní adresné body jsou dopočítávány jako výsledek filtrace, geometrie jako taková v HEN modelu není.

**Co dopracovat:**
- Potvrdit business rozhodnutí: Bude `geometrie` v PP:
  - a) **Vizualizační pomůcka** — generovaná z adresy body (PP ji počítá z adresního modelu)? → Pak je `geometrie` odvozená, do DS nepatří, řešení je technické.
  - b) **Primární definice** — PP nebo RP mohou definovat zónu geometricky (bez vazby na HEN adresní hierarchii)? → Pak jsou dva modely zóny vedle sebe a je potřeba je vyjasnit.
- Pokud zůstane `geometrie`, doplnit zdůvodnění a oddělit ji od HEN adresního modelu (sekce „atributy specifické pro PP").

**✅ Stav po revizi v2:** Dle AR-02 je geometrie odvozená vizualizační pomůcka — odebrána z business DS entity `zone`. Entita `zona_polozka` (Gap #1) přenáší adresní definici z HEN. Pokud bude potřeba vizualizace na mapě, geometrie bude řešena technicky mimo DM. Gap uzavřen.

---

### 6. Chybějící atribut na RPO: External ID (číslo nonsubjektu)

**Stav v návrhu:** RPO má atribut `kod_polozky` (business zkratka z HEN), ale chybí technický external klíč.

**Co říkají závěry HEN:** Kritický předpoklad pro celou integraci — entity `CustomerItem` (= RPO/Predmet zmluvy) musí být rozšířeny o **primární klíč předmětu z HEN** (`číslo nonsubjektu`, třída `lcs.d2b_w_predmet_zmluvy`). Bez tohoto klíče nelze provázat záznamy napříč systémy (HEN ↔ PP ↔ RP) jednoznačně.

**Co dopracovat:**
- Přidat na entitu `rpo` atribut `external_id` (nebo `hen_predmet_id / hen_nonsubject_id`) jako `nvarchar(50) not null` (nebo `bigint`, dle formátu z HEN).
- Přidat unikátní index `UX_rpo_external_id` na tento sloupec.
- Specifikovat, zda `external_id` nahrazuje `kod_polozky`, nebo zda obě hodnoty plní jinou roli (`kod_polozky` = business reference pro uživatele, `external_id` = technický klíč pro integraci).
- Stejný princip ověřit u dalších synchronizovaných entit (Okruh, Rozvrh, Zóna, Región).

**✅ Stav po revizi v2:** Atribut `external_id nvarchar(50) not null` s UNIQUE indexem přidán na `rpo` (sekce 1 DM v2). Strategie `external_id` rozšířena na všechny synchronizované entity (RPO, Okruh, Rozvrh, Kalendar, Zóna, Région). Gap uzavřen.

---

### 7. Nekompletní atributy: Okruh

**Stav v návrhu:** `Okruh` má atributy `id`, `nazev`, `provozovna_id`, `vozidlo_id`, `aktivni`.

**Co říkají závěry HEN:**
- Chybí atribut `Poznámka` (text)
- `Referencia` v HEN (= identifikátor okruhu) se **NEgeneruje automaticky** — zadává uživatel. V návrhu je `id bigint identity`, což je technický PK. Klíčová otázka: jak se bude v PP zobrazovat/přenášet business reference okruhu z HEN? Je potřeba přidat `referencia nvarchar(50)` jako oddělenou hodnotu?
- Atributy `Dni zvozu`, `Typ týždňa`, `Celkový počet nádob`, `Celkový objem nádob` jsou v HEN **computed** (odvozené z navázaných Predmetov zmluvy). V PP budou pravděpodobně počítány, ale je vhodné je zmínit v návrhu jako součást zobrazení entit.

**Co dopracovat:**
- Přidat `poznamka nvarchar(max) null` na entitu `okruh`.
- Přidat atribut `referencia nvarchar(50) null` (business identifikátor z HEN) — alternativně zvážit, zda `nazev` tuto roli plní.
- Doplnit sekci „computed atributy / agregáty zobrazované v UI" pro okruh.
- Přidat `external_id` pro synchronizaci s HEN (viz bod 6).

**✅ Stav po revizi v2:** Atributy `poznamka nvarchar(max) null`, `referencia nvarchar(50) null` a `external_id nvarchar(50) not null` přidány na entitu `okruh` (sekce 2 DM v2). Gap uzavřen. **Otevřený bod:** Potvrdit vztah `provozovna_id` ↔ HEN `Útvar/Stredisko` pro entity Okruh, Rozvrh, Zóna.

---

### 8. Chybějící sekce: Integrační model — sada API služeb

**Stav v návrhu:** Integrace není v dokumentu popsána; dokument se soustředí na datový model PP.

**Co říkají závěry HEN:** Integrace **nepůjde rozšiřováním MDB** (explicitně potvrzeno). Místo toho bude sada integračních API služeb:

**Datasety (RP volá HEN, čtení):**
- Kalendáře (seznam rozvrhů vývozov)
- Dni svozů (dny pro daný kalendář)
- Zóna (definice vč. položek)
- Okruh (definice vč. položek — predmety)
- Región (číselník)

**Procesy (HEN notifikuje RP):**
- Vytvoření/Smazání kalendáře
- Změna dne svozu na kalendáři (přepnutí Vývoz ÁNO/NIE)
- Přiřazení/Odvázání kalendáře k predmetu
- Změna na definici Zóny (región, rozsah adresních bodů)
- Vytvoření/Smazání okruhu
- Přiřazení/Odvolání predmetu z okruhu

**Co dopracovat:**
- Do návrhu řešení přidat sekci „Integrační model" nebo odkaz na integrační specifikaci.
- Zmínit, že cílový přenos dat z HEN je přes API, nikoli přes MDB — tato informace má přímý dopad na:
  - způsob synchronizace entit v PP (`external_id` na každé synchronizované entitě)
  - business pravidla triggeru změny (HEN pushuje, nebo PP pulluje?)
  - dopad na atributy `platnost_od/platnost_do` (pp spravuje platnost, nebo jen přebírá co HEN zašle?)

**⚠️ Stav po revizi v2:** Gap mimo scope DM. Integrační model (API služby HEN → PP, triggery synchronizace, datové kontrakty, chybové stavy) bude řešen v samostatné integrační analýze navazující na DM v2. DM v2 obsahuje odkaz na tento záměr a strategie `external_id` tvoří technický základ pro integraci.

---

### 9. Nesoulad modelů: Okruh-položky vs. RPO_Okruh_Rozvrh

**Stav v návrhu:** Vazba RPO ↔ Okruh ↔ Rozvrh je vedena přes jednu temporální entitu `RPO_Okruh_Rozvrh` s atributy `platnost_od/platnost_do`. Přímá entita „Okruh položky" neexistuje.

**Co říkají závěry HEN:** V HEN existují dvě oddělené datové struktury:
- **Okruh trasy položky** — snapshot (predmety zahrnuté v okruhu, bez platnosti)
- **Přiřazení Rozvrhu k Predmetu** — přiřazení je na úrovni Predmetu, nikoli okruhu

Tj. HEN neoperuje s kombinovanou entitou „RPO ↔ Okruh ↔ Rozvrh" v jedné tabulce — Okruh i Rozvrh jsou na Predmetu zmluvy vedeny zvlášť.

**Co dopracovat:**
- Potvrdit business rozhodnutí: Bude PP přebírat **HEN model** (okruh a rozvrh zvlášť na RPO) nebo zachová **vlastní temporální model** `RPO_Okruh_Rozvrh` jako konsolidaci?
- Pokud zůstane `RPO_Okruh_Rozvrh`, vyjasnit: jak se bude synchronizovat z HEN (HEN posílá zvlášť přiřazení okruhu a zvlášť přiřazení rozvrhu → PP je mapuje do jediné tabulky)?
- Pokud se přejde na HEN model, přidat entitu `okruh_polozka` (Okruh trasy ↔ RPO) a atributy `rozvrh_id` přímo na `rpo`.

**✅ Stav po revizi v2:** Dle AR-01 (Varianta A) zachován PP model `RPO_Okruh_Rozvrh`. Zdůvodnění: konsolidace dvou HEN struktur do jednoho záznamu PP, temporální platnost, výhoda pro Etapu 2+. Mapování dvou HEN estrutur do jednoho PP záznamu zajistí integrační vrstva. Gap uzavřen.

---

### 10. Chybějící vazby: Zóna → Région a Zóna → Útvar

**Stav v návrhu:** Entita `Zóna` má `organization_unit_id` (FK na provozovnu). Vazba na Région chybí.

**Co říkají závěry HEN:**
- `Zóna → Région` (N:1) — Région je na záhlaví Zóny jako statický vztah (číselník)
- `Zóna → DV Útvar` (N:1) — Stredisko ve "warehouseovém" smyslu (DV = dynamická vazba, odlišná od statické FK)

**Co dopracovat:**
- Přidat `region_id bigint null` (FK → `region`) na entitu `zone`.
- Přidat index `IX_zone_region`.
- Potvrdit, zda `organization_unit_id` (provozovna) v PP odpovídá HEN konceptu `Útvar/Stredisko`. HEN rozlišuje DV Útvar jako dynamický výběr — může jít o N:M vazbu.
- Stejnou otázku Útvar vs. provozovna ověřit pro entity `Okruh` a `Rozvrh`.

**✅ Stav po revizi v2:** FK `region_id bigint null` přidán na entitu `zone` (sekce 5 DM v2). Index `IX_zone_external_id` přidán. **Otevřený bod:** Potvrdit, zda `organization_unit_id` (provozovna) v PP sémanticky odpovídá HEN `Útvar/Stredisko`, nebo zda je potřeba N:M vazba — ověřit pro entity Okruh, Rozvrh, Zóna.

---

### 11. Otevřená otázka: `vozidlo_id` na Okruhu

**Stav v návrhu:** Entity `okruh` obsahuje `vozidlo_id bigint null` (FK → `vehicle`).

**Co říkají závěry HEN:** Entita Okruh trasy v HEN atribut vozidla **neobsahuje**. Vozidlo v HEN není součástí definice okruhu.

**Co udělat:**
- Potvrdit, zda vazba `okruh → vozidlo` je záměrné rozšíření PP nad rámec HEN (např. pro plánování v RP), nebo jde o omyl v návrhu.
- Pokud jde o záměr, popsat businessové pravidlo a zdroj pravdy pro tuto vazbu.

**✅ Stav po revizi v2:** Záměrné rozšíření PP nad HEN potvrzeno a zdůvodněno v sekci 2 DM v2 — `vozidlo_id` slouží k přednastavení pravidelného vozidla pro automatizaci generování DV v RP. Gap uzavřen.

---

## Přehled: které entity a atributy z HEN nejsou v návrhu

> Stav aktualizován po revizi DM v2 (2026-04-07)

| HEN entita | HEN atributy zahrnout/doplnit | Stav po revizi v2 |
|---|---|---|
| `Zóna položky` | viz bod 1 | ✅ Entita `zona_polozka` přidána (sekce 15) |
| `Región` | `referencia`, `nazev`, `external_id` | ✅ Entita `region` přidána (sekce 16) |
| `Dni vývozu` | `den_v_tydnu`, `tyden_v_mesici`, `stvrteleti`, `tyden`, `typ_tydne`, `vikend`, `svatek`, `posledni_v_mesici`, `poznamka` | ✅ Zapracováno v `kalendar` (sekce 4) |
| `Rozvrh vývozov` | `frekvence` + parametry dle type | ✅ Zapracováno v `rozvrh` (sekce 3, AR-03) |
| `Zóna (hlavička)` | `poznamka`, `region_id`, `stav` (enum) | ✅ Zapracováno v `zone` (sekce 5); `geometrie` odebrána (AR-02) |
| `Okruh trasy (hlavička)` | `poznamka`, `referencia` (HEN kód), `external_id` | ✅ Zapracováno v `okruh` (sekce 2) |
| `Okruh trasy položky` | `riadok`, `pocet_nadoby`, `typ_nadoby`, `objem_nadoby`, `odpad`, `nazov_odpadu` | ⚠️ Model `RPO_Okruh_Rozvrh` zachován (AR-01) — snapshot položek není přenesen |
| `RPO` | `external_id` (číslo nonsubjektu z HEN) | ✅ Zapracováno v `rpo` (sekce 1) |

---

## Otevřené body po revizi v2

> Všechny hlavní gapy jsou zapracovány nebo rozhodnuty. Níže jsou otevřené body vyžadující potvrzení před dalším krokem (integrační analýza / DDL finalizace).

| # | Otevřený bod | Dopad | Priorita |
|---|---|---|---|
| OB-01 | Atributy Kalendáře (`den_v_tydnu`, `tyden_v_mesici` aj.) — přenášeny z HEN jako data nebo počítány aplikačně v PP? | Způsob synchronizace, výkon, konzistence | Střední |
| OB-02 | `provozovna_id` v PP vs. HEN `Útvar/Stredisko` — sémantická shoda? (ověřit pro Okruh, Rozvrh, Zóna) | Případná potřeba N:M vazby místo FK | Střední |
| OB-03 | `RPO_Okruh_Rozvrh` — max 1 aktivní vazba na RPO nebo možné souběžné?  | Filtrovaný UNIQUE index, validační logika | Střední |
| OB-04 | Kolize DDL: vazba `RPO → Nádoba` — přímý FK `rpo_id` na Nádobě (ER v2) vs. `container_order_item_assignment` v inventáři PP | Migrační strategie, dopad na existující DDL | Vysoká |
| OB-05 | `Skupina odpadu.provozovna_id` — inventář PP tuto vazbu neuvádí, potvrdit migrační strategii | Dopad na existující `garbage_group` | Střední |
| OB-06 | Lokace nových entit (Okruh, Rozvrh, Kalendář, Zóna) — inventář zmiňuje možné umístění v RP místo PP. Potvrdit cílový systém/databázi | Dopad na integrační API a přístupová práva | Vysoká |
| OB-07 | Integrační model (Gap #8) — zahájit samostatnou integrační analýzu navazující na DM v2 | Kompletnost specifikace před implementací | Vysoká |
