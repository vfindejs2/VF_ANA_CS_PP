# Progress: CS-MPG Analýza

## Aktuální stav
Fáze: Rozšířen scope projektu, zmapovány vstupní podklady
Poslední session: 2026-02-23
Další krok: Dodat OpenAPI specs stávajících služeb, validace SoT matice, zahájení detailní specifikace toků

## TODO
### Integrace
- [x] Vyplnit SPEC.md
- [x] Definovat plán prací (PLAN.md)
- [x] Analyzovat vstupní dokumenty (data/input-analysis.md)
- [x] Analyzovat MDB schéma (DDL) — 22 BCED_ entit, 6 candidate tabulek, sync tracking
- [x] Doplnit odpovědi na otevřené otázky (OQ-01 až OQ-10) — 5 uzavřeno, 4 částečně/otevřeno, 1 mimo scope
- [ ] Validace SoT matice se stakeholdery ← PRÁVĚ TADY
- [ ] Detailní specifikace integračních toků (INT-01 až INT-06)
- [ ] Návrh integrační architektury
- [ ] Business zadání per integrační služba

### Datové modely
- [x] Zmapovat vstupní podklady (DDL PP, DDL RP, MDB DDL, Confluence exporty)
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
