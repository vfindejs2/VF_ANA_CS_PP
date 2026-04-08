# Progress: CS-MPG-ANALYZY-E1

## Aktuální stav
Fáze: Návrh datového modelu — kompletní BA návrh PP v2 dokončen (16 entit)
Poslední session: 2026-04-07 (session 13)
Další krok: Review návrhu se stakeholdery, potvrzení otevřených bodů

## TODO

### Datové modely (aktuální priorita)
- [x] BA návrh datového modelu PP pro CS v2 → `02-Datovy-model-PP/PP-datovy-model-navrh-reseni-v1/datovy-model-navrh-reseni-123372122.md` — **16/16 entit**
  - [x] Gap analýza (11 mezer) zapracována
  - [x] 3 architektonická rozhodnutí (AR-01 až AR-03)
  - [x] SoT per entita per etapa
  - [x] external_id strategie
  - [x] 2 nové entity (Zóna položky, Región)
- [ ] Review BA návrhu PP se stakeholdery ← DALŠÍ KROK
- [ ] Potvrzení otevřených bodů (viz sekce v dokumentu)
- [ ] Entitní analýza RP (9 entit)
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
| KR-11 | Zóna = adresní hierarchie + Zóna položky | Geometrie odebrána z DS (AR-02), přidána entita Zóna položky pro HEN adresní definici |
| KR-12 | HEN ZVOZ Guide jako vstup | Vstup #5 do strategie DM, referovat od PP-3 |
| KR-13 | Zachovat RPO_Okruh_Rozvrh (AR-01) | PP konsoliduje dva HEN modely do jedné temporální tabulky |
| KR-14 | Rozvrh — nullable parametry frekvence (AR-03) | Denormalizovaný přístup, 3 typy frekvence (Denne/Tyzdenne/Vlastne) |
| KR-15 | external_id strategie | Každá synchronizovaná entita má external_id (nvarchar) jako integrační klíč z HEN |

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
| PP-3 | Zóna | Navrženo (v2) | Rozšířeno o region_id, stav, external_id. Geometrie odebrána (AR-02). |
| PP-4 | Rozvrh | Navrženo (v2) | Přidáno frekvence + nullable parametry (AR-03), external_id. |
| PP-5 | Kalendar | Navrženo (v2) | Přidáno 8 atributů z HEN, vyvoz místo typ_dne, external_id. |
| PP-6 | Okruh | Navrženo (v2) | Přidáno referencia, poznamka, external_id. vozidlo_id zdůvodněno. |
| PP-7 | Typ nádoby | Existující + SoT | Rozšíření o cas_obsluhy_sec (editovatelný v PP). |
| PP-8 | RPO | Navrženo (v2) | Přidáno external_id (klíčový integrační předpoklad), SoT per etapa. |
| PP-9 | Nádoba | Existující + SoT | Aplikační pravidla priority skupina_odpadu_id. |
| PP-10 | Stanoviště | Existující + SoT | Beze změn pro CS. |
| — | Adresy | Existující + SoT | Rozšíření o X, Y souřadnice. |
| — | RPO_Okruh_Rozvrh | Navrženo (v2) | Zachován PP model dle AR-01, SoT per etapa. |
| — | Nadoba_Stanoviste | Existující + SoT | Beze změn pro CS. |
| — | Skupina_Druh_odpadu | Navrženo + SoT | Temporální mapování druh → skupina. |
| — | Zóna položky | **Nová** | Adresní definice zóny z HEN (dle AR-02). |
| — | Región | **Nový** | Číselník pro kategorizaci zón z HEN. |

## Session log (komprimovaný)

Sessions 1–3 (2026-02-11): Založení projektu, analýza vstupních dokumentů, SPEC.md, MDB DDL analýza.
Sessions 4–6 (2026-02-23): Rozšíření scope, CK vs. DDL analýza, dvouvrstvá analýza, ER diagramy, nalezené DDL bugy.
Session 8: Strukturovaný výtah z CK (1 165 řádků).
Session 9: Inventáře entit PP (22+4) a RP (31+9+5), popisy a reference z tech. projektů.
Session 10: Strategie DM, šablona, plán 19 entit, reorganizace docs/.
Session 11: Entitní analýza PP zahájena — PP-1 (Skupina odpadu), PP-2 (Druh odpadu), PP-3 (Zóna). HEN guide jako vstup.
Session 12: PP-4 (Rozvrh) — vstupy a otázky předloženy. Změna přístupu: nejdřív vstupy+otázky, pak návrh. Přejmenování projektu na CS-MPG-ANALYZY-E1.
Session 13 (2026-04-07): Kompletní přepracování BA návrhu PP v2. Přestrukturování workspace. Zapracování 11 mezer z gap analýzy, 3 architektonická rozhodnutí (AR-01–03), SoT per entita, external_id strategie, 2 nové entity (Zóna položky, Región). Výstup: 16 entit s kompletní BA analýzou.
