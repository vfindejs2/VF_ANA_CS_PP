# Progress: CS-MPG Integrace

## Aktuální stav
Fáze: Vstupní analýza dokončena, MDB schéma analyzováno
Poslední session: 2026-02-11
Další krok: Dodat OpenAPI specs stávajících služeb, validace SoT matice, zahájení detailní specifikace toků

## TODO
- [x] Vyplnit SPEC.md
- [x] Definovat plán prací (PLAN.md)
- [x] Analyzovat vstupní dokumenty (data/input-analysis.md)
- [x] Analyzovat MDB schéma (DDL) — 22 BCED_ entit, 6 candidate tabulek, sync tracking
- [x] Doplnit odpovědi na otevřené otázky (OQ-01 až OQ-10) — 5 uzavřeno, 4 částečně/otevřeno, 1 mimo scope
- [ ] Validace SoT matice se stakeholdery
- [ ] Detailní specifikace integračních toků (INT-01 až INT-06)
- [ ] Návrh integrační architektury
- [ ] Business zadání per integrační služba

## Klíčová rozhodnutí
(zatím žádná)

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
