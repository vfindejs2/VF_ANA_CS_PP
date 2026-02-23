# Projekt: CS-MPG Analýza

## Cíl
Business analýza a konzultační podpora projektu Cyklické svozy MP SK. Zahrnuje tři hlavní oblasti:
1. **Integrace** — analýza požadavků, návrh architektury a business zadání pro integrační toky mezi systémy
2. **Datové modely** — návrh a review datových modelů pro jednotlivé systémy
3. **Konzultace** — podpora zbytku týmu při návrhu dílčích funkčností

Samotná implementace není ve scope — výstupem jsou specifikace, zadání a doporučení.

## Typ projektu
business-analysis

## Skills pro tento projekt
Primární: business-analysis, product-requirements

## Klíčové soubory
- `SPEC.md` — zadání, požadavky, scope
- `PROGRESS.md` — aktuální stav, TODO, rozhodnutí

## Struktura projektu
- `docs/datovy-model/` — návrh datového modelu CS (strategie, inventáře, entitní analýzy)
- `docs/integrace/` — specifikace integračních toků
- `meetings/` — agendy a zápisy z jednání
- `data/input/` — původní vstupní dokumenty

## Data
- `data/input-analysis.md` — strukturovaný výtah ze vstupních dokumentů (generovaný přes /analyze-inputs)

## Reference z knowledge base
- Metodika: ~/second-brain/01-knowledge/methodologies/requirements-gathering.md
- Metodika: ~/second-brain/01-knowledge/methodologies/acceptance-criteria.md
- Tone of voice: ~/second-brain/01-knowledge/tone-of-voice/internal-cz.md
- Šablona: ~/second-brain/01-knowledge/templates/spec-template.md
- Glossář: ~/second-brain/01-knowledge/glossary/company-glossary.md

## Doménový kontext pro analýzu datových modelů

### Vztah datový slovník (DS) vs. fyzický model (DDL)
- **Datový slovník** je konceptuální model, vlastníkem je analytik aplikace. Definuje business entity, asociace a pravidla.
- **Fyzický model (DDL)** navrhuje vývojář při respektování business logiky. Má právo odlišit se v:
  - Pojmenování sloupců a tabulek
  - Datových typech (DS "Celé číslo" → DDL `float` může být záměr)
  - Přidání technických sloupců (`*_search`, `geom`, `winyx_id`, audit sloupce)
  - Vynechání DS atributů řešených na aplikační úrovni
- **Co je problém k nahlášení**: chybějící business entita nebo asociace v DS, která by tam být měla (DS není aktuální). Neaktuálnost DS z pohledu business logiky.
- **Co NENÍ problém**: implementační odchylky mezi DS a DDL (typ, pojmenování, technické sloupce).

### Architektura RoadPlan — dva paralelní světy
V aplikaci RoadPlan existují (budou existovat) dva paralelní a oddělené domény:
1. **Kontejnerová doprava** — stávající objednávky, objednané služby, denní výkony, plánování. Stávající UI a datový model.
2. **Cyklické svozy (CS)** — separátní UI i datové modely v rámci stejné aplikace. Pro CS:
   - Se z PP importují **položky objednávek a jejich revize (RPO)**, nádoby, stanoviště, okruhy, rozvrhy, skupiny odpadu
   - Vzniknou **vlastní objednané služby** (separátní od kontejnerových)
   - Vznikne **vlastní sestava plánu / denní výkon** (přes okruhy dne)
   - UI bude separátní (nové obrazovky, nový FOB modul)
- Důvod separace: malé nádoby (CS) a kontejnery mají zásadně odlišný způsob práce.
- Opora v CK: "datový model RP pro OS CS bude postaven tak, aby v nezbytné míře podporoval potřebnou funkčnost" + celý koncept okruhů dne jako samostatná struktura.

## Pravidla pro tuto session
- Před /clear vždy spusť /save-session
- Výstupy analýzy ukládej do `docs/`, zápisy z meetingů do `meetings/`
- Při komplikaci navrhni řešení, nerob změnu tiše
- Při analýze datových modelů: entity zpracovávej po jedné, ukazuj ke schválení, než pokračuješ dál
- Při entitní analýze: u každé entity se aktivně ptej na nejasnosti a business rozhodnutí, než finalizuješ návrh
