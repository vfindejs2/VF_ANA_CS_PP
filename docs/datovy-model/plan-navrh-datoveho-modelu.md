# Plán návrhu datového modelu CS

> **Účel:** Checklist entit ke zpracování pro `docs/datovy-model/datovy-model-PP.md` a `docs/datovy-model/datovy-model-RP.md`.
> **Šablona:** `docs/datovy-model/sablona-entitni-analyza.md`
> **Vytvořeno:** 2026-02-23
> **Princip řazení:** Závislosti první — entity, na které odkazují jiné, se analyzují dřív.

---

## PP — Pasport (10 entit)

| # | Entita | Stav | Proč | Závisí na | Analýza |
|---|---|---|---|---|---|
| PP-1 | **Skupina odpadu** | Rozšíření | Nová role pro CS — řídí přiřazení na RPO, klíč pro okruh dne | — | [ ] |
| PP-2 | **Druh odpadu** | Rozšíření | Nová asociace → Skupina odpadu (mapování) | PP-1 | [ ] |
| PP-3 | **Zóna** | Nová | Geografické zónování, 1:1 vazba z RPO | — | [ ] |
| PP-4 | **Rozvrh** | Nová | Kalendářní dny obsluhy, M:N vazba z RPO | — | [ ] |
| PP-5 | **Kalendář** | Nová | Konkrétní den obsluhy, vazba na Rozvrh | PP-4 | [ ] |
| PP-6 | **Okruh** | Nová | Sdružuje RPO obsluhované společně, M:N z RPO | — | [ ] |
| PP-7 | **Typ nádoby** | Rozšíření | Nový atribut: čas obsluhy (sekundy) | — | [ ] |
| PP-8 | **RPO** | Rozšíření | Centrální entita CS — nové vazby na Okruh, Rozvrh, Zóna, Skupina odpadu | PP-1..6 | [ ] |
| PP-9 | **Nádoba** | Rozšíření | Dědí z RPO (okruh, rozvrh, zóna), skupina odpadu řízena RPO | PP-8 | [ ] |
| PP-10 | **Stanoviště** | Rozšíření | Dědí z RPO (okruh, rozvrh, zóna), lokalizace OS v RP | PP-8 | [ ] |

## RP — RoadPlan (9 entit)

| # | Entita | Stav | Proč | Závisí na | Analýza |
|---|---|---|---|---|---|
| RP-1 | **Skupina odpadu** | Nová/sdílená | Seskupení druhů odpadu, klíč pro okruh dne (1 okruh dne = 1 skupina) | — | [ ] |
| RP-2 | **Okruh** | Nová | Definice svozové trasy, vazba na vozidlo | — | [ ] |
| RP-3 | **Rozvrh** | Nová | Rozvrh obsluh, podmínka pro generování OS | — | [ ] |
| RP-4 | **RPO** | Nová (import z PP) | Importovaná podmnožina dat z PP, minimální atributová sada | RP-1..3 | [ ] |
| RP-5 | **Okruh dne** | Nová | Soubor OS pro konkrétní den, stavový automat, TOD | RP-2, RP-3, RP-4 | [ ] |
| RP-6 | **Objednaná služba** | Rozšíření | Nové vazby: Okruh dne, RPO; generování z RPO | RP-4, RP-5 | [ ] |
| RP-7 | **Typ položky DV** | Rozšíření (enum) | Nová hodnota: „okruh dne" | RP-5 | [ ] |
| RP-8 | **Denní výkon** | Rozšíření | Separátní CS záložka, plánování přes okruhy dne | RP-5 | [ ] |
| RP-9 | **Realizace DV** | Rozšíření | Stavy okruhů z FOB, odbavení lokací OS | RP-8 | [ ] |

---

## Poznámky

- **Sdílené entity** (Skupina odpadu, Okruh, Rozvrh, RPO) existují v obou aplikacích — analyzují se separátně per aplikaci s odkazem na protějšek
- **PP se zpracovává první** — je datovým zdrojem pro RP
- **Postup:** entita po entitě s review před pokračováním na další
- **Výstup:** `docs/datovy-model/datovy-model-PP.md` a `docs/datovy-model/datovy-model-RP.md`
