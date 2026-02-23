# Strategie návrhu datového modelu CS

> **Vytvořeno:** 2026-02-23

---

## Cíl

Navrhnout rozšíření datových modelů PP a RP pro podporu Cyklických svozů. Výstupem je návrh změn datového slovníku (DS) — business model, který analytik aplikace promítne do oficiální dokumentace.

## Vstupy

1. **Cílový koncept (CK)** — business požadavky, doménový model, etapizace, business pravidla
2. **Datové slovníky (DS)** — stávající konceptuální model (entity, atributy, asociace, pravidla)
3. **DDL** — stávající fyzický model (orientačně, pro ověření existence entit)
4. **Technické projekty PP/RP** — use cases, UI specifikace, integrační dokumentace

## Principy

- **DS je primární výstup.** Navrhujeme business model pro analytika. DDL doporučení jsou sekundární — developer má prostor pro implementační odchylky.
- **DS a DDL jsou záměrně odlišné vrstvy.** DS je konceptuální (analytik), DDL je implementační (vývojář). Odchylky v pojmenování, typech nebo technických sloupcích nejsou problém k řešení.
- **RP má dva paralelní světy.** Kontejnerová doprava a Cyklické svozy mají separátní UI i datové modely v rámci jedné aplikace. CS entity se navrhují samostatně.
- **Entita po entitě s review.** Každá entita se zpracuje, předloží ke schválení a teprve pak se pokračuje dál.

## Postup

1. **Strukturovaný výtah z CK** — business kontext zaměřený na datový model a integraci
2. **Inventáře entit** — čistý přehled DS atributů a DDL sloupců per entita, bez hodnocení a bez vzájemného mapování
3. **Entitní analýza** — pro každou dotčenou entitu dle šablony:
   - Business kontext z CK (proč se mění / proč je nová)
   - Návrh nových/změněných atributů a asociací pro DS
   - Doporučení pro DDL (sekundární)
   - Vazba na integrační toky
4. **Konsolidace** — výstupní dokumenty `docs/datovy-model/datovy-model-PP.md` a `docs/datovy-model/datovy-model-RP.md`

## Řazení entit

Závislosti první — entity, na které odkazují jiné, se analyzují dřív. PP před RP, protože PP je datovým zdrojem.

Konkrétní pořadí: `docs/datovy-model/plan-navrh-datoveho-modelu.md`

## Referenční dokumenty

| Dokument | Účel |
|---|---|
| `docs/datovy-model/vytah-cilovy-koncept.md` | Business kontext z CK |
| `docs/datovy-model/inventar-entit-PP.md` | Stávající entity PP (DS + DDL) |
| `docs/datovy-model/inventar-entit-RP.md` | Stávající entity RP (DS + DDL) |
| `docs/datovy-model/sablona-entitni-analyza.md` | Šablona pro zpracování entity |
| `docs/datovy-model/plan-navrh-datoveho-modelu.md` | Checklist entit s pořadím |
