# Datový model PP — Entitní analýza pro CS

> **Účel:** Návrh rozšíření datového slovníku PP pro podporu Cyklických svozů.
> **Šablona:** `docs/datovy-model/sablona-entitni-analyza.md`
> **Plán:** `docs/datovy-model/plan-navrh-datoveho-modelu.md`
> **Vytvořeno:** 2026-02-23

---

## 1. Skupina odpadu

**Stav:** Existující — rozšíření pro CS
**DS:** Skupina odpadu
**DDL:** `garbage_group`

### Business kontext

> Skupina odpadu je existující číselník v PP. Pro CS získává zásadně novou roli:
>
> - **Řídí přiřazení na RPO** — systém při synchronizaci RPO s předměty smluv HEN automaticky vyplní Skupinu odpadu na RPO dle mapování Druh odpadu → Skupina odpadu (CK: Aplikace Pasport → Přiřazení skupiny odpadu položce objednávky)
> - **Klíč pro plánování v RP** — jeden Okruh dne obsahuje OS pouze jedné skupiny odpadu (CK: Doména objednaných služeb a plánů)
> - **Podporuje mísitelnost** — druhy odpadu, které se mohou svážet společně (např. 150101 a 200101), sdílejí stejnou skupinu (CK: Aplikace Pasport → Druhy odpadu)
> - **Sdílený číselník PP ↔ RP** — nastavení vazby Druh ↔ Skupina možné pouze v PP; RP čerpá read-only kopii přes integraci (CK: Aplikace Pasport → Druhy odpadu)
>
> Entita sama o sobě zůstává globální (bez vazby na Provozovnu). Provozovní kontext vzniká přes novou vazební entitu Mapování druh–skupina (viz PP-2).

### Návrh rozšíření DS

#### Nové atributy

Entita Skupina odpadu nepotřebuje nové atributy. Stávající sada (Název, Zkrácený název, Popis, Barva, Externí identifikátor + systémové) je pro CS dostatečná.

#### Nové asociace

Na entitě Skupina odpadu nevznikají přímé nové asociace. Nové vazby jsou:
- **Mapování druh–skupina** (nová vazební entita) → Skupina odpadu — viz PP-2
- **RPO** → Skupina odpadu (N:1) — viz PP-8
- **Nádoba** → Skupina odpadu (N:1) — stávající asociace, beze změny

#### Nová vazební entita: Mapování druh–skupina

Potřeba odlišit mapování Druh odpadu → Skupina odpadu per provozovna vyžaduje novou entitu:

| Atribut | Typ (DS) | Povinný | Popis | Zdroj požadavku |
|---|---|---|---|---|
| Druh odpadu | Reference → Druh odpadu | Ano | Mapovaný druh odpadu | CK: Druhy odpadu |
| Skupina odpadu | Reference → Skupina odpadu | Ano | Cílová skupina pro daný druh | CK: Druhy odpadu |
| Provozovna | Reference → Provozovna | Ano | Provozovna, pro kterou mapování platí | CK: Druhy odpadu ("evidována per Provozovna") |
| + systémové atributy | | | audit, tenant | |

**Business pravidlo:** Kombinace (Druh odpadu, Provozovna) je unikátní — jeden druh odpadu má v rámci provozovny právě jednu skupinu.

### Business pravidla — skupina MIX

Speciální skupina „MIX" (Kombinovaný svoz) má aplikační pravidla:
1. Automaticky plněna na RPO, pokud je na Předmětu smlouvy HEN vyplněn atribut „Odpad kombinovaného zberu"
2. Není přepisována standardním mapováním Druh → Skupina
3. Nelze ji nabídnout jako skupinu pro zadání k druhu odpadu v číselníku Druhy odpadu

> Pravidla jsou řešena aplikační logikou, nevyžadují změnu DS (MIX je běžný záznam v číselníku Skupina odpadu).

### Dopad na DDL (doporučení)

