# Progress: CS-MPG Analýza

## Aktuální stav
Fáze: Analýza datových modelů — příprava referenčních podkladů dokončena, připraveno na entitní analýzu
Poslední session: 2026-02-23 (session 9)
Další krok: Zahájit entitní analýzu PP — první entita (RPO) s review, výstup do docs/datovy-model-PP.md

## TODO
### Integrace
- [x] Vyplnit SPEC.md
- [x] Definovat plán prací (PLAN.md)
- [x] Analyzovat vstupní dokumenty (data/input-analysis.md)
- [x] Analyzovat MDB schéma (DDL) — 22 BCED_ entit, 6 candidate tabulek, sync tracking
- [x] Doplnit odpovědi na otevřené otázky (OQ-01 až OQ-10) — 5 uzavřeno, 4 částečně/otevřeno, 1 mimo scope
- [ ] Validace SoT matice se stakeholdery
- [ ] Detailní specifikace integračních toků (INT-01 až INT-06)
- [ ] Návrh integrační architektury
- [ ] Business zadání per integrační služba

### Datové modely
- [x] Zmapovat vstupní podklady (DDL PP, DDL RP, MDB DDL, Confluence exporty)
- [x] Analýza Cílový koncept vs. DDL — první průchod (docs/analyza-cilovy-koncept-vs-ddl.md)
  - PP: 11 požadavků, 6 nových tabulek + 6 nových sloupců
  - RP: 13 požadavků, ~8 nových tabulek + 11+ nových sloupců
- [x] Doplnit analýzu o datové slovníky (entitní model) — v1 (archivováno do docs/archive/v1/)
  - Plán: docs/PLAN-doplneni-analyzy-datovy-model.md
  - v1 výstupy archivovány — nedostatečný doménový kontext (DS vs DDL rozdíly, separátní CS model v RP)
  - **PRAVIDLO: z docs/archive/ nepřebírat** — DS↔DDL mapování vzniklo před doplněním doménového kontextu, hlásí implementační odchylky jako problémy
- [x] Strukturovaný výtah z Cílového konceptu (docs/vytah-cilovy-koncept.md, 1 165 řádků)
  - Zaměření: datový model a integrace
  - Zpracováno 4 paralelními agenty, konsolidováno do jednoho dokumentu
  - Všechny sekce CK: business procesy, doménový model, etapizace, PP, RP, FOB, integrace, SO, architektura, přílohy
- [x] Inventář entit DS + DDL (čistý, bez hodnocení)
  - `docs/inventar-entit-PP.md` (~1 180 řádků, 22 entit + 4 nové pro CS)
  - `docs/inventar-entit-RP.md` (~2 020 řádků, 31+9 entit + 5 nových pro CS)
  - Pravidlo: žádné DS↔DDL mapování, žádné hodnocení odchylek
- [x] Doplnění popisů entit a referencí z tech. projektů do inventářů
  - Ke každé entitě doplněn popis (z DS/slovníku pojmů) a reference na konkrétní UC/UI stránky tech. projektu
  - PP: 22 entit s popisy + referencemi (UC 200–500, integrace, synchronizace)
  - RP: 41 entit s popisy + referencemi (UC 100–800, UI spec, integrace FOB/FLWW2)
- [ ] Přepracovat podklady pro datové modely — v2 ← PRÁVĚ TADY
  - Postup: po entitách s review
  - Výstup: docs/datovy-model-PP.md a docs/datovy-model-RP.md
  - Nový kontext: DS je konceptuální (analytik), DDL je implementační (vývojář) — vědomé rozdíly jsou OK
  - Nový kontext: RP má dva paralelní světy (kontejnery vs. CS s vlastním UI a datovým modelem)
  - Referenční vstupy: inventáře entit (docs/inventar-entit-PP.md, docs/inventar-entit-RP.md) + výtah CK (docs/vytah-cilovy-koncept.md)
- [ ] Review datových modelů se stakeholdery
- [ ] Návrh cílových datových modelů pro dotčené systémy
- [ ] Mapování entit mezi systémy

### Konzultace
- [x] Zmapovat dostupnou referenční bázi (Confluence PP 300+ stránek, RP 655 stránek, cílový koncept)
- [ ] (ad-hoc — výstupy průběžně do docs/)

### Průřezové
- [x] Rozšířit scope projektu z "Integrace" na "Analýza" (integrace + datové modely + konzultace)
- [x] Zmapovat kompletní strukturu data/input/ a aktualizovat input-analysis.md

