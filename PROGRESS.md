# Progress: CS-MPG Analýza

## Aktuální stav
Fáze: Analýza datových modelů — první průchod hotov, plán na doplnění z datových slovníků připraven
Poslední session: 2026-02-23 (session 5)
Další krok: Doplnit analýzu o datové slovníky dle plánu `docs/PLAN-doplneni-analyzy-datovy-model.md` (krok 1+2 → 3 → 5)

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
- [ ] Doplnit analýzu o datové slovníky (entitní model) ← PRÁVĚ TADY
  - Plán: docs/PLAN-doplneni-analyzy-datovy-model.md
  - 5 kroků: extrakce entit → mapování DS↔DDL → revize požadavků → nesrovnalosti → konsolidace
- [ ] Review/návrh datových modelů pro dotčené systémy
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