- **`garbage_group`** — beze změn
- **Nová tabulka `garbage_type_garbage_group_mapping`** (nebo obdobná) pro vazební entitu Mapování druh–skupina:
  - `garbage_type_id` (FK → garbage_type)
  - `garbage_group_id` (FK → garbage_group)
  - `organization_unit_id` (FK → organization_unit)
  - UNIQUE constraint na (`garbage_type_id`, `organization_unit_id`)

### Integrace

| Tok | Role | Popis |
|---|---|---|
| PP → RP (sync číselníků) | Zdroj (SoT) | Skupina odpadu se synchronizuje z PP do RP jako read-only kopie |
| HEN → PP (sync RPO) | Cíl | Při synchronizaci RPO se automaticky vyplňuje Skupina odpadu dle mapování |

### Otevřené otázky

- OQ-PP1-01: ~~Provozovna na Skupině odpadu~~ → **Vyřešeno**: Skupina je globální, provozovní kontext přes vazební tabulku
- OQ-PP1-02: ~~Skupina MIX~~ → **Vyřešeno**: Běžný záznam, speciální chování na aplikační úrovni
- OQ-PP1-03: ~~Sdílení PP ↔ RP~~ → **Vyřešeno**: Synchronizace přes integraci, RP má vlastní kopii

### Stav schválení

- [x] Navrženo
- [ ] Schváleno analytikem
- [ ] Promítnuto do DS

> **Schvalovací poznámky:** Odsouhlaseno: Skupina odpadu zůstává globální (bez Provozovny), provozovní kontext přes vazební tabulku. MIX = běžný záznam, speciální chování aplikační. Sdílení PP↔RP přes integraci (RP má vlastní kopii).

---

## 2. Druh odpadu

**Stav:** Existující — rozšíření pro CS
**DS:** Druh odpadu
**DDL:** `garbage_type`

### Business kontext

> Druh odpadu je existující číselník v PP (katalóg odpadov — kódy dle legislativy, např. 200301). Pro CS získává novou roli:
>
> - **Číselník Druhy odpadu** je místem, kde se spravuje vazba **Druh odpadu → Skupina odpadu** (CK: Aplikace Pasport → Druhy odpadu)
> - Tato vazba se liší per provozovna — jeden druh odpadu může v různých provozovnách spadat do různých skupin
> - Mapování je spravováno výhradně v PP; RP čerpá výsledek (Skupinu odpadu na RPO) read-only
> - Při synchronizaci RPO s předměty smluv HEN systém automaticky vyplní Skupinu odpadu na RPO dle tohoto mapování (CK: Přiřazení skupiny odpadu položce objednávky)
>
> Samotná entita Druh odpadu se v DS nemění — nemá nové atributy ani přímé nové asociace. Veškerá nová funkcionalita je na vazební entitě Mapování druh–skupina (navržena v PP-1).

### Návrh rozšíření DS

#### Nové atributy

Žádné. Stávající sada (Kód, Popis, Kategorie, Externí identifikátor + systémové) je dostatečná.

#### Nové asociace

Na entitě Druh odpadu nevznikají přímé nové asociace. Vztah ke Skupině odpadu je realizován přes vazební entitu **Mapování druh–skupina** (viz PP-1):
- Druh odpadu ← Mapování druh–skupina (1:N per provozovna)

### Dopad na DDL (doporučení)

- **`garbage_type`** — beze změn
- Vazební tabulka `garbage_type_garbage_group_mapping` navržena v PP-1

### Business pravidla pro UI číselníku Druhy odpadu

1. V číselníku Druhy odpadu se pro každý druh nastavuje přiřazení ke Skupině odpadu **v kontextu provozovny**
2. Skupinu MIX nelze nabídnout jako cílovou skupinu pro přiřazení (CK: Speciální skupina MIX)
3. Mapování slouží jako vstup pro automatické vyplnění Skupiny odpadu na RPO při synchronizaci z HEN

### Integrace

| Tok | Role | Popis |
|---|---|---|
| HEN → PP (sync RPO) | Kontext | Druh odpadu na RPO (z PZ v HEN) slouží jako klíč pro dohledání Skupiny odpadu přes mapování |
| PP → RP (sync číselníků) | — | Samotný číselník Druh odpadu se do RP nepřenáší; RP pracuje se Skupinou odpadu (přes RPO) |

