# Plán: Doplnění analýzy Cílový koncept vs. datové modely PP a RP

**Datum vytvoření:** 2026-02-23
**Kontext:** Existující analýza `docs/analyza-cilovy-koncept-vs-ddl.md` byla zpracována primárně na základě DDL souborů (fyzický model). Datové slovníky z technických projektů PP a RP nebyly plnohodnotně využity. Tento plán řeší jejich systematické zapracování.

**Cíl:** Doplnit analýzu o vrstvu entitního (logického) modelu z datových slovníků a důsledně rozlišovat:
- **Entitní model** (datový slovník) — business atributy, asociace, popisy, pravidla
- **Fyzický model** (DDL) — tabulky, sloupce, datové typy, FK, indexy

---

## Zdroje

| Zkratka | Soubor | Obsah | Velikost |
|---------|--------|-------|----------|
| **DS-PP** | `dokumentace_PP/.../datovy-88033012.html` | Datový slovník PP — 77 entit, atributy s popisy, asociace, povinnosti | ~97k tokenů |
| **PS-PP** | `dokumentace_PP/.../persistence-88033014.html` | Persistence PP — prázdný placeholder | — |
| **DS-RP** | `dokumentace_RP/.../datovy-78681044.html` | Datový slovník RP — 88 entit, atributy s popisy, asociace, stavy | ~97k tokenů |
| **DDL-PP** | `dokumentace_PP/DDL_PP_DB.txt` | Fyzický model PP (PasPort_MPGSK) | ~1500 řádků |
| **DDL-RP** | `dokumentace_RP/DDL_RP_DB.txt` | Fyzický model RP (RoadPlan_MPGSK) | ~2000+ řádků |
| **CK** | `cilovy_koncept_CS/Příloha č. 3_Cílový koncept.docx` | Cílový koncept CS — požadavky | Zpracováno |

### Struktura datových slovníků (DS-PP, DS-RP)
Každá entita dokumentována jako tabulka se sloupci:
- `Atribut` | `Datový typ` | `Rozsah` | `Asociace` | `Povinnost` | `Popis` | `Persistentní reference` (PP) | `Poznámka`

Persistentní reference mapuje atribut entity na fyzický sloupec v DDL (klíč pro propojení logického a fyzického modelu).

---

## Rozsah prací

### Krok 1: Extrakce klíčových entit z datových slovníků

Pro každou entitu dotčenou požadavky z Cílového konceptu extrahovat kompletní atributový profil z datového slovníku.

**PP entity k extrakci (z DS-PP):**

| # | Entita v DS-PP | Priorita | Důvod |
|---|----------------|----------|-------|
| 1 | Revize položky objednávky (RPO) | Kritická | Centrální entita — nové vazby na okruh, rozvrh, zónu, skupinu odpadu |
| 2 | Položka objednávky | Vysoká | Nadřazená entita RPO, grouping revizí |
| 3 | Objednávka | Střední | Kontejner pro položky, vazba na zákazníka |
| 4 | Stanoviště | Vysoká | Souřadnice pro lokalizaci OS v RP |
| 5 | Nádoba | Vysoká | Vazba na skupinu odpadu, přiřazení ke stanovišti a RPO |
| 6 | Skupina odpadu | Kritická | Nový atribut na RPO, sdílený číselník PP↔RP |
| 7 | Druh odpadu | Vysoká | Mapování druh → skupina |
| 8 | Typ nádoby | Vysoká | Nový atribut čas obsluhy |
| 9 | Frekvence vývozu | Střední | Kontext pro rozvrhy |
| 10 | Adresa | Vysoká | Chybějící lat/lng v PP |
| 11 | Přiřazení nádoby ke stanovišti | Střední | Kontext pro lokalizaci OS |
| 12 | Přiřazení nádoby k položce objednávky | Střední | Kontext pro generování OS |
| 13 | Přiřazení dní svozu k RPO | Vysoká | Stávající mechanismus vs. nové Rozvrhy |

**RP entity k extrakci (z DS-RP):**

| # | Entita v DS-RP | Priorita | Důvod |
|---|----------------|----------|-------|
| 1 | Objednaná služba | Kritická | Hlavní entita pro CS — rozšíření vs. nová tabulka |
| 2 | Lokace objednané služby | Kritická | Lokace pro CS — nové atributy |
| 3 | Denní výkon | Vysoká | Nový transport_type, vazba na okruhy dne |
| 4 | Položka denního výkonu | Vysoká | Nový typ položky (okruh dne) |
| 5 | Vozidlo | Vysoká | Nové kapacitní atributy pro SP |
| 6 | Likvidační místo | Střední | Vazba na skupinu odpadu |
| 7 | Druh odpadu lik. místa | Střední | Aktuální vazby LM↔odpady |
| 8 | Typ dopravy | Střední | Nový typ pro CS |
| 9 | Typ položky denního výkonu | Vysoká | Nový kód pro okruh dne |
| 10 | Stav objednané služby | Střední | Stavový automat pro CS OS |
| 11 | Místo realizace (execution_site) | Střední | Alternativa k lokaci z PP |

**Technika:** HTML parsing (Python) → extrakce tabulek per entita → Markdown výstup.

---

### Krok 2: Mapování entitní model ↔ fyzický model

Pro každou extrahovanou entitu vytvořit mapovací tabulku:

```
| Atribut (DS) | Datový typ (DS) | Sloupec (DDL) | Typ (DDL) | Shoda | Poznámka |
```

