# Plán prací: CS-MPG Integrace

## Fáze 1: Vstupní analýza
- [x] Sběr a analýza vstupních dokumentů
- [ ] Doplnění odpovědí na otevřené otázky (OQ-01 až OQ-10)
- [ ] Validace SoT matice se stakeholdery

## Fáze 2: Detailní specifikace integračních toků
- [ ] INT-01: HEN → PP (číselníky, vazby)
- [ ] INT-02: PP → RP (RPO, nádoby, stanoviště, ...)
- [ ] INT-03: RP → HEN (export DV pro doklady)
- [ ] INT-04: RP → FOB (DV + okruhy dne)
- [ ] INT-05: FOB → RP (události realizace)
- [ ] INT-06: RP → PP (dotaz na RPO pro generování OS)

Pro každý tok: účel, triggery, datový kontrakt, sekvence, chybové stavy, SLA, závislosti.

## Fáze 3: Integrační architektura
- [ ] Architektonický návrh (C4 diagram kontextu a kontejnerů)
- [ ] Přechodná architektura (MDB + REST koexistence)
- [ ] Bezpečnostní model (autentizace, autorizace)
- [ ] Observabilita a monitoring

## Fáze 4: Business zadání
- [ ] Sestavit business zadání per integrační služba
- [ ] Review a schválení stakeholdery
- [ ] Finální dokumentace do docs/