### Otevřené otázky

Žádné — vše vyřešeno v rámci PP-1.

### Stav schválení

- [x] Navrženo
- [ ] Schváleno analytikem
- [ ] Promítnuto do DS

---

## 3. Zóna

**Stav:** Nová entita pro CS
**DS:** —
**DDL:** —

### Business kontext

> Zóna je nová entita v PP. Geograficky sdružuje Předměty smluv HEN do celků a pomáhá při filtraci pro plánování (CK: Aplikace Pasport → Zóny).
>
> - **Zdroj v Etapě 1:** HEN — import Zón z HEN do DB PP + uložení vazby RPO↔Zóna
> - **Kardinalita:** RPO : Zóna = **1:1** (CK: Vazba RPO na okruhy, rozvrhy a zóny)
> - **Využití v PP:** evidence, filtrování, zobrazení na mapě
> - **Etapa 2:** možnost změny přiřazení Zóny na RPO (CK: Rozšíření panelu přiřazení)
>
> V HEN je Zóna definována přes adresní strukturu (okres → obec → ulice) a automaticky se přiřazuje na PZ dle adresy stanoviska (HEN guide kap. 2.3). V PP se modeluje jako jednoduchá entita — bez replikace adresní struktury z HEN.
>
> V Etapě 1 se Zóna do RP nepřenáší.

### Návrh DS — nová entita

#### Atributy

| Atribut | Typ (DS) | Povinný | Popis | Zdroj požadavku |
|---|---|---|---|---|
| Název | Řetězec (255) | Ano | Název zóny (v HEN např. "KO - Beluša - Pondelok") | HEN guide: Názov subjektu |
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor zóny v HEN | Standardní vzor PP pro importované entity |
| Provozovna | Reference → Provozovna | Ano | Provozovna, pod kterou zóna spadá | CK: Vazba na provozovnu = Ano; HEN guide: Útvar |
| Poznámka | Řetězec (255) | Ne | Volitelná poznámka | HEN guide: Poznámka |
| Je aktivní | Boolean | Ano | Systémový atribut — podpora deaktivace místo mazání | HEN guide: Stav (aktívny) |
| + systémové atributy | | | audit (vytvořil, datum vytvoření, zdroj vytvoření, ...), tenant | Standardní vzor PP |

#### Asociace

| Asociace | Kardinalita | Popis | Zdroj |
|---|---|---|---|
| Provozovna → Provozovna | N:1 | Zóna patří pod jednu provozovnu | CK: Vazba na provozovnu |

> Asociace RPO → Zóna (1:1) je na straně RPO — viz PP-8.

### Dopad na DDL (doporučení)

- **Nová tabulka `zone`** (nebo `area_zone` apod.):
  - `id` (PK, int IDENTITY)
  - `name` (nvarchar(255), NOT NULL)
  - `external_id` (nvarchar(255), NULL)
  - `organization_unit_id` (FK → organization_unit, NOT NULL)
  - `note` (nvarchar(255), NULL)
  - `is_active` (bit, NOT NULL)
  - `tenant_id`, audit sloupce (standardní vzor PP)

### Integrace

| Tok | Role | Popis |
|---|---|---|
| HEN → PP (FR-01) | Cíl | Import zón z HEN do PP — hlavička zóny (název, provozovna) |
| PP → RP | — | V Etapě 1 se Zóna do RP nepřenáší |

### Otevřené otázky

- OQ-PP3-01: ~~Položky zóny (adresní struktura)~~ → **Vyřešeno**: Jednoduchá entita, bez replikace adresní struktury z HEN
- OQ-PP3-02: ~~Región~~ → **Vyřešeno**: HEN-specifický koncept, v PP nemodelujeme
- OQ-PP3-03: ~~Přenos do RP~~ → **Vyřešeno**: V E1 se Zóna do RP nepřenáší

### Stav schválení

- [x] Navrženo
- [ ] Schváleno analytikem
- [ ] Promítnuto do DS
