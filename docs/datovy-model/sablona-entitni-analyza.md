# Šablona: Entitní analýza pro návrh datového modelu CS

> **Účel:** Referenční šablona pro zpracování každé entity v `docs/datovy-model/datovy-model-PP.md` a `docs/datovy-model/datovy-model-RP.md`.
> **Vytvořeno:** 2026-02-23

---

## Zásady

1. **Nové entity pro CS** (Okruh, Rozvrh, ...) — kompletní DS návrh (všechny atributy, asociace, business pravidla)
2. **Existující entity s rozšířením** (RPO, Nádoba, ...) — jen delta (nové/změněné atributy), stávající se nekopírují
3. **Existující entity beze změny** — do dokumentu se nezařazují
4. **Zdroj požadavku** — vždy konkrétní reference (CK kapitola, UC, business pravidlo)
5. **Sekce „Integrace"** — propojuje datový model s integračními toky (INT-01..06)
6. **DS je primární výstup** — navrhujeme business model pro analytika aplikace
7. **DDL doporučení jsou sekundární** — developer má prostor pro implementační odchylky

---

## Šablona

```markdown
## N. Název entity

**Stav:** Existující / Existující — rozšíření pro CS / Nová entita pro CS
**DS:** Název v DS (nebo "—" pokud nová)
**DDL:** `tabulka` (nebo "—" pokud nová)

### Business kontext
> Co CK říká o roli entity v CS. Proč je potřeba / proč se mění.
> Reference na kapitoly CK.

### Návrh rozšíření DS

#### Nové atributy

| Atribut | Typ (DS) | Povinný | Popis | Zdroj požadavku |
|---|---|---|---|---|
| Skupina odpadu | Reference → Skupina odpadu | Ne | Skupina odpadu přiřazená k RPO | CK kap. X.Y |

#### Změněné atributy
*(pouze pokud se mění typ, povinnost nebo popis stávajícího atributu)*

| Atribut | Původní stav | Nový stav | Důvod změny |
|---|---|---|---|

#### Nové asociace

| Asociace | Kardinalita | Popis | Zdroj |
|---|---|---|---|
| → Okruh | M:N | RPO může být přiřazena k více okruhům | CK kap. X.Y |

### Dopad na DDL (doporučení)
> Stručné doporučení pro vývojáře — nové tabulky, sloupce, FK.
> Jen pokud je to z business pohledu relevantní (např. vazební tabulka pro M:N).

### Integrace
> Které integrační toky entitu využívají (INT-01..06), v jaké roli (zdroj/cíl).

### Otevřené otázky
- OQ-XX: ...

### Stav schválení
- [ ] Navrženo
- [ ] Schváleno analytikem
- [ ] Promítnuto do DS
```
