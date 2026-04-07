# Gap analýza: návrh řešení vs. závěry se HEN

**Účel:** Porovnání `datovy-model-navrh-reseni-123372122.md` (návrh řešení PP, stav ER v1) s `hen-nove-entity-analyza.md` (závěry s dodavatelem HEN ze StudiaRealizovatelnosti.docx + Obec Košariská.xlsx).
**Datum:** 2026-04-01
**Výstup:** seznam mezer a položek, které je potřeba v návrhu řešení dopracovat.

---

## Přehled nalezených mezer

| # | Oblast | Typ | Závažnost |
|---|--------|-----|-----------|
| 1 | Zóna položky | Chybějící entita | Vysoká |
| 2 | Región | Chybějící entita | Střední |
| 3 | Dni vývozu — atributy | Nekompletní DDL | Vysoká |
| 4 | Rozvrh — frekvence a parametry | Nekompletní DDL | Vysoká |
| 5 | Zóna — atributy a adresní model | Nesprávný přístup | Vysoká |
| 6 | External ID (číslo nonsubjektu) na RPO | Chybějící atribut | Vysoká |
| 7 | Okruh — chybějící atributy | Nekompletní DDL | Střední |
| 8 | Integrační model — sada API služeb | Chybějící sekce | Vysoká |
| 9 | Okruh-položky vs. RPO_Okruh_Rozvrh | Nesoulad modelů | Vysoká |
| 10 | Vazba Zóna → Région a Útvar | Chybějící vazba | Střední |
| 11 | `vozidlo_id` na Okruhu | Otevřená otázka | Nízká |

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

---

### 5. Nesoulad v modelu Zóny — geometrie vs. adresní definice

**Stav v návrhu:** Zóna má atribut `geometrie geometry(Polygon)` (geodatový typ), sloužící k prostorovému vymezení.

**Co říkají závěry HEN:** Zóna v HEN **neobsahuje geometrii** — je definována hierarchicky jako adresní podmínka (Okres/Obec/Ulica/č.domu). Konkrétní adresné body jsou dopočítávány jako výsledek filtrace, geometrie jako taková v HEN modelu není.

**Co dopracovat:**
- Potvrdit business rozhodnutí: Bude `geometrie` v PP:
  - a) **Vizualizační pomůcka** — generovaná z adresy body (PP ji počítá z adresního modelu)? → Pak je `geometrie` odvozená, do DS nepatří, řešení je technické.
  - b) **Primární definice** — PP nebo RP mohou definovat zónu geometricky (bez vazby na HEN adresní hierarchii)? → Pak jsou dva modely zóny vedle sebe a je potřeba je vyjasnit.
- Pokud zůstane `geometrie`, doplnit zdůvodnění a oddělit ji od HEN adresního modelu (sekce „atributy specifické pro PP").

---

### 6. Chybějící atribut na RPO: External ID (číslo nonsubjektu)

**Stav v návrhu:** RPO má atribut `kod_polozky` (business zkratka z HEN), ale chybí technický external klíč.

**Co říkají závěry HEN:** Kritický předpoklad pro celou integraci — entity `CustomerItem` (= RPO/Predmet zmluvy) musí být rozšířeny o **primární klíč předmětu z HEN** (`číslo nonsubjektu`, třída `lcs.d2b_w_predmet_zmluvy`). Bez tohoto klíče nelze provázat záznamy napříč systémy (HEN ↔ PP ↔ RP) jednoznačně.

**Co dopracovat:**
- Přidat na entitu `rpo` atribut `external_id` (nebo `hen_predmet_id / hen_nonsubject_id`) jako `nvarchar(50) not null` (nebo `bigint`, dle formátu z HEN).
- Přidat unikátní index `UX_rpo_external_id` na tento sloupec.
- Specifikovat, zda `external_id` nahrazuje `kod_polozky`, nebo zda obě hodnoty plní jinou roli (`kod_polozky` = business reference pro uživatele, `external_id` = technický klíč pro integraci).
- Stejný princip ověřit u dalších synchronizovaných entit (Okruh, Rozvrh, Zóna, Región).

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

---

### 11. Otevřená otázka: `vozidlo_id` na Okruhu

**Stav v návrhu:** Entity `okruh` obsahuje `vozidlo_id bigint null` (FK → `vehicle`).

**Co říkají závěry HEN:** Entita Okruh trasy v HEN atribut vozidla **neobsahuje**. Vozidlo v HEN není součástí definice okruhu.

**Co udělat:**
- Potvrdit, zda vazba `okruh → vozidlo` je záměrné rozšíření PP nad rámec HEN (např. pro plánování v RP), nebo jde o omyl v návrhu.
- Pokud jde o záměr, popsat businessové pravidlo a zdroj pravdy pro tuto vazbu.

---

## Přehled: které entity a atributy z HEN nejsou v návrhu

| HEN entita | HEN atributy zahrnout/doplnit | Stav v návrhu |
|---|---|---|
| `Zóna položky` | viz bod 1 | Entita chybí |
| `Región` | `referencia`, `nazev`, `external_id` | Entita chybí |
| `Dni vývozu` | `den_v_tydnu`, `tyden_v_mesici`, `stvrteleti`, `tyden`, `typ_tydne`, `vikend`, `svatek`, `posledni_v_mesici`, `poznamka` | Atributy chybí |
| `Rozvrh vývozov` | `frekvence` + parametry dle type | Atributy chybí |
| `Zóna (hlavička)` | `typ` (uživatelské číselné pole), `poznamka`, `region_id`, `stav` (enum) | Část/vše chybí |
| `Okruh trasy (hlavička)` | `poznamka`, `referencia` (HEN kód), `external_id` | Atributy chybí |
| `Okruh trasy položky` | `riadok`, `pocet_nadoby`, `typ_nadoby`, `objem_nadoby`, `odpad`, `nazov_odpadu` | Model neodpovídá |
| `RPO` | `external_id` (číslo nonsubjektu z HEN) | Atribut chybí |

---

## Doporučené pořadí dopracování

1. **Rozhodnutí: model integrace Okruh-položky** (bod 9) — ovlivňuje architekturu více entit, je třeba ho udělat dřív než DDL detaily
2. **Rozhodnutí: geometrie vs. adresní definice Zóny** (bod 5) — ovlivňuje potřebu entity Zóna položky
3. **External ID na RPO + synchronizovaných entitách** (bod 6) — kritické pro integraci
4. **Doplnit entitu Zóna položky** (bod 1)
5. **Doplnit entitu Région** (bod 2)
6. **Dopracovat DDL: Dni vývozu atributy** (bod 3)
7. **Dopracovat DDL: Rozvrh frekvence** (bod 4)
8. **Dopracovat DDL: Okruh atributy** (bod 7)
9. **Dopracovat DDL: Zóna atributy a vazby** (bod 10)
10. **Doplnit sekci Integrační model** (bod 8)