## Klíčová rozhodnutí
### 2026-02-23
- **KR-01: Rozšíření scope projektu** — Projekt přejmenován z "CS-MPG Integrace" na "CS-MPG Analýza". Scope rozšířen o dvě nové oblasti: návrh datových modelů a konzultační podpora pro tým. Promítnuto do CLAUDE.md, SPEC.md, PLAN.md.
- **KR-02: Dvouvrstvý přístup k analýze datových modelů** — Analýza dopadů na datový model bude důsledně rozlišovat entitní model (datový slovník — business atributy, asociace, pravidla) a fyzický model (DDL — tabulky, sloupce, FK, indexy). Datové slovníky z technických projektů PP a RP jsou primárním zdrojem pro logickou vrstvu.
- **KR-03: Nalezené bugy v DDL** — Identifikovány 4 potenciální chyby v produkčních DDL, které je třeba vyřešit před implementací CS: `address.updated_by` je `bit` místo `int FK` (PP), `address.city` je `nvarchar(50)` vs. DS specifikace 80 znaků (PP), `liquidation_garbage_types` uložen jako JSON místo proper FK (RP), `time_from/time_to` je `varchar(150)` na OS (RP).
- **KR-04: Archiv = nepřebírat** — Materiály v docs/archive/ byly archivovány cíleně (DS↔DDL mapování vzniklo před doplněním doménového kontextu). Entitní analýzu zahájit od nuly s čistými vstupy (výtah CK + DS přímo + DDL přímo).
- **KR-05: Výtah CK je nutný, ale ne dostatečný podklad** — Pro datový model a integrace slouží jako business reference. Chybí atributové detaily entit, přesné integrační zprávy a obsah diagramů. Doplňuje se z DS a DDL při entitní analýze.

## Otevřené otázky
- OQ-01: Seznam entit MDB vs. REST API v Etapě 1 — **částečně** (DDL znám, rozřazení per entita TBD)
- ~~OQ-02: RP-sync-external-Pasport~~ → **Ne, až Etapa 2+**
- OQ-03: Potvrzení OAuth2 client-credentials — **otevřeno**, nepotvrzeno
- OQ-04: Testovací přístupy do HEN — **otevřeno, blocker**, nevyřešeno
- OQ-05: Paralelní plánování DV — **otevřeno**, RADIUM preferuje výhradně RP, zákazník možná prosadí paralelní
- ~~OQ-06: Komunikační protokol FOB 2.0~~ → **mimo scope** (řeší M. Taraba)
- ~~OQ-07: OpenAPI specifikace~~ → **existují**, dodat jako vstup
- OQ-08: SLA (RPO/RTO) — **částečně** (obecné SLA platformy, per tok navrhneme)
- ~~OQ-09: Limity payloadu~~ → **vyřeší implementace**
- ~~OQ-10: Verzování API~~ → **explicitní NFR požadavek**, detail na implementaci

## Session log
### 2026-02-11, session 1
- Projekt založen
- Další krok: vyplnit specifikaci

### 2026-02-11, session 2
- Analyzovány 2 vstupní dokumenty (PRD + HLC kapitola komunikace)
- Vytvořen data/input-analysis.md (strukturovaný výtah)
- Naplněn SPEC.md (6 FR, 5 NFR, scope, omezení, AC)
- Vytvořen PLAN.md (4 fáze)
- Identifikováno 6 nejasností, 10 otevřených otázek
- Další krok: řešení otevřených otázek, zahájení detailní specifikace toků

