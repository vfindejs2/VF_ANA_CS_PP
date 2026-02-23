# Progress: CS-MPG-ANALYZY-E1

## Aktuální stav
Fáze: Návrh datového modelu — entitní analýza PP, 3/10 entit hotové
Poslední session: 2026-02-23 (session 12)
Další krok: PP-4 (Rozvrh) — otázky předloženy, čekám na odpovědi

## TODO

### Datové modely (aktuální priorita)
- [ ] Entitní analýza PP (10 entit) → `docs/datovy-model/datovy-model-PP.md` — **3/10 hotovo (PP-1, PP-2, PP-3)**
  - [ ] PP-4: Rozvrh ← PRÁVĚ TADY
  - [ ] PP-5: Kalendář
  - [ ] PP-6: Okruh
  - [ ] PP-7: Typ nádoby
  - [ ] PP-8: RPO
  - [ ] PP-9: Nádoba
  - [ ] PP-10: Stanoviště
- [ ] Entitní analýza RP (9 entit) → `docs/datovy-model/datovy-model-RP.md`
- [ ] Review datových modelů se stakeholdery
- [ ] Mapování entit mezi systémy

### Integrace (po dokončení DM)
- [ ] Validace SoT matice se stakeholdery
- [ ] Detailní specifikace integračních toků (INT-01 až INT-06)
- [ ] Návrh integrační architektury
- [ ] Business zadání per integrační služba

### Konzultace
- [ ] Ad-hoc (výstupy průběžně do docs/)

## Klíčová rozhodnutí

| # | Rozhodnutí | Popis |
|---|---|---|
| KR-01 | Rozšíření scope | Projekt = integrace + datové modely + konzultace |
| KR-02 | Dvouvrstvý přístup DM | DS (business) a DDL (fyzický) se analyzují odděleně |
| KR-03 | Bugy v DDL | 4 potenciální chyby v produkčních DDL identifikovány |
| KR-04 | Archiv nepřebírat | v1 DS↔DDL mapování smazáno, entitní analýza od nuly |
| KR-05 | Výtah CK nutný, ne dostatečný | Doplňuje se z DS a DDL při entitní analýze |
| KR-06 | Struktura projektu | docs/datovy-model/, docs/integrace/, meetings/ |
| KR-07 | DM first | Nejdřív datový model (19 entit), pak integrace |
| KR-08 | Mapování Druh→Skupina per provozovna | Nová vazební entita (Druh + Skupina + Provozovna), UNIQUE na (Druh, Provozovna) |
| KR-09 | Skupina MIX = datový záznam | Speciální chování řešeno aplikační logikou |
| KR-10 | Sdílení číselníků PP↔RP přes integraci | RP má vlastní kopii, ne sdílená DB |
| KR-11 | Zóna = jednoduchá entita | Bez replikace adresní struktury z HEN, v E1 se nepřenáší do RP |
| KR-12 | HEN ZVOZ Guide jako vstup | Vstup #5 do strategie DM, referovat od PP-3 |

## Otevřené otázky

| # | Otázka | Stav |
|---|---|---|
| OQ-01 | Seznam entit MDB vs. REST API v E1 | Částečně (DDL znám, rozřazení per entita TBD) |
| OQ-03 | Potvrzení OAuth2 client-credentials | Otevřeno |
| OQ-04 | Testovací přístupy do HEN | **Otevřeno, blocker** |
| OQ-05 | Paralelní plánování DV (HEN vs. RP) | Otevřeno |
| OQ-08 | SLA (RPO/RTO) per tok | Částečně |

## Hotové entity PP (souhrn)

| # | Entita | Stav | Klíčový výstup |
|---|---|---|---|
| PP-1 | Skupina odpadu | Navrženo | Beze změn DS. Nová vazební entita Mapování druh–skupina. |
| PP-2 | Druh odpadu | Navrženo | Beze změn DS. Funkcionalita na vazební entitě z PP-1. |
| PP-3 | Zóna | Navrženo | Nová jednoduchá entita (název, ext. ID, provozovna, poznámka, je aktivní). |

## Session log (komprimovaný)

Sessions 1–3 (2026-02-11): Založení projektu, analýza vstupních dokumentů, SPEC.md, MDB DDL analýza.
Sessions 4–6 (2026-02-23): Rozšíření scope, CK vs. DDL analýza, dvouvrstvá analýza, ER diagramy, nalezené DDL bugy.
Session 8: Strukturovaný výtah z CK (1 165 řádků).
Session 9: Inventáře entit PP (22+4) a RP (31+9+5), popisy a reference z tech. projektů.
Session 10: Strategie DM, šablona, plán 19 entit, reorganizace docs/.
Session 11: Entitní analýza PP zahájena — PP-1 (Skupina odpadu), PP-2 (Druh odpadu), PP-3 (Zóna). HEN guide jako vstup.
Session 12: PP-4 (Rozvrh) — vstupy a otázky předloženy. Změna přístupu: nejdřív vstupy+otázky, pak návrh. Přejmenování projektu na CS-MPG-ANALYZY-E1.