Cíl:
- Potvrdit, že každý logický atribut má odpovídající sloupec v DDL
- Identifikovat **sloupce v DDL, které nemají popis v datovém slovníku** (technické/legacy sloupce)
- Identifikovat **atributy v DS, které chybí v DDL** (nesynchronizované, budoucí)

Využít sloupec `Persistentní reference` z DS-PP jako explicitní mapování tam, kde je dostupný.

---

### Krok 3: Revize požadavků — rozlišení dopadu na entitní vs. fyzický model

Přepracovat každý požadavek z existující analýzy (A1–A11, B1–B13) do dvouvrstvé struktury:

**Nový formát pro každý požadavek:**

```markdown
### Xx. [Název požadavku]

**Požadavek:** [text z CK]

**Dopad na entitní model (datový slovník):**
- Nová entita / rozšíření entity
- Nové atributy: název, datový typ, rozsah, asociace, povinnost, popis
- Nové/změněné asociace mezi entitami
- Business pravidla a omezení

**Dopad na fyzický model (DDL):**
- Nová tabulka / ALTER TABLE
- Nové sloupce: název, SQL typ, nullable, FK, default
- Nové indexy, constrainty
- Migrační dopady na existující data

**Současný stav:**
- Entitní model (DS): [co DS říká o dané entitě dnes]
- Fyzický model (DDL): [co DDL ukazuje dnes]
- Delta: [co chybí, co je navíc]
```

---

### Krok 4: Identifikace nesrovnalostí DS vs. DDL

Specificky hledat:
1. **Atributy v DS, které nejsou v DDL** — indikátor rozpracovanosti nebo chybějící migrace
2. **Sloupce v DDL, které nejsou v DS** — technické sloupce, legacy, nebo nedokumentované rozšíření
3. **Typové nesrovnalosti** — DS říká "Celé číslo", DDL má `nvarchar`
4. **Asociace v DS bez FK v DDL** — logická vazba bez fyzického enforced constraint

Tyto nesrovnalosti zachytit do separátní sekce výstupu.

---

### Krok 5: Konsolidace do výstupního dokumentu

Přepracovat `docs/analyza-cilovy-koncept-vs-ddl.md` na finální verzi:

**Navržená struktura výstupního dokumentu:**

```
# Analýza: Cílový koncept CS vs. datové modely PP a RP

## 0. Metodika a zdroje
  - Popis zdrojů (CK, DS-PP, DS-RP, DDL-PP, DDL-RP)
  - Vysvětlení dvou vrstev (entitní vs. fyzický model)

## 1. PASPORT (PP)
  ### 1.1 Současný stav — klíčové entity (z DS-PP + DDL-PP)
    Pro každou entity: atributy, mapování DS↔DDL, identifikované delty
  ### 1.2 Požadavky z CK s dopadem na PP model
    A1–A11 v novém formátu (entitní + fyzický dopad)
  ### 1.3 Souhrnná tabulka změn PP

## 2. ROADPLAN (RP)
  ### 2.1 Současný stav — klíčové entity (z DS-RP + DDL-RP)
  ### 2.2 Požadavky z CK s dopadem na RP model
    B1–B13 v novém formátu
  ### 2.3 Souhrnná tabulka změn RP

## 3. Meziaplikační vazby (PP ↔ RP)
  - Mapování entit mezi systémy
  - Sdílené číselníky a jejich synchronizace

## 4. Nesrovnalosti DS vs. DDL
  - PP: identifikované delty
  - RP: identifikované delty

## 5. Integrační dopady a rizika
  (přeneseno z existující analýzy, sekce C a D)
```

---

## Odhad pracnosti

| Krok | Popis | Náročnost |
|------|-------|-----------|
| 1 | Extrakce entit z DS-PP (13 entit) a DS-RP (11 entit) | Střední — HTML parsing, ~24 entit |
| 2 | Mapování DS ↔ DDL per entita | Střední — manuální cross-reference |
| 3 | Revize 24 požadavků (A1–A11, B1–B13) do nového formátu | Vysoká — hlavní analytická práce |
| 4 | Identifikace nesrovnalostí | Nízká — vedlejší produkt kroku 2 |
| 5 | Konsolidace do finálního dokumentu | Střední — redakce a strukturování |

**Doporučený postup:** Kroky 1+2 paralelně pro PP a RP (dva agenti), pak krok 3 sekvenčně, krok 4 jako vedlejší produkt, krok 5 na závěr.

---

## Předpoklady a omezení

- **PS-PP (persistence)** je prázdný soubor — nepoužitelný. Mapování DS↔DDL bude primárně přes sloupec `Persistentní reference` v DS-PP a přes naming convention.
- Datové slovníky reflektují **aktuální stav** aplikací (ne budoucí stav po implementaci CS). Nové entity (okruh, rozvrh, zóna, okruh dne) v nich pochopitelně nejsou.
- DDL soubory obsahují i TMP/BCKUP tabulky — ty ignorujeme, zajímají nás jen produkční tabulky.

---

## Návaznost

Výstup tohoto plánu (doplněná analýza) bude vstupem pro:
- **PLAN.md / Fáze 4** — Review/návrh datových modelů pro PP a RP
- **PLAN.md / Fáze 2** — Datové kontrakty pro integrační toky (INT-01 až INT-06)
- Konzultační podpora pro vývojový tým při implementaci