### 2026-02-11, session 3
- Přidán a analyzován DDL soubor MDB schématu (`MDB_schema_DDL.txt`)
- Identifikováno: 22 business entit (BCED_*), 6 candidate tabulek, 11 pasport sync + 13 roadplan sync tabulek
- Klíčové poznatky: WinyXId (legacy), composite PK na ContractItem, candidate workflow, asymetrie pasport vs. roadplan sync
- Aktualizován data/input-analysis.md (dokument #3, nová sekce 9 s detailní analýzou schématu)
- Částečně zodpovězena OQ-01 (seznam entit na MDB je nyní znám)
- Projity všechny OQ (OQ-01 až OQ-10): 5 uzavřeno, 4 částečně/otevřeno, 1 mimo scope
- Klíčová rozhodnutí: OQ-02 (RP→PP ne v Etapě 1), OQ-06 (FOB mimo scope), OQ-10 (verzování = NFR)
- Zúžen scope SPEC.md na 4 toky (bez RP↔FOB a RP→PP zápis)
- Přidán NFR-06 (verzování API)
- Zbývá otevřeno: OQ-03 (auth), OQ-04 (HEN testenv, blocker), OQ-05 (plánování DV, kritická)
- Další krok: dodat OpenAPI specs stávajících služeb, validace SoT matice, zahájení detailní specifikace toků

### 2026-02-23, session 4
- **Rozšířen scope projektu** z "CS-MPG Integrace" na "CS-MPG Analýza"
  - Nové oblasti: datové modely + konzultační podpora pro tým
  - Aktualizovány: CLAUDE.md, SPEC.md, PLAN.md, PROGRESS.md
  - SPEC.md: nové cíle (4–7), rozšířený in-scope, nová AC (AC-06 až AC-08)
  - PLAN.md: nová Fáze 4 (Datové modely) + průběžná sekce Konzultace
- **Zmapována kompletní struktura data/input/**
  - Identifikováno 8 podkladů ve 4 složkách (~166 MB, ~2 576 souborů)
  - Nově zachyceny: DDL PP (169 KB), DDL RP (125 KB), Confluence export PP (300+ stránek), Confluence export RP (655 stránek), Cílový koncept.docx (11 MB)
  - Každý podklad přiřazen k oblasti projektu (integrace/datové modely/konzultace)
- **Restrukturován data/input-analysis.md**
  - Přidána sekce 1: přehled vstupních podkladů (strom, inventář, využití per oblast)
  - Přidány detailní popisy nových dokumentů (dok. 2–6)
  - Stávající obsah zachován, přečíslován
- Další krok: dodat OpenAPI specs, validace SoT matice, zahájení detailní specifikace toků

### 2026-02-23, session 5
- **Zpracován Cílový koncept** (Příloha č. 3_Cílový koncept.docx)
  - Extrahován kompletní text (.docx → Python) a 12 tabulek (verze, pojmy, aktéři, parametry SO, frekvence, ...)
- **Vytvořena analýza Cílový koncept vs. DDL** (`docs/analyza-cilovy-koncept-vs-ddl.md`)
  - PP: 11 požadavků (A1–A11) — 6 nových tabulek, 6 nových sloupců, 1 tabulka Etapa 2
  - RP: 13 požadavků (B1–B13) — ~8 nových tabulek Etapa 1, ~4 tabulek Etapa 2, 11+ nových sloupců
  - Zcela nové entity bez DDL protějšku: Okruh, Rozvrh, Zóna, Okruh dne
  - Klíčový nález: PP `address` nemá lat/lng (RP ano), sdílené číselníky (garbage_group, circuit, schedule) neexistují v RP
- **Nalezeny a potvrzeny DDL soubory a datové slovníky**
  - DDL: `DDL_PP_DB.txt` (PasPort_MPGSK), `DDL_RP_DB.txt` (RoadPlan_MPGSK)
  - DS: `datovy-88033012.html` (PP, 77 entit), `datovy-78681044.html` (RP, 88 entit)
  - Slovníky pojmů: `slovnik-88032903.html` (PP), `slovnik-78680658.html` (RP)
  - PP persistence file (`persistence-88033014.html`) je prázdný placeholder
- **Identifikován gap v analýze** — datové slovníky nebyly plnohodnotně využity (96k+ tokenů per soubor)
  - DS obsahují business popisy atributů, asociace, povinnosti, pravidla
  - DS-PP má sloupec `Persistentní reference` pro mapování na DDL
- **Vytvořen plán doplnění** (`docs/PLAN-doplneni-analyzy-datovy-model.md`)
  - 5 kroků: extrakce 24 entit z DS → mapování DS↔DDL → revize 24 požadavků do dvouvrstvého formátu → nesrovnalosti → konsolidace
  - Výstup: přepracovaná analýza rozlišující entitní model vs. fyzický model
- Klíčové rozhodnutí: KR-02 (dvouvrstvý přístup — entitní + fyzický model)
- Další krok: realizace plánu doplnění analýzy (navazující session)

### 2026-02-23, session 6
- **Realizován kompletní plán doplnění analýzy** (`docs/PLAN-doplneni-analyzy-datovy-model.md` — všech 5 kroků)
  - Krok 1+2: Extrakce 13 PP entit a 11 RP entit z datových slovníků + mapování na DDL
    - `docs/extrakce-PP-entity-a-mapovani.md` (1 100 řádků)
    - `docs/extrakce-RP-entity-a-mapovani.md` (972 řádků)
  - Krok 3: Revize všech 24 požadavků (A1–A11, B1–B13) do dvouvrstvého formátu
    - `docs/revize-pozadavku-dvouvrstva.md` (914 řádků)
  - Krok 4: Identifikace nesrovnalostí DS vs. DDL
    - `docs/nesrovnalosti-DS-vs-DDL.md` (181 řádků)
    - 12 atributů DS bez DDL, 28 DDL sloupců bez DS, 15 typových nesrovnalostí, 4 potenciální bugy
  - Krok 5: Konsolidace do finální dvouvrstvé analýzy
    - `docs/analyza-cilovy-koncept-vs-ddl.md` přepsán na 1 185 řádků (z původních 535)
    - Nová struktura: 6 sekcí (metodika, PP entity+požadavky, RP entity+požadavky, meziaplikační vazby, nesrovnalosti, rizika)
- **Vytvořeny 4 ER diagramy** (HTML/SVG):
  - `docs/er-diagram-PP.html` — detailní ER s atributy (Mermaid)
  - `docs/er-diagram-RP.html` — detailní ER s atributy (Mermaid)
  - `docs/er-konceptualni-PP.html` — konceptuální ER bez atributů (SVG, pro BA)
  - `docs/er-konceptualni-RP.html` — konceptuální ER bez atributů (SVG, pro BA)
- **Klíčová zjištění:**
  - PP: `address.updated_by` je `bit` místo `int FK` — téměř jistě bug
  - PP: `container_type` chybí `organization_unit_id` a `Oblast použití` (DS je definuje, DDL ne)
  - RP: `liquidation_garbage_types` uložen jako JSON místo proper entity referencí
  - RP: `transport_type` chybí `Technická specifikace` (JSON) — kritické pro přidání nového typu CS
  - RP: 20 DDL sloupců bez dokumentace v DS (legacy WinyX ID, konfirmační pole)
- Rozhodnutí: KR-03 (nalezené DDL bugy)
- Další krok: review datových modelů se stakeholdery, zahájení detailní specifikace integračních toků

### 2026-02-23, session 8
- **Vytvořen strukturovaný výtah z Cílového konceptu** (`docs/vytah-cilovy-koncept.md`, 1 165 řádků)
  - Extrahován .docx do MD (Python, 1 377 řádků surového textu)
  - Rozděleno do 4 sekcí, zpracováno 4 paralelními agenty se sdíleným zadáním
  - Konsolidováno do jednoho konzistentního dokumentu se zaměřením na datový model a integraci
  - Zachovány odkazy na kapitoly CK (blockcitace), entity tučně, tabulky v MD formátu
  - Sekce: Manažerské shrnutí, Business procesy, Doménový model, Etapizace, Aplikace PP/RP/FOB, Komunikace mezi systémy, Strategická optimalizace, Architektura, NFR, Rizika, Přílohy (slovník, aktéři, parametry SP, otevřené body)
- **Klíčové poznatky z extrakce:**
  - Kardinality vazeb RPO: RPO↔Okruh M:N, RPO↔Rozvrh M:N, RPO↔Zóna 1:1
  - Stavový automat okruhů v FOB: 5 stavů, definované zprávy do RP (Zahájit/Ukončit/Přerušit → odesílá se; Vynechat/Zařadit → lokální)
  - Třístavový model odbavení stanovišť: Červená (neodbaveno) → Žlutá (odbaveno FOB) → Zelená (potvrzeno RP)
  - Generování OS: 4 scénáře dle vazeb RPO (Rozvrh je nutná podmínka)
  - SoT okruhy: Etapa 1 = HEN, Etapa 2 = PP
  - 16 otevřených bodů z CK s odpověďmi od zákazníka
- Další krok: zahájit entitní analýzu PP — první entita (RPO) s review, výstup do docs/datovy-model-PP.md

### 2026-02-23, session 9
- **Review výtahu z Cílového konceptu** (`docs/vytah-cilovy-koncept.md`)
  - Posouzeno, že výtah splňuje účel jako business reference pro datový model a integrace
  - Identifikováno 6 mezer: chybí atributové detaily entit, integrační zprávy bez atributové úrovně, diagramy jen referované, lifecycle/stavy ne všech entit, validační pravidla mimo scope CK, datové typy nespecifikovány
  - Mezery se budou řešit při entitní analýze z DS a DDL
- **Stanoveno pravidlo: z docs/archive/ nepřebírat** (KR-04)
  - DS↔DDL mapování v archivu vzniklo před doménovým kontextem o vztahu DS vs DDL
  - Entitní analýzu zahájit od nuly s čistými vstupy
- **Vytvořeny inventáře entit DS + DDL** (čistý inventář bez hodnocení)
  - `docs/inventar-entit-PP.md` — 22 existujících entit + 4 nové pro CS (Okruh, Rozvrh, Kalendář, Zóna)
  - `docs/inventar-entit-RP.md` — 31 hlavních entit + 9 enumerací + 5 nových pro CS (Okruh, Rozvrh, Okruh dne, Skupina odpadu, RPO)
  - Ke každé entitě: atributy z DS, sloupce z DDL, asociace z DS (oddělené, bez mapování)
- **Doplněny popisy a reference z tech. projektů** do obou inventářů
  - Popis entity (z DS/slovníku pojmů) jako blockquote
  - Reference na konkrétní UC/UI stránky technického projektu (PP: UC 200–500; RP: UC 100–800)
- Klíčová rozhodnutí: KR-04 (archiv nepřebírat), KR-05 (výtah CK nutný ale ne dostatečný)
- Další krok: zahájit entitní analýzu PP — první entita (RPO) s review, výstup do docs/datovy-model-PP.md
