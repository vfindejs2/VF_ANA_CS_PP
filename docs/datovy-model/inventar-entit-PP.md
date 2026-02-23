# Inventář entit PP pro Cyklické svozy

> **Účel:** Přehled existujících entit v DS a DDL relevantních pro CS. Čistý inventář bez hodnocení.
> **Zdroje:** DS (datovy-88033012.html, 77 entit), DDL (DDL_PP_DB.txt)
> **Vytvořeno:** 2026-02-23

---

## Obsah

1. [Objednávka](#1-objednávka)
2. [Položka objednávky](#2-položka-objednávky)
3. [Revize položky objednávky](#3-revize-položky-objednávky)
4. [Nádoba](#4-nádoba)
5. [Stanoviště](#5-stanoviště)
6. [Typ nádoby](#6-typ-nádoby)
7. [Druh odpadu](#7-druh-odpadu)
8. [Skupina odpadu](#8-skupina-odpadu)
9. [Frekvence vývozu](#9-frekvence-vývozu)
10. [Základní dny svozu](#10-základní-dny-svozu)
11. [Adresa](#11-adresa)
12. [Provozovna](#12-provozovna)
13. [Zákazník](#13-zákazník)
14. [Vzor nádoby](#14-vzor-nádoby)
15. [Kategorie nádoby](#15-kategorie-nádoby)
16. [Barva skupiny odpadu](#16-barva-skupiny-odpadu)
17. [Přiřazení nádoby ke stanovišti](#17-přiřazení-nádoby-ke-stanovišti)
18. [Přiřazení nádoby k položce objednávky](#18-přiřazení-nádoby-k-položce-objednávky)
19. [Přiřazení dočasných parametrů svozu k nádobě](#19-přiřazení-dočasných-parametrů-svozu-k-nádobě)
20. [Přiřazení základních dní svozu k revizi položky objednávky](#20-přiřazení-základních-dní-svozu-k-revizi-položky-objednávky)
21. [Rozšíření nádoby](#21-rozšíření-nádoby)
22. [Množstevní jednotka objemu nádoby](#22-množstevní-jednotka-objemu-nádoby)
23. [Entity neexistující v DS ani DDL (nové pro CS)](#23-entity-neexistující-v-ds-ani-ddl-nové-pro-cs)

---

## 1. Objednávka

> Entita, která charakterizuje konkrétní objednávku na realizaci služeb. Může být označována i jako Smlouva či Rámcová smlouva.

**DS:** Objednávka
**DDL:** `[order]`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Číslo objednávky | Řetězec (30) | Ano | Číslo objednávky |
| Odběratel | Reference → Zákazník | Ano | Zákazník představující konkrétního odběratele |
| Konečný příjemce | Reference → Zákazník | Ne | Zákazník představující konkrétního konečného příjemce |
| Platnost od | Datum | Ne | Začátek platnosti objednávky |
| Platnost do | Datum | Ne | Konec platnosti objednávky |
| Provozovna | Reference → Provozovna | Ano | Provozovna, pod kterou objednávka spadá |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| external_id | nvarchar(255) | NULL | — |
| number | nvarchar(30) | NOT NULL | — |
| customer_id | int | NOT NULL | → customer(id) |
| end_customer_id | int | NULL | → customer(id) |
| valid_from | date | NULL | — |
| valid_to | date | NULL | — |
| organization_unit_id | int | NOT NULL | → organization_unit(id) |
| is_active | bit | NOT NULL | — |
| tenant_id | int | NOT NULL | → tenant(id) |
| created_at | datetimeoffset | NOT NULL | — |
| created_by | int | NOT NULL | → [user](id) |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| updated_at | datetimeoffset | NULL | — |
| updated_by | int | NULL | → [user](id) |
| last_modification_source_id | int | NULL | → code_modification_source(id) |

**Asociace z DS:**
- Odběratel → Zákazník (N:1)
- Konečný příjemce → Zákazník (N:1)
- Provozovna → Provozovna (N:1)

**Reference v tech. projektu:**
- 303ucxx (správa dat – objednávky, UC 303uc03–303uc22)
- 402ucxx (správa objednávek, UC 402uc01–402uc04)
- 400ucxx (správa nádob – zobrazení vazby na objednávku)
- 200uc09, 200uc10 (synchronizační komponenta – objednávky)
- winyxuc08–winyxuc11 (integrace WinyX – synchronizace objednávek)
- 202ucxx (synchronizace dat z Helios MPG SK)
- synchronizace-93257926 (přehled integrace)
- datovy-88033012 (datový slovník)
- prehled-88032905 (přehled high-level konceptu)

---

## 2. Položka objednávky

> Entita, která sdružuje jednotlivé Revize položky objednávky (konkrétní verze této entity).

**DS:** Položka objednávky
**DDL:** `order_item`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Číslo položky objednávky | Řetězec (30) | Ano | Číslo položky objednávky (unikátní v rámci Provozovny a Tenanta) |
| Objednávka | Reference → Objednávka | Ano | Objednávka, pod kterou položka spadá |
| Nejnovější revize položky objednávky | Reference → Revize položky objednávky | Ne | Systémový atribut (automaticky aktualizováno) |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| external_id | nvarchar(255) | NULL | — |
| number | nvarchar(30) | NOT NULL | — |
| order_id | int | NOT NULL | → [order](id) |
| is_active | bit | NOT NULL | — |
| tenant_id | int | NOT NULL | → tenant(id) |
| created_at | datetimeoffset | NOT NULL | — |
| created_by | int | NOT NULL | → [user](id) |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| updated_at | datetimeoffset | NULL | — |
| updated_by | int | NULL | → [user](id) |
| last_modification_source_id | int | NULL | — |
| latest_order_item_revision_id | int | NULL | — |

**Asociace z DS:**
- Objednávka → Objednávka (N:1)
- Nejnovější revize položky objednávky → Revize položky objednávky (1:1)

**Reference v tech. projektu:**
- 303ucxx (správa dat – objednávky, zejména UC 303uc06–303uc12, 303uc15, 303uc18, 303uc21)
- 304ucxx (správa dat – vazby, UC 304uc04–304uc06, 304uc23–304uc25)
- 402ucxx (správa objednávek, UC 402uc02–402uc04)
- 400ucxx (správa nádob – zobrazení vazby na položku objednávky)
- 200uc10, 200uc14, 200uc16, 200uc26, 200uc32 (synchronizační komponenta)
- winyxuc09–winyxuc11 (integrace WinyX – synchronizace položek objednávek)
- 202ucxx (synchronizace dat z Helios MPG SK)
- synchronizace-93257926 (přehled integrace)
- datovy-88033012 (datový slovník)

---

## 3. Revize položky objednávky

> Entita, která charakterizuje objednávku na realizaci konkrétní služby. Jedná se o konkrétní verzi odpovídající Položky objednávky.

**DS:** Revize položky objednávky
**DDL:** `order_item_revision`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Revize | Řetězec (15) | Ano | Pořadí konkrétní revize položky objednávky |
| Položka objednávky | Reference → Položka objednávky | Ano | Položka objednávky, pod kterou spadá revize |
| Platnost od | Datum | Ano | Začátek platnosti revize |
| Platnost do | Datum | Ne | Konec platnosti revize |
| Adresa místa realizace | Reference → Adresa | Ne | Adresa umístění nádoby |
| Poplatník | Řetězec (255) | Ne | Skutečný uživatel nádoby |
| Počet nádob | Číslo (nezáporné, 2 des. místa) | Ano | Počet zasmluvněných nádob uvedeného typu |
| Typ nádoby | Reference → Typ nádoby | Ne | Typ odpovídajících nádob |
| Vlastnictví nádoby | Enumerace → Vlastnictví nádoby | Ne | Informace o vlastnictví odpovídajících nádob |
| Druh odpadu | Reference → Druh odpadu | Ano | Zasmluvněný druh odpadu |
| Frekvence vývozu | Reference → Frekvence vývozu | Ne | Frekvence vývozu odpovídajících nádob |
| Poznámka | Řetězec (255) | Ne | Poznámka k položce objednávky |
| Poznámka 2 | Řetězec (255) | Ne | Další poznámka |
| Poznámka 3 | Řetězec (255) | Ne | Další poznámka |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| external_id | nvarchar(255) | NULL | — |
| revision | nvarchar(15) | NOT NULL | — |
| order_item_id | int | NOT NULL | → order_item(id) |
| valid_from | date | NOT NULL | — |
| valid_to | date | NULL | — |
| realization_address_id | int | NULL | → address(id) |
| payer | nvarchar(255) | NULL | — |
| container_count | float | NOT NULL | — |
| container_type_id | int | NULL | → container_type(id) |
| container_ownership | nvarchar(255) | NULL | — |
| garbage_type_id | int | NOT NULL | → garbage_type(id) |
| waste_disposal_frequency_id | int | NULL | → waste_disposal_frequency(id) |
| waste_disposal_day_id | int | NULL | → waste_disposal_day(id) |
| note | nvarchar(255) | NULL | — |
| note2 | nvarchar(255) | NULL | — |
| note3 | nvarchar(255) | NULL | — |
| is_active | bit | NOT NULL | — |
| tenant_id | int | NOT NULL | → tenant(id) |
| created_at | datetimeoffset | NOT NULL | — |
| created_by | int | NOT NULL | → [user](id) |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| updated_at | datetimeoffset | NULL | — |
| updated_by | int | NULL | → [user](id) |
| last_modification_source_id | int | NULL | — |

**Asociace z DS:**
- Položka objednávky → Položka objednávky (N:1)
- Adresa místa realizace → Adresa (N:1)
- Typ nádoby → Typ nádoby (N:1)
- Druh odpadu → Druh odpadu (N:1)
- Frekvence vývozu → Frekvence vývozu (N:1)

**Reference v tech. projektu:**
- 303ucxx (správa dat – objednávky, zejména UC 303uc09–303uc12, 303uc15, 303uc18, 303uc21–303uc22)
- 304ucxx (správa dat – vazby, UC 304uc04–304uc06, 304uc23–304uc25)
- 402ucxx (správa objednávek, UC 402uc02–402uc04)
- 400ucxx (správa nádob – zobrazení vazby na RPO)
- 200uc10, 200uc14, 200uc26, 200uc32 (synchronizační komponenta)
- winyxuc10 (integrace WinyX – synchronizace revizí)
- 202ucxx (synchronizace dat z Helios MPG SK, UC 202uc09–202uc12)
- sprava-106463495 (správa revizí)
- datovy-88033012 (datový slovník)

---

## 4. Nádoba

> Entita, která charakterizuje konkrétní fyzickou nádobu.

**DS:** Nádoba
**DDL:** `container`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Zdroj externího identifikátoru | Boolean | Ano | Informace, zda byl externí identifikátor nastaven externě |
| Evidenční číslo | Řetězec (20) | Ano | Evidenční číslo nádoby (unikátní v rámci Provozovny a Tenanta) |
| Interní číslo | Řetězec (20) | Ne | Interní číslo nádoby (unikátní v rámci Provozovny a Tenanta) |
| Provozovna | Reference → Provozovna | Ano | Provozovna, pod kterou nádoba spadá |
| Vlastnictví | Enumerace → Vlastnictví nádoby | Ne | Informace o vlastnictví nádoby |
| Stav | Enumerace → Stav nádoby | Ano | Stav, ve kterém se nádoba nachází |
| Rok výroby | Číslo (nezáporné, celé) | Ne | Rok, ve kterém byla nádoba vyrobena |
| Poznámka | Řetězec (255) | Ne | Poznámka k nádobě |
| Veřejná poznámka | Řetězec (255) | Ne | Veřejná poznámka k nádobě (Poznámka pro zákazníka v aplikaci) |
| Skupina odpadu | Reference → Skupina odpadu | Ne | Skupina odpadu, pro kterou je nádoba určena |
| Kategorie nádoby | Reference → Kategorie nádoby | Ne | Kategorie, ve které je nádoba zařazena |
| Vzor nádoby | Reference → Vzor nádoby | Ne | Vzor, který určuje vlastnosti nádoby |
| Výchozí fotografie | Reference → Fotografie nádoby | Ne | Výchozí fotografie nádoby |
| Poslední záznam o pasportizaci nádoby | Reference → Záznam o pasportizaci nádoby | Ne | Poslední záznam o pasportizaci |
| Vyřazena ke dni | Datum | Podmíněná | Datum, ke kterému je nádoba vyřazena |
| Datum vyřazení | Datum a čas | Podmíněná | Datum a čas, ve kterém došlo k vyřazení |
| Vyřadil | Reference → Uživatel | Podmíněná | Uživatel, který provedl vyřazení |
| Zdroj vyřazení | Enumerace → Zdroj změny | Podmíněná | Zdroj, ze kterého proběhlo vyřazení |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| evidence_number | nvarchar(255) | NOT NULL | — |
| organization_unit_id | int | NOT NULL | → organization_unit(id) |
| ownership_status | nvarchar(255) | NULL | → code_container_ownership(name) |
| status | nvarchar(255) | NOT NULL | → code_container_status(name) |
| note | nvarchar(255) | NULL | — |
| category_id | int | NULL | → container_category(id) |
| pattern_id | int | NULL | → container_pattern(id) |
| default_photo_id | int | NULL | — |
| is_active | bit | NOT NULL | — |
| created_at | datetimeoffset | NOT NULL | — |
| updated_at | datetimeoffset | NOT NULL | — |
| created_by | int | NULL | → [user](id) |
| updated_by | int | NULL | → [user](id) |
| tenant_id | int | NOT NULL | → tenant(id) |
| external_id | nvarchar(255) | NULL | — |
| garbage_group_id | int | NULL | → garbage_group(id) |
| evidence_number_search | nvarchar(255) | NULL | — |
| internal_number | nvarchar(20) | NULL | — |
| last_modification_source_id | int | NULL | → code_modification_source(id) |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| last_container_pasport_record_id | int | NULL | → container_pasport_record(id) |
| discarded_at | datetimeoffset | NULL | — |
| discard_to_date | datetimeoffset | NULL | — |
| discarded_by | int | NULL | → [user](id) |
| discard_source_id | int | NULL | → code_modification_source(id) |
| year_production | int | NULL | — |
| external_id_source | bit | NOT NULL | — |
| public_note | nvarchar(255) | NULL | — |

**Asociace z DS:**
- Provozovna → Provozovna (N:1)
- Skupina odpadu → Skupina odpadu (N:1)
- Kategorie nádoby → Kategorie nádoby (N:1)
- Vzor nádoby → Vzor nádoby (N:1)

**Reference v tech. projektu:**
- 301ucxx (správa dat – nádoby, UC 301uc01–301uc13)
- 400ucxx (správa nádob, UC 400uc01–400uc25 – hlavní UI modul)
- 304ucxx (správa dat – vazby nádob, UC 304uc01–304uc25)
- 403ucxx (správa vazeb – UI, UC 403uc01–403uc14)
- 300ucxx (správa dat – původní umístění, UC 300uc04–300uc19)
- 305ucxx (správa dat – schvalování dat)
- 405ucxx (schvalování dat)
- 407ucxx (import záznamů)
- 201ucxx (rozhraní pro FLW Picker)
- 200ucxx (synchronizační komponenta – nádoby)
- 404ucxx (tiskové výstupy)
- winyxuc01, winyxuc05, winyxuc10, winyxuc11 (integrace WinyX)
- 202ucxx (synchronizace dat z Helios MPG SK)
- nadoby-98241941 (technická dokumentace – nádoby)
- synchronizace-93257926 (přehled integrace)
- prehled-88032905 (přehled high-level konceptu)
- datovy-88033012 (datový slovník)

---

## 5. Stanoviště

> Entita, která charakterizuje konkrétní stanoviště (místo, kde jsou umístěny Nádoby).

**DS:** Stanoviště
**DDL:** `site`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Interní identifikátor | Celé číslo | Ano | Interní identifikátor záznamu v databázi |
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Název | Řetězec (255) | Ano | Název stanoviště |
| Provozovna | Reference → Provozovna | Ano | Provozovna, pod kterou stanoviště spadá |
| Adresa | Reference → Adresa | Ano | Adresa stanoviště |
| Souřadnice | Point | Ne | Souřadnice stanoviště |
| Stav geokódování | Enumerace → Stav geokódování | Ano | Stav geokódování adresy stanoviště |
| Je sklad nádob | Boolean | Ano | Informace, zda je stanoviště skladem nádob |
| Poznámka | Řetězec (255) | Ne | Poznámka ke stanovišti |
| Výchozí fotografie | Reference → Fotografie stanoviště | Ne | Výchozí fotografie stanoviště |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| name | nvarchar(255) | NOT NULL | — |
| organization_unit_id | int | NOT NULL | → organization_unit(id) |
| address_id | int | NOT NULL | → address(id) |
| lat | float | NULL | — |
| lng | float | NULL | — |
| note | nvarchar(255) | NULL | — |
| is_active | bit | NOT NULL | — |
| created_at | datetimeoffset | NOT NULL | — |
| created_by | int | NOT NULL | → [user](id) |
| updated_at | datetimeoffset | NULL | — |
| updated_by | int | NULL | → [user](id) |
| tenant_id | int | NOT NULL | → tenant(id) |
| external_id | nvarchar(255) | NULL | — |
| geom | geography | NULL | — |
| geocoding_state | int | NOT NULL | → code_geocoding_state(id) |
| default_photo_id | int | NULL | → photos(id) |
| name_search | nvarchar(255) | NULL | — |
| last_modification_source_id | int | NULL | → code_modification_source(id) |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| is_container_warehouse | bit | NOT NULL | — |

**Asociace z DS:**
- Provozovna → Provozovna (N:1)
- Adresa → Adresa (N:1)
- Výchozí fotografie → Fotografie stanoviště (N:1)

**Reference v tech. projektu:**
- 302ucxx (správa dat – stanoviště, UC 302uc01–302uc11)
- 401ucxx (správa stanovišť – UI, UC 401uc01–401uc08)
- 304ucxx (správa dat – vazby, UC 304uc01–304uc03, 304uc19–304uc22)
- 300ucxx (správa dat – původní umístění, UC 300uc07, 300uc09, 300uc40)
- 403ucxx (správa vazeb – UC 403uc01–403uc05, 403uc15–403uc16)
- 400ucxx (správa nádob – zobrazení stanoviště nádoby)
- 405ucxx (schvalování dat)
- 201ucxx (rozhraní pro FLW Picker)
- 102ucxx (mapové služby)
- 404ucxx (tiskové výstupy)
- 202ucxx (jednorázové importy dat, synchronizace z Helios)
- winyxuc11 (integrace WinyX)
- synchronizace-93257926 (přehled integrace)
- prehled-88032905 (přehled high-level konceptu)
- datovy-88033012 (datový slovník)

---

## 6. Typ nádoby

> Jedná se o hrubou typologii nádob, která je důležitá z pohledu objednávky (nejedná se o přesnou evidenci technických parametrů nádoby).

**DS:** Typ nádoby
**DDL:** `container_type`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Název | Řetězec (50) | Ano | Název typu nádoby |
| Popis | Řetězec (255) | Ne | Popis typu nádoby |
| Provozovna | Reference → Provozovna | Ano | Provozovna, pod kterou typ nádoby spadá |
| Oblast použití | Řetězec (15) | Ano | Oblast, ve které je záznam použit z pohledu synchronizace |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| name | nvarchar(255) | NOT NULL | — |
| description | nvarchar(255) | NULL | — |
| tenant_id | int | NOT NULL | → tenant(id) |
| is_active | bit | NOT NULL | — |
| created_at | datetimeoffset | NULL | — |
| updated_at | datetimeoffset | NULL | — |
| external_id | nvarchar(255) | NULL | — |
| created_by | int | NULL | → [user](id) |
| updated_by | int | NULL | → [user](id) |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| last_modification_source_id | int | NULL | → code_modification_source(id) |

**Asociace z DS:**
- Provozovna → Provozovna (N:1)

**Reference v tech. projektu:**
- 303ucxx (správa dat – objednávky, UC 303uc19–303uc21)
- 402ucxx (správa objednávek – zobrazení typu nádoby)
- 403ucxx (správa vazeb – UC 403uc09–403uc10)
- 400ucxx (správa nádob – UC 400uc06, 400uc10)
- 200uc13, 200uc26 (synchronizační komponenta)
- winyxuc05, winyxuc10 (integrace WinyX – synchronizace typů nádob)
- 202ucxx (synchronizace dat z Helios MPG SK, UC 202uc05, 202uc10)
- 406ucxx (asynchronní export záznamů)
- synchronizace-93257926 (přehled integrace)
- prehled-88032905 (přehled high-level konceptu)
- datovy-88033012 (datový slovník)

---

## 7. Druh odpadu

> Číselník představující katalog odpadů.

**DS:** Druh odpadu
**DDL:** `garbage_type`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Kód | Řetězec (15) | Ano | Kód odpadu |
| Popis | Řetězec (255) | Ne | Popis odpadu |
| Kategorie | Řetězec (4) | Ano | Kategorie odpadu |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| external_id | nvarchar(255) | NULL | — |
| code | nvarchar(15) | NOT NULL | — |
| description | nvarchar(255) | NULL | — |
| category | nvarchar(4) | NOT NULL | — |
| is_active | bit | NOT NULL | — |
| tenant_id | int | NOT NULL | → tenant(id) |
| created_at | datetimeoffset | NOT NULL | — |
| created_by | int | NOT NULL | → [user](id) |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| updated_at | datetimeoffset | NULL | — |
| updated_by | int | NULL | → [user](id) |
| last_modification_source_id | int | NOT NULL | → code_modification_source(id) |

**Asociace z DS:**
- (žádné referenční asociace)

**Reference v tech. projektu:**
- 300ucxx (správa dat, UC 300uc04–300uc06)
- 402ucxx (správa objednávek – UC 402uc02–402uc03)
- 403ucxx (správa vazeb – UC 403uc09–403uc10)
- 400ucxx (správa nádob – UC 400uc10)
- 200uc23, 200uc26 (synchronizační komponenta)
- winyxuc04 (integrace WinyX – synchronizace druhů odpadu)
- 202ucxx (synchronizace dat z Helios MPG SK, UC 202uc04, 202uc10)
- synchronizace-93257926 (přehled integrace)
- prehled-88032905 (přehled high-level konceptu)
- datovy-88033012 (datový slovník)

---

## 8. Skupina odpadu

> Skupiny odpadu používané pro specifikaci použití konkrétní nádoby.

**DS:** Skupina odpadu
**DDL:** `garbage_group`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Zkrácený název | Řetězec (20) | Ne | Zkrácený název skupiny odpadu |
| Název | Řetězec (50) | Ano | Název skupiny odpadu |
| Popis | Řetězec (255) | Ne | Popis skupiny odpadu |
| Barva | Reference → Barva skupiny odpadu | Ne | Barva skupiny odpadu |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| name | nvarchar(255) | NOT NULL | — |
| description | nvarchar(255) | NULL | — |
| is_active | bit | NOT NULL | — |
| created_at | datetimeoffset | NOT NULL | — |
| updated_at | datetimeoffset | NOT NULL | — |
| created_by | int | NULL | → [user](id) |
| updated_by | int | NULL | → [user](id) |
| tenant_id | int | NOT NULL | → tenant(id) |
| external_id | nvarchar(255) | NULL | — |
| short_name | nvarchar(255) | NULL | — |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| last_modification_source_id | int | NULL | → code_modification_source(id) |
| garbage_group_color_id | int | NULL | → garbage_group_color(id) |

**Asociace z DS:**
- Barva → Barva skupiny odpadu (N:1)

**Reference v tech. projektu:**
- 300ucxx (správa dat – původní umístění, UC 300uc29–300uc31)
- 400ucxx (správa nádob – UC 400uc06–400uc08, 400uc11, 400uc14–400uc16, 400uc18, 400uc25)
- 401ucxx (správa stanovišť – UC 401uc01, 401uc03)
- 402ucxx (správa objednávek – UC 402uc02)
- 403ucxx (správa vazeb – UC 403uc01, 403uc07, 403uc09, 403uc12)
- 301uc07 (správa dat – nádoby)
- 404ucxx (tiskové výstupy)
- 406ucxx (asynchronní export záznamů)
- 202ucxx (jednorázové importy dat)
- inicialni-93593223 (iniciální importy)
- nadoby-98241941 (technická dokumentace – nádoby)
- sprava-88033331 (správa)
- datovy-88033012 (datový slovník)

---

## 9. Frekvence vývozu

> Číselník pro uchování frekvencí vývozu -- typických kombinací, ve kterých dochází k vývozu nádoby.

**DS:** Frekvence vývozu
**DDL:** `waste_disposal_frequency`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Název | Řetězec (50) | Ano | Název frekvence vývozu |
| Popis | Řetězec (80) | Ne | Popis frekvence vývozu |
| Skupina | Řetězec (80) | Ne | Skupina frekvence vývozu |
| Provozovna | Reference → Provozovna | Ne | Provozovna, pro kterou je frekvence vývozu definována |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| external_id | nvarchar(255) | NULL | — |
| name | nvarchar(50) | NOT NULL | — |
| description | nvarchar(80) | NULL | — |
| [group] | nvarchar(80) | NULL | — |
| is_active | bit | NOT NULL | — |
| tenant_id | int | NOT NULL | → tenant(id) |
| created_at | datetimeoffset | NOT NULL | — |
| created_by | int | NOT NULL | → [user](id) |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| updated_at | datetimeoffset | NULL | — |
| updated_by | int | NULL | → [user](id) |
| last_modification_source_id | int | NULL | → code_modification_source(id) |
| external_organization_unit_id | nvarchar(255) | NULL | — |

**Asociace z DS:**
- Provozovna → Provozovna (N:1)

**Reference v tech. projektu:**
- 303ucxx (správa dat – objednávky, UC 303uc13–303uc15)
- 402ucxx (správa objednávek – UC 402uc02–402uc03)
- 403ucxx (správa vazeb – UC 403uc09–403uc10, 403uc13–403uc14)
- 400ucxx (správa nádob – UC 400uc06, 400uc10, 400uc14, 400uc16)
- 401ucxx (správa stanovišť – UC 401uc01)
- 200uc11, 200uc26 (synchronizační komponenta)
- winyxuc01, winyxuc06, winyxuc10 (integrace WinyX – synchronizace frekvencí)
- 202ucxx (synchronizace dat z Helios MPG SK, UC 202uc06, 202uc10)
- 404ucxx (tiskové výstupy)
- synchronizace-93257926 (přehled integrace)
- sprava-88033331 (správa)
- prehled-88032905 (přehled high-level konceptu)
- datovy-88033012 (datový slovník)

---

## 10. Základní dny svozu

> Číselník pro uchování základních dní svozu -- typických kombinací, ve kterých dochází k vývozu nádoby (interval svozu).

**DS:** Základní dny svozu
**DDL:** `waste_disposal_day`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Dny svozu | Řetězec (50) | Ano | Dny, ve kterých je prováděn svoz |
| Počet svozů za měsíc | Číslo (kladné, celé) | Ano | Počet svozů, který proběhne za měsíc |
| Provozovna | Reference → Provozovna | Ne | Provozovna, pro kterou jsou základní dny svozu definovány |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| external_id | nvarchar(255) | NULL | — |
| disposal_day | nvarchar(50) | NOT NULL | — |
| number_of_disposals_per_month | int | NOT NULL | — |
| is_active | bit | NOT NULL | — |
| tenant_id | int | NOT NULL | → tenant(id) |
| created_at | datetimeoffset | NOT NULL | — |
| created_by | int | NOT NULL | → [user](id) |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| updated_at | datetimeoffset | NULL | — |
| updated_by | int | NULL | → [user](id) |
| last_modification_source_id | int | NULL | → code_modification_source(id) |
| external_organization_unit_id | nvarchar(255) | NULL | — |

**Asociace z DS:**
- Provozovna → Provozovna (N:1)

**Reference v tech. projektu:**
- 303ucxx (správa dat – objednávky, UC 303uc16–303uc18)
- 304ucxx (správa dat – vazby, UC 304uc23–304uc25)
- 402ucxx (správa objednávek – UC 402uc02–402uc03)
- 403ucxx (správa vazeb – UC 403uc09–403uc10, 403uc13–403uc14)
- 400ucxx (správa nádob – UC 400uc06, 400uc10, 400uc14, 400uc16)
- 401ucxx (správa stanovišť – UC 401uc01)
- 200uc12, 200uc26, 200uc32 (synchronizační komponenta)
- winyxuc01, winyxuc07, winyxuc10 (integrace WinyX – synchronizace dní svozu)
- 202ucxx (synchronizace dat z Helios MPG SK, UC 202uc07, 202uc10, 202uc12)
- 301uc11 (správa dat – nádoby)
- 404ucxx (tiskové výstupy)
- synchronizace-93257926 (přehled integrace)
- sprava-88033331 (správa)
- prehled-88032905 (přehled high-level konceptu)
- datovy-88033012 (datový slovník)

---

## 11. Adresa

> Entita pro uložení adresy.

**DS:** Adresa
**DDL:** `address`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Externí identifikátor v registru adres | Řetězec (10) | Ne | Identifikátor instance v registru adres |
| Ulice | Řetězec (80) | Ne | Název ulice |
| Číslo popisné | Řetězec (15) | Ne | Číslo popisné |
| Číslo orientační | Řetězec (15) | Ne | Číslo orientační |
| Město | Řetězec (80) | Ano | Název města |
| PSČ | Řetězec (15) | Ne | Poštovní směrovací číslo |
| Stát | Řetězec (50) | Ano | Stát |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| street | nvarchar(80) | NULL | — |
| registry_number | nvarchar(15) | NULL | — |
| orientation_number | nvarchar(15) | NULL | — |
| city | nvarchar(50) | NOT NULL | — |
| postal_code | nvarchar(15) | NULL | — |
| country | nvarchar(50) | NOT NULL | — |
| created_at | datetimeoffset | NULL | — |
| updated_at | datetimeoffset | NULL | — |
| is_active | bit | NOT NULL | — |
| external_id | nvarchar(255) | NULL | — |
| tenant_id | int | NOT NULL | → tenant(id) |
| created_by | int | NOT NULL | → [user](id) |
| updated_by | bit | NULL | — |
| last_modification_source_id | int | NULL | → code_modification_source(id) |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| full_address | nvarchar(255) | NULL | — |
| address_register_external_id | nvarchar(255) | NULL | — |

**Asociace z DS:**
- (žádné referenční asociace)

**Reference v tech. projektu:**
- 300ucxx (správa dat, UC 300uc01–300uc03)
- 401ucxx (správa stanovišť – UC 401uc01–401uc05, 401uc07–401uc08)
- 402ucxx (správa objednávek – UC 402uc02–402uc04)
- 400ucxx (správa nádob – UC 400uc06–400uc07, 400uc10–400uc11, 400uc14–400uc16)
- 403ucxx (správa vazeb – UC 403uc01, 403uc09–403uc10)
- 102ucxx (mapové služby – geokódování)
- 200uc14, 200uc25, 200uc26 (synchronizační komponenta)
- winyxuc01, winyxuc02, winyxuc10 (integrace WinyX – synchronizace adres)
- 202ucxx (synchronizace dat z Helios MPG SK, UC 202uc02–202uc03, 202uc10)
- 404ucxx (tiskové výstupy)
- synchronizace-93257926 (přehled integrace)
- prehled-88032905 (přehled high-level konceptu)
- datovy-88033012 (datový slovník)

---

## 12. Provozovna

> Jednotlivé provozovny, které vedou vlastní evidence nádob, stanovišť a objednávek. Svou úlohu hrají také při autorizaci uživatelů.

**DS:** Provozovna
**DDL:** `organization_unit`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Kód | Řetězec (10) | Ne | Kód provozovny |
| Název | Řetězec (50) | Ano | Název provozovny |
| Nadřazená provozovna | Reference → Provozovna | Ne | Nadřazená provozovna |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int | NOT NULL | PK |
| prefix | nvarchar(255) | NULL | — |
| name | nvarchar(150) | NOT NULL | — |
| note | nvarchar(255) | NULL | — |
| superior_organization_unit | int | NULL | — (self-ref, CHECK constraint) |
| [level] | int | NOT NULL | — |
| tenant_id | int | NOT NULL | → tenant(id) |
| is_active | bit | NOT NULL | — |
| created_at | datetimeoffset | NOT NULL | — |
| updated_at | datetimeoffset | NOT NULL | — |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| last_modification_source_id | int | NULL | → code_modification_source(id) |
| external_id | nvarchar(255) | NULL | — |

**Asociace z DS:**
- Nadřazená provozovna → Provozovna (N:1, self-reference)

**Reference v tech. projektu:**
- 200ucxx (synchronizační komponenta – UC 200uc09–200uc10, 200uc16–200uc18, 200uc26, 200uc30–200uc31)
- 203ucxx (synchronizace dat z FLWW2 – UC 203uc01–203uc04, 203uc06)
- 300ucxx (správa dat – UC 300uc38–300uc40)
- 400ucxx (správa nádob – UC 400uc06–400uc08, 400uc10, 400uc12, 400uc14, 400uc16–400uc17, 400uc24)
- 401ucxx (správa stanovišť – UC 401uc01, 401uc03–401uc05, 401uc08)
- 402ucxx (správa objednávek – UC 402uc02–402uc04)
- 403ucxx (správa vazeb – UC 403uc01, 403uc07, 403uc09–403uc10, 403uc12–403uc14, 403uc16)
- 404ucxx (tiskové výstupy)
- 405ucxx (schvalování dat)
- 406ucxx (asynchronní export záznamů)
- 407ucxx (import záznamů)
- 202ucxx (synchronizace dat z Helios MPG SK)
- flwuc02, flwuc03 (integrace Fleetware)
- synchronizace-93257926 (přehled integrace)
- datovy-88033012 (datový slovník)

---

## 13. Zákazník

> Evidence Zákazníků je společná pro všechny Provozovny.

**DS:** Zákazník
**DDL:** `customer`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Název | Řetězec (255) | Ano | Název organizace (může být i jméno a příjmení) |
| IČO | Řetězec (20) | Ne | Identifikační číslo organizace |
| DIČ | Řetězec (20) | Ne | Daňové identifikační číslo |
| IČ DPH | Řetězec (20) | Ne | Daňové identifikační číslo plátce DPH |
| Plátce DPH | Boolean | Ano | Informace, zda je Zákazník plátcem DPH |
| Adresa sídla | Reference → Adresa | Ne | Adresa sídla organizace |
| Poznámka | Řetězec (255) | Ne | Poznámka k Zákazníkovi |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| external_id | nvarchar(255) | NULL | — |
| name | nvarchar(255) | NOT NULL | — |
| legal_id | nvarchar(20) | NULL | — |
| tax_number | nvarchar(20) | NULL | — |
| vat_number | nvarchar(20) | NULL | — |
| is_vat_payer | bit | NOT NULL | — |
| headquarters_address_id | int | NULL | → address(id) |
| note | nvarchar(255) | NULL | — |
| is_active | bit | NOT NULL | — |
| tenant_id | int | NOT NULL | → tenant(id) |
| created_at | datetimeoffset | NOT NULL | — |
| created_by | int | NOT NULL | → [user](id) |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| updated_at | datetimeoffset | NULL | — |
| updated_by | int | NULL | → [user](id) |
| last_modification_source_id | int | NOT NULL | → code_modification_source(id) |

**Asociace z DS:**
- Adresa sídla → Adresa (N:1)

**Reference v tech. projektu:**
- 303ucxx (správa dat – objednávky, UC 303uc01–303uc03)
- 402ucxx (správa objednávek – UC 402uc02–402uc03)
- 403ucxx (správa vazeb – UC 403uc09–403uc10)
- 400ucxx (správa nádob – UC 400uc06–400uc08, 400uc10, 400uc14–400uc20, 400uc23, 400uc25)
- 401ucxx (správa stanovišť – UC 401uc01, 401uc04)
- 307ucxx (správa dat – zákaznické validace)
- 406ucxx (asynchronní export záznamů)
- 200uc09, 200uc14 (synchronizační komponenta)
- winyxuc01, winyxuc03, winyxuc08–winyxuc11 (integrace WinyX – synchronizace zákazníků)
- 202ucxx (synchronizace dat z Helios MPG SK)
- 404ucxx (tiskové výstupy)
- nadoby-98241941, vysypy-98241945 (technická dokumentace)
- synchronizace-93257926 (přehled integrace)
- prehled-88032905 (přehled high-level konceptu)
- datovy-88033012 (datový slovník)

---

## 14. Vzor nádoby

> Vzor sdružuje společné vlastnosti jednotlivých nádob. Vlastnosti tedy nejsou uloženy přímo u konkrétní nádoby, ale prostřednictvím vzoru.

**DS:** Vzor nádoby
**DDL:** `container_pattern`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Zkrácený název | Řetězec (20) | Ne | Zkrácený název vzoru nádoby |
| Název | Řetězec (50) | Ano | Název vzoru nádoby |
| Popis | Řetězec (255) | Ne | Popis vzoru nádoby |
| Barva nádoby | Reference → Barva nádoby | Ano | Barva nádoby |
| Materiál nádoby | Reference → Materiál nádoby | Ano | Materiál nádoby |
| Objem nádoby | Číslo (nezáporné, 2 des. místa) | Ano | Objem nádoby |
| Množstevní jednotka objemu nádoby | Reference → Množstevní jednotka objemu nádoby | Ano | Množstevní jednotka objemu |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| name | nvarchar(255) | NOT NULL | — |
| description | nvarchar(255) | NULL | — |
| container_color_id | int | NOT NULL | — |
| container_material_id | int | NOT NULL | — |
| container_quantity_unit_id | int | NOT NULL | — |
| volume | float | NOT NULL | — |
| tenant_id | int | NOT NULL | → tenant(id) |
| is_active | bit | NOT NULL | — |
| created_at | datetimeoffset | NULL | — |
| updated_at | datetimeoffset | NULL | — |
| external_id | nvarchar(255) | NULL | — |
| created_by | int | NULL | → [user](id) |
| updated_by | int | NULL | → [user](id) |
| short_name | nvarchar(20) | NULL | — |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| last_modification_source_id | int | NULL | → code_modification_source(id) |

**Asociace z DS:**
- Barva nádoby → Barva nádoby (N:1)
- Materiál nádoby → Materiál nádoby (N:1)
- Množstevní jednotka objemu nádoby → Množstevní jednotka objemu nádoby (N:1)

**Reference v tech. projektu:**
- 300ucxx (správa dat – původní umístění, UC 300uc04, 300uc06–300uc07, 300uc13, 300uc16, 300uc19)
- 400ucxx (správa nádob – UC 400uc02–400uc03, 400uc06–400uc08, 400uc14–400uc16)
- 401ucxx (správa stanovišť – UC 401uc01, 401uc04)
- 403ucxx (správa vazeb – UC 403uc01, 403uc07, 403uc09, 403uc12)
- 200uc02 (synchronizační komponenta)
- 202ucxx (jednorázové importy dat)
- 404ucxx (tiskové výstupy)
- inicialni-93593223 (iniciální importy)
- synchronizacia-98241937 (technická dokumentace – synchronizace)
- prehled-88032905 (přehled high-level konceptu)
- datovy-88033012 (datový slovník)

---

## 15. Kategorie nádoby

> Stromová struktura umožňující kategorizaci jednotlivých nádob.

**DS:** Kategorie nádoby
**DDL:** `container_category`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Zkrácený název | Řetězec (20) | Ne | Zkrácený název kategorie nádoby |
| Název | Řetězec (50) | Ano | Název kategorie nádoby |
| Popis | Řetězec (255) | Ne | Popis kategorie nádoby |
| Nadřazená kategorie nádoby | Reference → Kategorie nádoby | Ne | Nadřazená kategorie (pokud není k dispozici, jedná se o kořenový element) |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| name | nvarchar(255) | NOT NULL | — |
| description | nvarchar(255) | NULL | — |
| parent_id | int | NULL | → container_category(id) (self-ref) |
| tenant_id | int | NOT NULL | → tenant(id) |
| is_active | bit | NOT NULL | — |
| created_at | datetimeoffset | NULL | — |
| updated_at | datetimeoffset | NULL | — |
| external_id | nvarchar(255) | NULL | — |
| created_by | int | NULL | → [user](id) |
| updated_by | int | NULL | → [user](id) |
| short_name | nvarchar(20) | NULL | — |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| last_modification_source_id | int | NULL | → code_modification_source(id) |

**Asociace z DS:**
- Nadřazená kategorie nádoby → Kategorie nádoby (N:1, self-reference)

**Reference v tech. projektu:**
- 300ucxx (správa dat – původní umístění, UC 300uc08–300uc10)
- 400ucxx (správa nádob – UC 400uc04–400uc08)
- 403ucxx (správa vazeb – UC 403uc01, 403uc07, 403uc09, 403uc12)
- 202ucxx (jednorázové importy dat, UC 202uc01–202uc02)
- inicialni-93593223 (iniciální importy)
- synchronizacia-98241937 (technická dokumentace – synchronizace)
- sprava-106463495 (správa revizí)
- datovy-88033012 (datový slovník)

---

## 16. Barva skupiny odpadu

> Dostupné barvy pro Skupiny odpadu.

**DS:** Barva skupiny odpadu
**DDL:** `garbage_group_color`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Název | Řetězec (50) | Ano | Název barvy skupiny odpadu |
| Kód barvy | Řetězec (7) | Ano | Kód konkrétní barvy skupiny odpadu (formát #RRGGBB) |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| external_id | nvarchar(255) | NULL | — |
| name | nvarchar(50) | NOT NULL | — |
| color_code | nvarchar(7) | NOT NULL | — |
| tenant_id | int | NOT NULL | → tenant(id) |
| created_at | datetimeoffset | NOT NULL | — |
| created_by | int | NOT NULL | → [user](id) |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| updated_at | datetimeoffset | NULL | — |
| updated_by | int | NULL | → [user](id) |
| last_modification_source_id | int | NULL | → code_modification_source(id) |
| is_active | bit | NOT NULL | — |

**Asociace z DS:**
- (žádné referenční asociace)

**Reference v tech. projektu:**
- 101uc13 (společné prvky UI -- barevné rozlišení skupin odpadu)
- datovy-88033012 (datový slovník)

---

## 17. Přiřazení nádoby ke stanovišti

> Vazební entita sloužící k přiřazení konkrétní Nádoby k odpovídajícímu Stanovišti.

**DS:** Přiřazení nádoby ke stanovišti
**DDL:** `site_container_assignment`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Nádoba | Reference → Nádoba | Ano | Konkrétní nádoba v rámci přiřazení |
| Stanoviště | Reference → Stanoviště | Ano | Konkrétní stanoviště v rámci přiřazení |
| Datum přistavení | Datum a čas | Ano | Začátek období přistavení |
| Datum stažení | Datum a čas | Ne | Konec období přistavení |
| Poznámka | Řetězec (255) | Ne | Poznámka k přiřazení |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| container_id | int | NOT NULL | → container(id) |
| site_id | int | NOT NULL | → site(id) |
| valid_from | datetimeoffset | NOT NULL | — |
| valid_to | datetimeoffset | NULL | — |
| note | nvarchar(255) | NULL | — |
| is_active | bit | NOT NULL | — |
| created_at | datetimeoffset | NOT NULL | — |
| created_by | int | NOT NULL | → [user](id) |
| updated_at | datetimeoffset | NULL | — |
| updated_by | int | NULL | → [user](id) |
| tenant_id | int | NOT NULL | → tenant(id) |
| external_id | nvarchar(255) | NULL | — |
| last_modification_source_id | int | NULL | → code_modification_source(id) |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |

**Asociace z DS:**
- Nádoba → Nádoba (N:1)
- Stanoviště → Stanoviště (N:1)

**Reference v tech. projektu:**
- 304ucxx (správa dat – vazby, UC 304uc01–304uc03, 304uc22)
- 403ucxx (správa vazeb – UI, UC 403uc01–403uc05)
- 401ucxx (správa stanovišť – UC 401uc01–401uc05, 401uc07)
- 400ucxx (správa nádob – UC 400uc06–400uc07, 400uc10–400uc11, 400uc14–400uc16)
- 301ucxx (správa dat – nádoby, UC 301uc02–301uc03)
- 302ucxx (správa dat – stanoviště, UC 302uc03, 302uc07–302uc08)
- 202ucxx (jednorázové importy dat, UC 202uc02)
- 404ucxx (tiskové výstupy)
- synchronizacia-98241937 (technická dokumentace – synchronizace)
- sprava-106463495 (správa revizí)
- datovy-88033012 (datový slovník)

---

## 18. Přiřazení nádoby k položce objednávky

> Vazební entita sloužící k přiřazení konkrétní Nádoby k odpovídající Položce objednávky.

**DS:** Přiřazení nádoby k položce objednávky
**DDL:** `container_order_item_assignment`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Nádoba | Reference → Nádoba | Ano | Konkrétní nádoba v rámci přiřazení |
| Položka objednávky | Reference → Položka objednávky | Ano | Konkrétní položka objednávky v rámci přiřazení |
| Platnost od | Datum a čas | Ano | Začátek období platnosti přiřazení |
| Platnost do | Datum a čas | Ne | Konec období platnosti přiřazení |
| Poznámka | Řetězec (255) | Ne | Poznámka k přiřazení |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| external_id | nvarchar(255) | NULL | — |
| container_id | int | NOT NULL | → container(id) |
| order_item_id | int | NOT NULL | → order_item(id) |
| valid_from | datetimeoffset | NOT NULL | — |
| valid_to | datetimeoffset | NULL | — |
| note | nvarchar(255) | NULL | — |
| is_active | bit | NOT NULL | — |
| tenant_id | int | NOT NULL | → tenant(id) |
| created_at | datetimeoffset | NOT NULL | — |
| created_by | int | NOT NULL | → [user](id) |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| updated_at | datetimeoffset | NULL | — |
| updated_by | int | NULL | → [user](id) |
| last_modification_source_id | int | NULL | — |

**Asociace z DS:**
- Nádoba → Nádoba (N:1)
- Položka objednávky → Položka objednávky (N:1)

**Reference v tech. projektu:**
- 304ucxx (správa dat – vazby, UC 304uc04–304uc06)
- 403ucxx (správa vazeb – UI, UC 403uc06, 403uc09–403uc10)
- 303ucxx (správa dat – objednávky, UC 303uc09, 303uc22)
- 400ucxx (správa nádob – UC 400uc06, 400uc10, 400uc14, 400uc16, 400uc18–400uc19)
- 401ucxx (správa stanovišť – UC 401uc01, 401uc04)
- 402ucxx (správa objednávek – UC 402uc02, 402uc04)
- 301ucxx (správa dat – nádoby, UC 301uc02–301uc03)
- 200uc16 (synchronizační komponenta)
- 202ucxx (synchronizace dat z Helios MPG SK, UC 202uc11)
- winyxuc11 (integrace WinyX)
- 404ucxx (tiskové výstupy)
- synchronizace-93257926 (přehled integrace)
- synchronizacia-98241937 (technická dokumentace – synchronizace)
- datovy-88033012 (datový slovník)

---

## 19. Přiřazení dočasných parametrů svozu k nádobě

> Vazební entita slouží k dočasnému přiřazení Frekvence vývozu a Základních dní svozu k odpovídající Nádobě.

**DS:** Přiřazení dočasných parametrů svozu k nádobě
**DDL:** `temporary_disposal_parameters_container_assignment`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Nádoba | Reference → Nádoba | Ano | Konkrétní nádoba v rámci přiřazení |
| Frekvence vývozu | Reference → Frekvence vývozu | Ano | Frekvence vývozu odpovídající nádoby |
| Základní dny svozu | Reference → Základní dny svozu | Ano | Základní dny svozu odpovídající nádoby |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| external_id | nvarchar(255) | NULL | — |
| container_id | int | NOT NULL | — |
| waste_disposal_day_id | int | NOT NULL | — |
| waste_disposal_frequency_id | int | NOT NULL | — |
| is_active | bit | NOT NULL | — |
| tenant_id | int | NOT NULL | → tenant(id) |
| created_at | datetimeoffset | NOT NULL | — |
| created_by | int | NOT NULL | → [user](id) |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| updated_at | datetimeoffset | NULL | — |
| updated_by | int | NULL | → [user](id) |
| last_modification_source_id | int | NULL | → code_modification_source(id) |

**Asociace z DS:**
- Nádoba → Nádoba (N:1)
- Frekvence vývozu → Frekvence vývozu (N:1)
- Základní dny svozu → Základní dny svozu (N:1)

**Reference v tech. projektu:**
- 304ucxx (správa dat – vazby, UC 304uc16–304uc18, 304uc20)
- 403ucxx (správa vazeb – UI, UC 403uc13–403uc14)
- 301ucxx (správa dat – nádoby, UC 301uc02–301uc03)
- 400ucxx (správa nádob – UC 400uc06, 400uc10)
- sprava-88033331 (správa)
- datovy-88033012 (datový slovník)

---

## 20. Přiřazení základních dní svozu k revizi položky objednávky

> Vazební entita slouží k přiřazení Základních dní svozu k odpovídající Revizi položky objednávky.

**DS:** Přiřazení základních dní svozu k revizi položky objednávky
**DDL:** `waste_disposal_day_order_item_revision_assignment`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (255) | Ne | Identifikátor instance v externím systému |
| Revize položky objednávky | Reference → Revize položky objednávky | Ano | Konkrétní revize položky objednávky v rámci přiřazení |
| Základní dny svozu | Reference → Základní dny svozu | Ano | Konkrétní základní dny svozu v rámci přiřazení |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| external_id | nvarchar(255) | NULL | — |
| order_item_revision_id | int | NOT NULL | — |
| waste_disposal_day_id | int | NOT NULL | — |
| is_active | bit | NOT NULL | — |
| tenant_id | int | NOT NULL | → tenant(id) |
| created_at | datetimeoffset | NOT NULL | — |
| created_by | int | NOT NULL | → [user](id) |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| updated_at | datetimeoffset | NULL | — |
| updated_by | int | NULL | → [user](id) |
| last_modification_source_id | int | NULL | — |

**Asociace z DS:**
- Revize položky objednávky → Revize položky objednávky (N:1)
- Základní dny svozu → Základní dny svozu (N:1)

**Reference v tech. projektu:**
- 304ucxx (správa dat – vazby, UC 304uc23–304uc25)
- 402ucxx (správa objednávek – UC 402uc02–402uc03)
- 403ucxx (správa vazeb – UC 403uc09)
- 400ucxx (správa nádob – UC 400uc06, 400uc10, 400uc14, 400uc16)
- 401ucxx (správa stanovišť – UC 401uc01)
- 200uc32 (synchronizační komponenta)
- winyxuc10 (integrace WinyX – synchronizace přiřazení)
- 202ucxx (synchronizace dat z Helios MPG SK, UC 202uc12)
- 404ucxx (tiskové výstupy)
- datovy-88033012 (datový slovník)

---

## 21. Rozšíření nádoby

> Entita obsahující informace související s Nádobou, které jsou typicky použity pro optimalizaci výkonu při práci s ní (např. poslední výsyp této nádoby, aktuální stanoviště, přiřazená položka objednávky apod.).

*Poznámka: Tato entita je materializovaný pohled / pomocná tabulka, která agreguje aktuální stav vazeb nádoby z přiřazovacích tabulek.*

**DS:** Rozšíření nádoby
**DDL:** `container_extended`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Nádoba | Reference → Nádoba | Ano | Nádoba, ke které jsou pomocné informace vázány |
| Poslední událost výsypu | Reference → Událost výsypu nádoby | Ne | Poslední událost výsypu evidovaná u dané nádoby |
| Datum a čas poslední události výsypu | Datum a čas | Ne | Datum a čas poslední události výsypu |
| Stanoviště | Reference → Stanoviště | Ne | Id stanoviště přiřazené k nádobě |
| Nádoba - stanoviště | Reference → Přiřazení nádoby ke stanovišti | Ne | Aktuální vazba stanoviště přiřazené k nádobě |
| Uživatel nádoby | Reference → Přiřazení poplatníka k nádobě | Ne | Id přiřazení uživatele k nádobě |
| Typ uživatele nádoby | Reference → Typ poplatníka | Ne | Id typu přiřazeného uživatele k nádobě |
| Název uživatele nádoby | nvarchar (255) | Ne | Name přiřazeného uživatele k nádobě |
| Identifikátor nádoby | Reference → Přiřazení identifikátoru k nádobě | Ne | Id přiřazeného identifikátoru k nádobě |
| Kód identifikátoru | nvarchar (255) | Ne | Kód přiřazeného identifikátoru |
| Stav přiřazení identifikátoru | bit | Ano | K nádobě je/není přiřazen identifikátor |
| Číslo položky objednávky | nvarchar (255) | Ne | Externí identifikátor položky objednávky |
| Odběratel | Reference → Objednávka | Ne | Id odběratele na objednávce |
| Konečný příjemce | Reference → Objednávka | Ne | Id konečného příjemce objednávky |
| Revize položky objednávky | Reference → Revize položky objednávky | Ne | Id revize položky objednávky |
| Adresa revize pol. obj. | Reference → Revize položky objednávky | Ne | Id adresy na revizi položky objednávky |
| Frekvence revize pol. obj. | Reference → Revize položky objednávky | Ne | Id frekvence svozu na revizi |
| Den svozu revize | Reference → Revize položky objednávky | Ne | Id základního dne svozu |
| Aktivní nádoba | bit | Ne | Informace, zda je aktuální den nádoba aktivní |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| container_id | int | NOT NULL | → container(id) |
| object_empty_container_event_id | int | NULL | → object_empty_container_event(id) |
| object_empty_container_event_timestamp | datetimeoffset | NULL | — |
| is_active | bit | NOT NULL | — |
| tenant_id | int | NOT NULL | → tenant(id) |
| created_at | datetimeoffset | NOT NULL | — |
| created_by | int | NOT NULL | → [user](id) |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| updated_at | datetimeoffset | NULL | — |
| updated_by | int | NULL | → [user](id) |
| last_modification_source_id | int | NULL | → code_modification_source(id) |
| object_empty_container_event_weight | int | NULL | — |

**Asociace z DS:**
- Nádoba → Nádoba (1:1)
- Poslední událost výsypu → Událost výsypu nádoby (N:1)

**Reference v tech. projektu:**
- 304ucxx (správa dat – vazby, UC 304uc01–304uc12, 304uc23–304uc25 – aktualizace rozšíření při změnách vazeb)
- 301ucxx (správa dat – nádoby, UC 301uc01, 301uc03, 301uc10–301uc11, 301uc13)
- 400ucxx (správa nádob – UC 400uc06–400uc07, 400uc21–400uc22)
- sprava-88033331 (správa)
- datovy-88033012 (datový slovník)

---

## 22. Množstevní jednotka objemu nádoby

> Množstevní jednotky používané pro specifikaci objemu konkrétní nádoby. Jedná se o systémový číselník, kdy jsou data zadávána na úrovni databáze.

**DS:** Množstevní jednotka objemu nádoby
**DDL:** `container_quantity_unit`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Název | Řetězec (50) | Ano | Název množstevní jednotky objemu nádoby |
| Značka | Řetězec (15) | Ano | Značka množstevní jednotky objemu nádoby |
| Je aktivní | Boolean | Ano | Systémový atribut |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut |
| Vytvořil | Reference → Uživatel | Ano | Systémový atribut |
| Zdroj vytvoření | Enumerace → Zdroj změny | Ano | Systémový atribut |
| Datum poslední změny | Datum a čas | Ne | Systémový atribut |
| Provedl poslední změnu | Reference → Uživatel | Ne | Systémový atribut |
| Zdroj poslední změny | Enumerace → Zdroj změny | Ne | Systémový atribut |
| Tenant | Reference → Tenant | Ano | Systémový atribut |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | PK |
| name | nvarchar(255) | NOT NULL | — |
| unit | nvarchar(255) | NOT NULL | — |
| tenant_id | int | NOT NULL | → tenant(id) |
| is_active | bit | NOT NULL | — |
| created_at | datetimeoffset | NULL | — |
| updated_at | datetimeoffset | NULL | — |
| created_by | int | NULL | → [user](id) |
| updated_by | int | NULL | → [user](id) |
| external_id | nvarchar(255) | NULL | — |
| creation_source_id | int | NOT NULL | → code_modification_source(id) |
| last_modification_source_id | int | NULL | → code_modification_source(id) |

**Asociace z DS:**
- (žádné referenční asociace)

**Reference v tech. projektu:**
- 300ucxx (správa dat – původní umístění, UC 300uc17–300uc19)
- 400ucxx (správa nádob – UC 400uc02–400uc03, 400uc14)
- 404ucxx (tiskové výstupy)
- sprava-88033331 (správa)
- datovy-88033012 (datový slovník)

---

## 23. Entity neexistující v DS ani DDL (nové pro CS)

Následující entity jsou zmíněny v Cílovém konceptu jako nové pro Cyklické svozy, ale v aktuálním DS ani DDL aplikace PasPort **neexistují**:

| Entita | Popis z Cílového konceptu |
|---|---|
| **Okruh** (circuit) | Nová entita pro CS -- definice okruhů svozu |
| **Rozvrh** (schedule) | Nová entita pro CS -- rozvrhy svozů |
| **Kalendář** (calendar) | Nová entita pro CS -- vazba na rozvrh |
| **Zóna** (zone) | Nová entita pro CS -- geografické zónování |

Tyto entity budou vytvářeny v rámci projektu CS (pravděpodobně v aplikaci RoadPlan, nikoliv v PasPortu).
