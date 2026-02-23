# Extrakce entit PP a mapování DS-PP na DDL-PP

> **Datum:** 2026-02-23
> **Zdroj DS-PP:** `data/input/dokumentace_PP/2026-02-19_technicky-projekt-PP/technicky-projekt/datove-modely/datovy-88033012.html`
> **Zdroj DDL-PP:** `data/input/dokumentace_PP/DDL_PP_DB.txt`
> **Počet analyzovaných entit:** 13

---

## Obsah

1. [Revize položky objednávky (RPO)](#1-revize-položky-objednávky-rpo) — Kritická
2. [Položka objednávky](#2-položka-objednávky) — Vysoká
3. [Objednávka](#3-objednávka) — Střední
4. [Stanoviště](#4-stanoviště) — Vysoká
5. [Nádoba](#5-nádoba) — Vysoká
6. [Skupina odpadu](#6-skupina-odpadu) — Kritická
7. [Druh odpadu](#7-druh-odpadu) — Vysoká
8. [Typ nádoby](#8-typ-nádoby) — Vysoká
9. [Frekvence vývozu](#9-frekvence-vývozu) — Střední
10. [Adresa](#10-adresa) — Vysoká
11. [Přiřazení nádoby ke stanovišti](#11-přiřazení-nádoby-ke-stanovišti) — Střední
12. [Přiřazení nádoby k položce objednávky](#12-přiřazení-nádoby-k-položce-objednávky) — Střední
13. [Přiřazení dní svozu k RPO](#13-přiřazení-dní-svozu-k-rpo) — Vysoká

---

## 1. Revize položky objednávky (RPO)

**Priorita: Kritická** — Centrální entita s novými vazbami na okruh, rozvrh, zónu, skupinu odpadu.

### Atributový profil z datového slovníku (DS-PP)

| Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis | Persist. ref. | Poznámka |
|---------|-----------|--------|----------|-----------|-------|---------------|----------|
| Externí identifikátor | Řetězec | 255 | – | Ne | Identifikátor instance v externím systému. | – | Jedinečný v rámci Položky objednávky. |
| Revize | Řetězec | 15 | – | Ano | Pořadí konkrétní revize položky objednávky. | – | Jedinečná v rámci Položky objednávky. Nemusí začínat od 1. |
| Položka objednávky | Reference | – | Položka objednávky | Ano | Položka objednávky, pod kterou spadá RPO. | – | – |
| Platnost od | Datum | – | – | Ano | Začátek platnosti RPO. | – | – |
| Platnost do | Datum | – | – | Ne | Konec platnosti RPO. | – | – |
| Adresa místa realizace | Reference | – | Adresa | Ne | Adresa umístění nádoby. | Ne | – |
| Poplatník | Řetězec | 255 | – | Ne | Skutečný uživatel nádoby. | – | – |
| Počet nádob | Číslo (nezáporné, 2 des. místa) | – | – | Ano | Počet zasmluvněných nádob uvedeného typu. | – | – |
| Typ nádoby | Reference | – | Typ nádoby | Ne | Typ odpovídajících nádob. | – | – |
| Vlastnictví nádoby | Enumerace | – | Vlastnictví nádoby | Ne | Informace o vlastnictví odpovídajících nádob. | – | – |
| Druh odpadu | Reference | – | Druh odpadu | Ano | Zasmluvněný druh odpadu. | Ne | – |
| Frekvence vývozu | Reference | – | Frekvence vývozu | Ne | Frekvence vývozu odpovídajících nádob. | Ne | – |
| Poznámka | Řetězec | 255 | – | Ne | Poznámka k položce objednávky. | – | – |
| Poznámka 2 | Řetězec | 255 | – | Ne | Další poznámka k RPO. | – | – |
| Poznámka 3 | Řetězec | 255 | – | Ne | Další poznámka k RPO. | – | – |
| Je aktivní | Boolean | – | – | Ano | Zda je instance RPO aktivní. | – | Systémový atribut. |
| Datum vytvoření | Datum a čas | – | – | Ano | Datum a čas vytvoření RPO. | – | Systémový atribut. |
| Vytvořil | Reference | – | Uživatel | Ano | Uživatel, který vytvořil záznam. | Ne | Systémový atribut. |
| Zdroj vytvoření | Enumerace | – | Zdroj změny | Ano | Zdroj vytvoření záznamu. | – | Systémový atribut. |
| Datum poslední změny | Datum a čas | – | – | Ne | Datum a čas poslední změny RPO. | – | Systémový atribut. |
| Provedl poslední změnu | Reference | – | Uživatel | Ne | Uživatel, který provedl poslední změnu. | Ne | Systémový atribut. |
| Zdroj poslední změny | Enumerace | – | Zdroj změny | Ne | Zdroj poslední změny. | – | Systémový atribut. |
| Tenant | Reference | – | Tenant | Ano | Rozlišení firmy/organizace. | Ne | Systémový atribut. |

### Odpovídající tabulka v DDL (DDL-PP)

**Table name:** `PasPort_MPGSK.dbo.order_item_revision`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | – |
| external_id | nvarchar(255) | NULL | – | – |
| revision | nvarchar(15) | NOT NULL | – | – |
| order_item_id | int | NOT NULL | order_item(id) | – |
| valid_from | date | NOT NULL | – | – |
| valid_to | date | NULL | – | – |
| realization_address_id | int | NULL | address(id) | – |
| payer | nvarchar(255) | NULL | – | – |
| container_count | float | NOT NULL | – | – |
| container_type_id | int | NULL | container_type(id) | – |
| container_ownership | nvarchar(255) | NULL | code_container_ownership(name) | – |
| garbage_type_id | int | NOT NULL | garbage_type(id) | – |
| waste_disposal_frequency_id | int | NULL | waste_disposal_frequency(id) | – |
| waste_disposal_day_id | int | NULL | waste_disposal_day(id) | – |
| note | nvarchar(255) | NULL | – | – |
| note2 | nvarchar(255) | NULL | – | – |
| note3 | nvarchar(255) | NULL | – | – |
| is_active | bit | NOT NULL | – | 1 |
| tenant_id | int | NOT NULL | tenant(id) | – |
| created_at | datetimeoffset | NOT NULL | – | getutcdate() |
| created_by | int | NOT NULL | user(id) | – |
| creation_source_id | int | NOT NULL | code_modification_source(id) | – |
| updated_at | datetimeoffset | NULL | – | – |
| updated_by | int | NULL | user(id) | – |
| last_modification_source_id | int | NULL | code_modification_source(id) | – |

### Mapování DS-PP <> DDL-PP

| Atribut (DS) | Persist. ref. | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|---------------|----------|-----------|-------|----------|
| *(implicitní PK)* | – | id | – | int IDENTITY | – | DS neuvádí interní identifikátor |
| Externí identifikátor | – | external_id | Řetězec(255) | nvarchar(255) | ✓ | – |
| Revize | – | revision | Řetězec(15) | nvarchar(15) | ✓ | – |
| Položka objednávky | – | order_item_id | Reference | int FK | ✓ | – |
| Platnost od | – | valid_from | Datum | date | ✓ | – |
| Platnost do | – | valid_to | Datum | date | ✓ | – |
| Adresa místa realizace | Ne | realization_address_id | Reference→Adresa | int FK | ✓ | – |
| Poplatník | – | payer | Řetězec(255) | nvarchar(255) | ✓ | – |
| Počet nádob | – | container_count | Číslo(2 des.) | float | ~  | DS: 2 des. místa, DDL: float (nepřesný typ) |
| Typ nádoby | – | container_type_id | Reference→Typ nádoby | int FK | ✓ | – |
| Vlastnictví nádoby | – | container_ownership | Enumerace | nvarchar(255) FK | ✓ | FK na code_container_ownership |
| Druh odpadu | Ne | garbage_type_id | Reference→Druh odpadu | int FK | ✓ | – |
| Frekvence vývozu | Ne | waste_disposal_frequency_id | Reference→Frekvence vývozu | int FK | ✓ | – |
| Poznámka | – | note | Řetězec(255) | nvarchar(255) | ✓ | – |
| Poznámka 2 | – | note2 | Řetězec(255) | nvarchar(255) | ✓ | – |
| Poznámka 3 | – | note3 | Řetězec(255) | nvarchar(255) | ✓ | – |
| Je aktivní | – | is_active | Boolean | bit | ✓ | – |
| Datum vytvoření | – | created_at | Datum a čas | datetimeoffset | ✓ | – |
| Vytvořil | Ne | created_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj vytvoření | – | creation_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Datum poslední změny | – | updated_at | Datum a čas | datetimeoffset | ✓ | – |
| Provedl poslední změnu | Ne | updated_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj poslední změny | – | last_modification_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Tenant | Ne | tenant_id | Reference→Tenant | int FK | ✓ | – |

### Identifikované delty

- **V DS, chybí v DDL:** *(žádné)*
- **V DDL, chybí v DS:** `id` (interní PK — standardní, DS jej neuvádí explicitně u většiny entit); `waste_disposal_day_id` (FK na waste_disposal_day — DDL obsahuje přímou vazbu na den svozu, DS tento atribut na RPO nemá, dny svozu jsou řešeny přes vazební tabulku entity #13)
- **Typové nesrovnalosti:** `container_count` — DS definuje "Číslo (nezáporné, 2 desetinná místa)", DDL má `float` (nepřesný typ, decimal by byl přesnější)

---

## 2. Položka objednávky

**Priorita: Vysoká** — Rodičovská entita RPO, sdružuje revize.

### Atributový profil z datového slovníku (DS-PP)

| Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis | Persist. ref. | Poznámka |
|---------|-----------|--------|----------|-----------|-------|---------------|----------|
| Externí identifikátor | Řetězec | 255 | – | Ne | Identifikátor instance v externím systému. | – | Jedinečný v rámci Objednávky. |
| Číslo položky objednávky | Řetězec | 30 | – | Ano | Číslo položky objednávky. | – | Jedinečné v rámci Objednávky. |
| Objednávka | Reference | – | Objednávka | Ano | Objednávka, pod kterou položka spadá. | Ne | – |
| Nejnovější revize položky objednávky | Reference | – | Revize položky objednávky | Ne | Nejnovější RPO dle platnosti. |  | Systémový atribut (výkonnostní optimalizace). |
| Je aktivní | Boolean | – | – | Ano | Zda je instance aktivní. | – | Systémový atribut. |
| Datum vytvoření | Datum a čas | – | – | Ano | Datum a čas vytvoření. | – | Systémový atribut. |
| Vytvořil | Reference | – | Uživatel | Ano | Uživatel, který vytvořil záznam. | Ne | Systémový atribut. |
| Zdroj vytvoření | Enumerace | – | Zdroj změny | Ano | Zdroj vytvoření záznamu. | – | Systémový atribut. |
| Datum poslední změny | Datum a čas | – | – | Ne | Datum a čas poslední změny. | – | Systémový atribut. |
| Provedl poslední změnu | Reference | – | Uživatel | Ne | Uživatel poslední změny. | Ne | Systémový atribut. |
| Zdroj poslední změny | Enumerace | – | Zdroj změny | Ne | Zdroj poslední změny. | – | Systémový atribut. |
| Tenant | Reference | – | Tenant | Ano | Rozlišení firmy/organizace. | Ne | Systémový atribut. |

### Odpovídající tabulka v DDL (DDL-PP)

**Table name:** `PasPort_MPGSK.dbo.order_item`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | – |
| external_id | nvarchar(255) | NULL | – | – |
| number | nvarchar(30) | NOT NULL | – | – |
| order_id | int | NOT NULL | order(id) | – |
| is_active | bit | NOT NULL | – | 1 |
| tenant_id | int | NOT NULL | tenant(id) | – |
| created_at | datetimeoffset | NOT NULL | – | getutcdate() |
| created_by | int | NOT NULL | user(id) | – |
| creation_source_id | int | NOT NULL | code_modification_source(id) | – |
| updated_at | datetimeoffset | NULL | – | – |
| updated_by | int | NULL | user(id) | – |
| last_modification_source_id | int | NULL | code_modification_source(id) | – |
| latest_order_item_revision_id | int | NULL | order_item_revision(id) | – |

### Mapování DS-PP <> DDL-PP

| Atribut (DS) | Persist. ref. | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|---------------|----------|-----------|-------|----------|
| *(implicitní PK)* | – | id | – | int IDENTITY | – | – |
| Externí identifikátor | – | external_id | Řetězec(255) | nvarchar(255) | ✓ | – |
| Číslo položky objednávky | – | number | Řetězec(30) | nvarchar(30) | ✓ | – |
| Objednávka | Ne | order_id | Reference→Objednávka | int FK | ✓ | – |
| Nejnovější revize položky objednávky |  | latest_order_item_revision_id | Reference→RPO | int FK | ✓ | – |
| Je aktivní | – | is_active | Boolean | bit | ✓ | – |
| Datum vytvoření | – | created_at | Datum a čas | datetimeoffset | ✓ | – |
| Vytvořil | Ne | created_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj vytvoření | – | creation_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Datum poslední změny | – | updated_at | Datum a čas | datetimeoffset | ✓ | – |
| Provedl poslední změnu | Ne | updated_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj poslední změny | – | last_modification_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Tenant | Ne | tenant_id | Reference→Tenant | int FK | ✓ | – |

### Identifikované delty

- **V DS, chybí v DDL:** *(žádné)*
- **V DDL, chybí v DS:** `id` (standardní interní PK)
- **Typové nesrovnalosti:** *(žádné)*

---

## 3. Objednávka

**Priorita: Střední** — Kontejner pro položky, vazba na zákazníka.

### Atributový profil z datového slovníku (DS-PP)

| Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis | Persist. ref. | Poznámka |
|---------|-----------|--------|----------|-----------|-------|---------------|----------|
| Externí identifikátor | Řetězec | 255 | – | Ne | Identifikátor instance v externím systému. | – | Jedinečný v rámci Provozovny. |
| Číslo objednávky | Řetězec | 30 | – | Ano | Číslo objednávky. | – | – |
| Odběratel | Reference | – | Zákazník | Ano | Zákazník — konkrétní odběratel. | Ne | – |
| Konečný příjemce | Reference | – | Zákazník | Ne | Zákazník — konkrétní konečný příjemce. | Ne | Na rozhraní pro MPG SK aktuálně povinný. |
| Platnost od | Datum | – | – | Ne | Začátek platnosti objednávky. | – | – |
| Platnost do | Datum | – | – | Ne | Konec platnosti objednávky. | – | – |
| Provozovna | Reference | – | Provozovna | Ano | Provozovna, pod kterou objednávka spadá. | Ne | – |
| Je aktivní | Boolean | – | – | Ano | Zda je instance aktivní. | – | Systémový atribut. |
| Datum vytvoření | Datum a čas | – | – | Ano | Datum a čas vytvoření. | – | Systémový atribut. |
| Vytvořil | Reference | – | Uživatel | Ano | Uživatel, který vytvořil záznam. | Ne | Systémový atribut. |
| Zdroj vytvoření | Enumerace | – | Zdroj změny | Ano | Zdroj vytvoření záznamu. | – | Systémový atribut. |
| Datum poslední změny | Datum a čas | – | – | Ne | Datum a čas poslední změny. | – | Systémový atribut. |
| Provedl poslední změnu | Reference | – | Uživatel | Ne | Uživatel poslední změny. | Ne | Systémový atribut. |
| Zdroj poslední změny | Enumerace | – | Zdroj změny | Ne | Zdroj poslední změny. | – | Systémový atribut. |
| Tenant | Reference | – | Tenant | Ano | Rozlišení firmy/organizace. | Ne | Systémový atribut. |

### Odpovídající tabulka v DDL (DDL-PP)

**Table name:** `PasPort_MPGSK.dbo.[order]`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | – |
| external_id | nvarchar(255) | NULL | – | – |
| number | nvarchar(30) | NOT NULL | – | – |
| customer_id | int | NOT NULL | customer(id) | – |
| end_customer_id | int | NULL | customer(id) | – |
| valid_from | date | NULL | – | – |
| valid_to | date | NULL | – | – |
| organization_unit_id | int | NOT NULL | organization_unit(id) | – |
| is_active | bit | NOT NULL | – | 1 |
| tenant_id | int | NOT NULL | tenant(id) | – |
| created_at | datetimeoffset | NOT NULL | – | getutcdate() |
| created_by | int | NOT NULL | user(id) | – |
| creation_source_id | int | NOT NULL | code_modification_source(id) | – |
| updated_at | datetimeoffset | NULL | – | – |
| updated_by | int | NULL | user(id) | – |
| last_modification_source_id | int | NULL | code_modification_source(id) | – |

### Mapování DS-PP <> DDL-PP

| Atribut (DS) | Persist. ref. | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|---------------|----------|-----------|-------|----------|
| *(implicitní PK)* | – | id | – | int IDENTITY | – | – |
| Externí identifikátor | – | external_id | Řetězec(255) | nvarchar(255) | ✓ | – |
| Číslo objednávky | – | number | Řetězec(30) | nvarchar(30) | ✓ | – |
| Odběratel | Ne | customer_id | Reference→Zákazník | int FK | ✓ | – |
| Konečný příjemce | Ne | end_customer_id | Reference→Zákazník | int FK | ✓ | – |
| Platnost od | – | valid_from | Datum | date | ✓ | – |
| Platnost do | – | valid_to | Datum | date | ✓ | – |
| Provozovna | Ne | organization_unit_id | Reference→Provozovna | int FK | ✓ | – |
| Je aktivní | – | is_active | Boolean | bit | ✓ | – |
| Datum vytvoření | – | created_at | Datum a čas | datetimeoffset | ✓ | – |
| Vytvořil | Ne | created_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj vytvoření | – | creation_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Datum poslední změny | – | updated_at | Datum a čas | datetimeoffset | ✓ | – |
| Provedl poslední změnu | Ne | updated_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj poslední změny | – | last_modification_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Tenant | Ne | tenant_id | Reference→Tenant | int FK | ✓ | – |

### Identifikované delty

- **V DS, chybí v DDL:** *(žádné)*
- **V DDL, chybí v DS:** `id` (standardní interní PK)
- **Typové nesrovnalosti:** *(žádné)*

---

## 4. Stanoviště

**Priorita: Vysoká** — Souřadnice pro lokalizaci OS v RP.

### Atributový profil z datového slovníku (DS-PP)

| Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis | Persist. ref. | Poznámka |
|---------|-----------|--------|----------|-----------|-------|---------------|----------|
| Interní identifikátor | Celé číslo | – | – | Ano | Interní identifikátor záznamu v DB. | – | Jedinečný v rámci tabulky. |
| Externí identifikátor | Řetězec | 255 | – | Ne | Identifikátor instance v externím systému. | – | Jedinečný v rámci tabulky. |
| Název | Řetězec | 255 | – | Ano | Název stanoviště. | – | – |
| Provozovna | Reference | – | Provozovna | Ano | Provozovna, pod kterou stanoviště spadá. | Ne | – |
| Adresa | Reference | – | Adresa | Ano | Adresa stanoviště. | Ne | – |
| Souřadnice | Point | – | – | Ne | Souřadnice stanoviště. | Ne | – |
| Stav geokódování | Enumerace | – | Stav geokódování | Ano | Stav geokódování adresy. | – | – |
| Je sklad nádob | Boolean | – | – | Ano | Zda je stanoviště skladem nádob. | – | Migrace: výchozí FALSE. |
| Poznámka | Řetězec | 255 | – | Ne | Poznámka ke stanovišti. | – | – |
| Výchozí fotografie | Reference | – | Fotografie stanoviště | Ne | Výchozí fotografie stanoviště. | Ne | – |
| Je aktivní | Boolean | – | – | Ano | Zda je instance aktivní. | – | Systémový atribut. |
| Datum vytvoření | Datum a čas | – | – | Ano | Datum a čas vytvoření. | – | Systémový atribut. |
| Vytvořil | Reference | – | Uživatel | Ano | Uživatel, který vytvořil záznam. | Ne | Systémový atribut. |
| Zdroj vytvoření | Enumerace | – | Zdroj změny | Ano | Zdroj vytvoření záznamu. | – | Systémový atribut. |
| Datum poslední změny | Datum a čas | – | – | Ne | Datum a čas poslední změny. | – | Systémový atribut. |
| Provedl poslední změnu | Reference | – | Uživatel | Ne | Uživatel poslední změny. | Ne | Systémový atribut. |
| Zdroj poslední změny | Enumerace | – | Zdroj změny | Ne | Zdroj poslední změny. | – | Systémový atribut. |
| Tenant | Reference | – | Tenant | Ano | Rozlišení firmy/organizace. | Ne | Systémový atribut. |

### Odpovídající tabulka v DDL (DDL-PP)

**Table name:** `PasPort_MPGSK.dbo.site`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | – |
| name | nvarchar(255) | NOT NULL | – | – |
| organization_unit_id | int | NOT NULL | organization_unit(id) | – |
| address_id | int | NOT NULL | address(id) | – |
| lat | float | NULL | – | – |
| lng | float | NULL | – | – |
| note | nvarchar(255) | NULL | – | – |
| is_active | bit | NOT NULL | – | 1 |
| created_at | datetimeoffset | NOT NULL | – | getutcdate() |
| created_by | int | NOT NULL | user(id) | – |
| updated_at | datetimeoffset | NULL | – | – |
| updated_by | int | NULL | user(id) | – |
| tenant_id | int | NOT NULL | tenant(id) | – |
| external_id | nvarchar(255) | NULL | – | – |
| geom | geography | NULL | – | – |
| geocoding_state | int | NOT NULL | code_geocoding_state(id) | – |
| default_photo_id | int | NULL | photos(id) | – |
| name_search | nvarchar(255) | NULL | – | – |
| last_modification_source_id | int | NULL | code_modification_source(id) | – |
| creation_source_id | int | NOT NULL | code_modification_source(id) | – |
| is_container_warehouse | bit | NOT NULL | – | 0 |

### Mapování DS-PP <> DDL-PP

| Atribut (DS) | Persist. ref. | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|---------------|----------|-----------|-------|----------|
| Interní identifikátor | – | id | Celé číslo | int IDENTITY | ✓ | – |
| Externí identifikátor | – | external_id | Řetězec(255) | nvarchar(255) | ✓ | – |
| Název | – | name | Řetězec(255) | nvarchar(255) | ✓ | – |
| Provozovna | Ne | organization_unit_id | Reference→Provozovna | int FK | ✓ | – |
| Adresa | Ne | address_id | Reference→Adresa | int FK | ✓ | – |
| Souřadnice | Ne | lat + lng + geom | Point | float + float + geography | ~ | DS: jeden Point; DDL: lat/lng (float) + geom (geography) |
| Stav geokódování | – | geocoding_state | Enumerace | int FK | ✓ | – |
| Je sklad nádob | – | is_container_warehouse | Boolean | bit | ✓ | – |
| Poznámka | – | note | Řetězec(255) | nvarchar(255) | ✓ | – |
| Výchozí fotografie | Ne | default_photo_id | Reference→Fotografie | int FK | ✓ | – |
| Je aktivní | – | is_active | Boolean | bit | ✓ | – |
| Datum vytvoření | – | created_at | Datum a čas | datetimeoffset | ✓ | – |
| Vytvořil | Ne | created_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj vytvoření | – | creation_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Datum poslední změny | – | updated_at | Datum a čas | datetimeoffset | ✓ | – |
| Provedl poslední změnu | Ne | updated_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj poslední změny | – | last_modification_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Tenant | Ne | tenant_id | Reference→Tenant | int FK | ✓ | – |

### Identifikované delty

- **V DS, chybí v DDL:** *(žádné)*
- **V DDL, chybí v DS:** `name_search` (vyhledávací sloupec — technický, generovaný); `geom` (geography — doplňuje lat/lng pro prostorové dotazy)
- **Typové nesrovnalosti:** `Souřadnice` — DS definuje typ `Point` (jeden atribut); DDL realizuje jako 3 sloupce: `lat` (float), `lng` (float), `geom` (geography). Jedná se o validní implementační rozhodnutí.

---

## 5. Nádoba

**Priorita: Vysoká** — Vazba na skupinu odpadu, přiřazení ke stanovišti a RPO.

### Atributový profil z datového slovníku (DS-PP)

| Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis | Persist. ref. | Poznámka |
|---------|-----------|--------|----------|-----------|-------|---------------|----------|
| Externí identifikátor | Řetězec | 255 | – | Ne | Identifikátor instance v externím systému. | – | Jedinečný v rámci tabulky. |
| Zdroj externího identifikátoru | Boolean | – | – | Ano | Zda byl ext. ID získán z externího systému. | – | Dočasné řešení. Systémový atribut. |
| Evidenční číslo | Řetězec | 20 | – | Ano | Evidenční číslo nádoby. | – | Jedinečné v rámci Provozovny. |
| Interní číslo | Řetězec | 20 | – | Ne | Interní číslo nádoby. | – | Jedinečné v rámci Provozovny. |
| Provozovna | Reference | – | Provozovna | Ano | Provozovna, pod kterou nádoba spadá. | Ne | – |
| Vlastnictví | Enumerace | – | Vlastnictví nádoby | Ne | Informace o vlastnictví nádoby. | – | – |
| Stav | Enumerace | – | Stav nádoby | Ano | Stav nádoby. | – | – |
| Rok výroby | Číslo (nezáporné, celé) | – | – | Ne | Rok výroby nádoby. | – | – |
| Poznámka | Řetězec | 255 | – | Ne | Poznámka k nádobě. | – | – |
| Veřejná poznámka | Řetězec | 255 | – | Ne | Veřejná poznámka k nádobě. | – | Migrace: výchozí NULL. |
| Skupina odpadu | Reference | – | Skupina odpadu | Ne | Skupina odpadu pro nádobu. | Ne | – |
| Kategorie nádoby | Reference | – | Kategorie nádoby | Ne | Kategorie nádoby. | Ne | – |
| Vzor nádoby | Reference | – | Vzor nádoby | Ne | Vzor určující vlastnosti nádoby. | Ne | – |
| Výchozí fotografie | Reference | – | Fotografie nádoby | Ne | Výchozí fotografie nádoby. | Ne | – |
| Poslední záznam o pasportizaci nádoby | Reference | – | Záznam o pasportizaci nádoby | Ne | Poslední záznam o pasportizaci. | Ne | – |
| Vyřazena ke dni | Datum | – | – | Podmíněná | Datum, ke kterému je nádoba vyřazena. | – | – |
| Datum vyřazení | Datum a čas | – | – | Podmíněná | Datum a čas vyřazení. | – | Systémový atribut. |
| Vyřadil | Reference | – | Uživatel | Podmíněná | Uživatel, který provedl vyřazení. | Ne | Systémový atribut. |
| Zdroj vyřazení | Enumerace | – | Zdroj změny | Podmíněná | Zdroj vyřazení. | – | Systémový atribut. |
| Je aktivní | Boolean | – | – | Ano | Zda je instance aktivní. | – | Systémový atribut. |
| Datum vytvoření | Datum a čas | – | – | Ano | Datum a čas vytvoření. | – | Systémový atribut. |
| Vytvořil | Reference | – | Uživatel | Ano | Uživatel, který vytvořil záznam. | Ne | Systémový atribut. |
| Zdroj vytvoření | Enumerace | – | Zdroj změny | Ano | Zdroj vytvoření záznamu. | – | Systémový atribut. |
| Datum poslední změny | Datum a čas | – | – | Ne | Datum a čas poslední změny. | – | Systémový atribut. |
| Provedl poslední změnu | Reference | – | Uživatel | Ne | Uživatel poslední změny. | Ne | Systémový atribut. |
| Zdroj poslední změny | Enumerace | – | Zdroj změny | Ne | Zdroj poslední změny. | – | Systémový atribut. |
| Tenant | Reference | – | Tenant | Ano | Rozlišení firmy/organizace. | Ne | Systémový atribut. |

### Odpovídající tabulka v DDL (DDL-PP)

**Table name:** `PasPort_MPGSK.dbo.container`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | – |
| evidence_number | nvarchar(255) | NOT NULL | – | – |
| organization_unit_id | int | NOT NULL | organization_unit(id) | – |
| ownership_status | nvarchar(255) | NULL | code_container_ownership(name) | – |
| status | nvarchar(255) | NOT NULL | code_container_status(name) | – |
| note | nvarchar(255) | NULL | – | – |
| category_id | int | NULL | container_category(id) | – |
| pattern_id | int | NULL | container_pattern(id) | – |
| default_photo_id | int | NULL | photos(id) | – |
| is_active | bit | NOT NULL | – | 1 |
| created_at | datetimeoffset | NOT NULL | – | getutcdate() |
| updated_at | datetimeoffset | NOT NULL | – | getutcdate() |
| created_by | int | NULL | user(id) | – |
| updated_by | int | NULL | user(id) | – |
| tenant_id | int | NOT NULL | tenant(id) | – |
| external_id | nvarchar(255) | NULL | – | – |
| garbage_group_id | int | NULL | garbage_group(id) | – |
| evidence_number_search | nvarchar(255) | NULL | – | – |
| internal_number | nvarchar(20) | NULL | – | – |
| last_modification_source_id | int | NULL | code_modification_source(id) | – |
| creation_source_id | int | NOT NULL | code_modification_source(id) | – |
| last_container_pasport_record_id | int | NULL | container_pasport_record(id) | – |
| discarded_at | datetimeoffset | NULL | – | – |
| discard_to_date | datetimeoffset | NULL | – | – |
| discarded_by | int | NULL | user(id) | – |
| discard_source_id | int | NULL | code_modification_source(id) | – |
| year_production | int | NULL | – | – |
| external_id_source | bit | NOT NULL | – | 0 |
| public_note | nvarchar(255) | NULL | – | – |

### Mapování DS-PP <> DDL-PP

| Atribut (DS) | Persist. ref. | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|---------------|----------|-----------|-------|----------|
| *(implicitní PK)* | – | id | – | int IDENTITY | – | – |
| Externí identifikátor | – | external_id | Řetězec(255) | nvarchar(255) | ✓ | – |
| Zdroj ext. identifikátoru | – | external_id_source | Boolean | bit | ✓ | – |
| Evidenční číslo | – | evidence_number | Řetězec(20) | nvarchar(255) | ~ | DS: max 20; DDL: 255 (šířší rozsah) |
| Interní číslo | – | internal_number | Řetězec(20) | nvarchar(20) | ✓ | – |
| Provozovna | Ne | organization_unit_id | Reference→Provozovna | int FK | ✓ | – |
| Vlastnictví | – | ownership_status | Enumerace | nvarchar(255) FK | ✓ | – |
| Stav | – | status | Enumerace | nvarchar(255) FK | ✓ | – |
| Rok výroby | – | year_production | Číslo (celé) | int | ✓ | – |
| Poznámka | – | note | Řetězec(255) | nvarchar(255) | ✓ | – |
| Veřejná poznámka | – | public_note | Řetězec(255) | nvarchar(255) | ✓ | – |
| Skupina odpadu | Ne | garbage_group_id | Reference→Skupina odpadu | int FK | ✓ | – |
| Kategorie nádoby | Ne | category_id | Reference→Kategorie | int FK | ✓ | – |
| Vzor nádoby | Ne | pattern_id | Reference→Vzor | int FK | ✓ | – |
| Výchozí fotografie | Ne | default_photo_id | Reference→Fotografie | int FK | ✓ | – |
| Poslední záznam o pasportizaci | Ne | last_container_pasport_record_id | Reference | int FK | ✓ | – |
| Vyřazena ke dni | – | discard_to_date | Datum | datetimeoffset | ~ | DS: Datum; DDL: datetimeoffset (širší) |
| Datum vyřazení | – | discarded_at | Datum a čas | datetimeoffset | ✓ | – |
| Vyřadil | Ne | discarded_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj vyřazení | – | discard_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Je aktivní | – | is_active | Boolean | bit | ✓ | – |
| Datum vytvoření | – | created_at | Datum a čas | datetimeoffset | ✓ | – |
| Vytvořil | Ne | created_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj vytvoření | – | creation_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Datum poslední změny | – | updated_at | Datum a čas | datetimeoffset | ✓ | – |
| Provedl poslední změnu | Ne | updated_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj poslední změny | – | last_modification_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Tenant | Ne | tenant_id | Reference→Tenant | int FK | ✓ | – |

### Identifikované delty

- **V DS, chybí v DDL:** *(žádné)*
- **V DDL, chybí v DS:** `id` (interní PK); `evidence_number_search` (technický vyhledávací sloupec)
- **Typové nesrovnalosti:**
  - `evidence_number` — DS: Řetězec(20), DDL: nvarchar(255). DDL má širší rozsah.
  - `discard_to_date` — DS: Datum (bez času), DDL: datetimeoffset (s časem a offset). Širší typ v DDL.

---

## 6. Skupina odpadu

**Priorita: Kritická** — Nový atribut na RPO, sdílený číselník PP a RP.

### Atributový profil z datového slovníku (DS-PP)

| Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis | Persist. ref. | Poznámka |
|---------|-----------|--------|----------|-----------|-------|---------------|----------|
| Externí identifikátor | Řetězec | 255 | – | Ne | Identifikátor instance v externím systému. | – | Jedinečný v rámci tabulky. |
| Zkrácený název | Řetězec | 20 | – | Ne | Zkrácený název skupiny odpadu. | – | – |
| Název | Řetězec | 50 | – | Ano | Název skupiny odpadu. | – | – |
| Popis | Řetězec | 255 | – | Ne | Popis skupiny odpadu. | – | – |
| Barva | Reference | – | Barva skupiny odpadu | Ne | Barva skupiny odpadu. | – | – |
| Je aktivní | Boolean | – | – | Ano | Zda je instance aktivní. | – | Systémový atribut. |
| Datum vytvoření | Datum a čas | – | – | Ano | Datum a čas vytvoření. | – | Systémový atribut. |
| Vytvořil | Reference | – | Uživatel | Ano | Uživatel, který vytvořil záznam. | Ne | Systémový atribut. |
| Zdroj vytvoření | Enumerace | – | Zdroj změny | Ano | Zdroj vytvoření záznamu. | – | Systémový atribut. |
| Datum poslední změny | Datum a čas | – | – | Ne | Datum a čas poslední změny. | – | Systémový atribut. |
| Provedl poslední změnu | Reference | – | Uživatel | Ne | Uživatel poslední změny. | Ne | Systémový atribut. |
| Zdroj poslední změny | Enumerace | – | Zdroj změny | Ne | Zdroj poslední změny. | – | Systémový atribut. |
| Tenant | Reference | – | Tenant | Ano | Rozlišení firmy/organizace. | Ne | Systémový atribut. |

### Odpovídající tabulka v DDL (DDL-PP)

**Table name:** `PasPort_MPGSK.dbo.garbage_group`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | – |
| name | nvarchar(255) | NOT NULL | – | – |
| description | nvarchar(255) | NULL | – | – |
| is_active | bit | NOT NULL | – | 1 |
| created_at | datetimeoffset | NOT NULL | – | getutcdate() |
| updated_at | datetimeoffset | NOT NULL | – | getutcdate() |
| created_by | int | NULL | user(id) | – |
| updated_by | int | NULL | user(id) | – |
| tenant_id | int | NOT NULL | tenant(id) | – |
| external_id | nvarchar(255) | NULL | – | – |
| short_name | nvarchar(255) | NULL | – | – |
| creation_source_id | int | NOT NULL | code_modification_source(id) | 1 |
| last_modification_source_id | int | NULL | code_modification_source(id) | – |
| garbage_group_color_id | int | NULL | garbage_group_color(id) | – |

### Mapování DS-PP <> DDL-PP

| Atribut (DS) | Persist. ref. | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|---------------|----------|-----------|-------|----------|
| *(implicitní PK)* | – | id | – | int IDENTITY | – | – |
| Externí identifikátor | – | external_id | Řetězec(255) | nvarchar(255) | ✓ | – |
| Zkrácený název | – | short_name | Řetězec(20) | nvarchar(255) | ~ | DS: max 20; DDL: 255 |
| Název | – | name | Řetězec(50) | nvarchar(255) | ~ | DS: max 50; DDL: 255 |
| Popis | – | description | Řetězec(255) | nvarchar(255) | ✓ | – |
| Barva | – | garbage_group_color_id | Reference→Barva | int FK | ✓ | – |
| Je aktivní | – | is_active | Boolean | bit | ✓ | – |
| Datum vytvoření | – | created_at | Datum a čas | datetimeoffset | ✓ | – |
| Vytvořil | Ne | created_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj vytvoření | – | creation_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Datum poslední změny | – | updated_at | Datum a čas | datetimeoffset | ✓ | – |
| Provedl poslední změnu | Ne | updated_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj poslední změny | – | last_modification_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Tenant | Ne | tenant_id | Reference→Tenant | int FK | ✓ | – |

### Identifikované delty

- **V DS, chybí v DDL:** *(žádné)*
- **V DDL, chybí v DS:** `id` (interní PK)
- **Typové nesrovnalosti:**
  - `name` — DS: Řetězec(50), DDL: nvarchar(255)
  - `short_name` — DS: Řetězec(20), DDL: nvarchar(255)

---

## 7. Druh odpadu

**Priorita: Vysoká** — Mapování druh na skupinu.

### Atributový profil z datového slovníku (DS-PP)

| Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis | Persist. ref. | Poznámka |
|---------|-----------|--------|----------|-----------|-------|---------------|----------|
| Externí identifikátor | Řetězec | 255 | – | Ne | Identifikátor instance v externím systému. | – | Jedinečný v rámci tabulky. |
| Kód | Řetězec | 15 | – | Ano | Kód odpadu. | – | – |
| Popis | Řetězec | 255 | – | Ne | Popis odpadu. | – | – |
| Kategorie | Řetězec | 4 | – | Ano | Kategorie odpadu. | – | – |
| Je aktivní | Boolean | – | – | Ano | Zda je instance aktivní. | – | Systémový atribut. |
| Datum vytvoření | Datum a čas | – | – | Ano | Datum a čas vytvoření. | – | Systémový atribut. |
| Vytvořil | Reference | – | Uživatel | Ano | Uživatel, který vytvořil záznam. | Ne | Systémový atribut. |
| Zdroj vytvoření | Enumerace | – | Zdroj změny | Ano | Zdroj vytvoření záznamu. | – | Systémový atribut. |
| Datum poslední změny | Datum a čas | – | – | Ne | Datum a čas poslední změny. | – | Systémový atribut. |
| Provedl poslední změnu | Reference | – | Uživatel | Ne | Uživatel poslední změny. | Ne | Systémový atribut. |
| Zdroj poslední změny | Enumerace | – | Zdroj změny | Ne | Zdroj poslední změny. | – | Systémový atribut. |
| Tenant | Reference | – | Tenant | Ano | Rozlišení firmy/organizace. | Ne | Systémový atribut. |

### Odpovídající tabulka v DDL (DDL-PP)

**Table name:** `PasPort_MPGSK.dbo.garbage_type`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | – |
| external_id | nvarchar(255) | NULL | – | – |
| code | nvarchar(15) | NOT NULL | – | – |
| description | nvarchar(255) | NULL | – | – |
| category | nvarchar(4) | NOT NULL | – | – |
| is_active | bit | NOT NULL | – | 1 |
| tenant_id | int | NOT NULL | tenant(id) | – |
| created_at | datetimeoffset | NOT NULL | – | getutcdate() |
| created_by | int | NOT NULL | user(id) | – |
| creation_source_id | int | NOT NULL | code_modification_source(id) | – |
| updated_at | datetimeoffset | NULL | – | – |
| updated_by | int | NULL | user(id) | – |
| last_modification_source_id | int | NOT NULL | code_modification_source(id) | – |

### Mapování DS-PP <> DDL-PP

| Atribut (DS) | Persist. ref. | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|---------------|----------|-----------|-------|----------|
| *(implicitní PK)* | – | id | – | int IDENTITY | – | – |
| Externí identifikátor | – | external_id | Řetězec(255) | nvarchar(255) | ✓ | – |
| Kód | – | code | Řetězec(15) | nvarchar(15) | ✓ | – |
| Popis | – | description | Řetězec(255) | nvarchar(255) | ✓ | – |
| Kategorie | – | category | Řetězec(4) | nvarchar(4) | ✓ | – |
| Je aktivní | – | is_active | Boolean | bit | ✓ | – |
| Datum vytvoření | – | created_at | Datum a čas | datetimeoffset | ✓ | – |
| Vytvořil | Ne | created_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj vytvoření | – | creation_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Datum poslední změny | – | updated_at | Datum a čas | datetimeoffset | ✓ | – |
| Provedl poslední změnu | Ne | updated_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj poslední změny | – | last_modification_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Tenant | Ne | tenant_id | Reference→Tenant | int FK | ✓ | – |

### Identifikované delty

- **V DS, chybí v DDL:** *(žádné)*
- **V DDL, chybí v DS:** `id` (interní PK)
- **Typové nesrovnalosti:** *(žádné)*
- **Poznámka k integraci:** DS nemá explicitní vazbu druh odpadu na skupinu odpadu. Mapování druh na skupinu není v DS-PP definováno — pravděpodobně řešeno na aplikační úrovni nebo přes jiný systém.

---

## 8. Typ nádoby

**Priorita: Vysoká** — Nový atribut service time.

### Atributový profil z datového slovníku (DS-PP)

| Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis | Persist. ref. | Poznámka |
|---------|-----------|--------|----------|-----------|-------|---------------|----------|
| Externí identifikátor | Řetězec | 255 | – | Ne | Identifikátor instance v externím systému. | – | Jedinečný v rámci tabulky. |
| Název | Řetězec | 50 | – | Ano | Název typu nádoby. | – | – |
| Popis | Řetězec | 255 | – | Ne | Popis typu nádoby. | – | – |
| Provozovna | Reference | – | Provozovna | Ano | Provozovna, pod kterou typ nádoby spadá. | Ne | Rozlišení na úrovni složeného klíče. |
| Oblast použití | Řetězec | 15 | – | Ano | Oblast použití (RP, Pasport apod.). | – | Řešeno na úrovni synchronizace. |
| Je aktivní | Boolean | – | – | Ano | Zda je instance aktivní. | – | Systémový atribut. |
| Datum vytvoření | Datum a čas | – | – | Ano | Datum a čas vytvoření. | – | Systémový atribut. |
| Vytvořil | Reference | – | Uživatel | Ano | Uživatel, který vytvořil záznam. | Ne | Systémový atribut. |
| Zdroj vytvoření | Enumerace | – | Zdroj změny | Ano | Zdroj vytvoření záznamu. | – | Systémový atribut. |
| Datum poslední změny | Datum a čas | – | – | Ne | Datum a čas poslední změny. | – | Systémový atribut. |
| Provedl poslední změnu | Reference | – | Uživatel | Ne | Uživatel poslední změny. | Ne | Systémový atribut. |
| Zdroj poslední změny | Enumerace | – | Zdroj změny | Ne | Zdroj poslední změny. | – | Systémový atribut. |
| Tenant | Reference | – | Tenant | Ano | Rozlišení firmy/organizace. | Ne | Systémový atribut. |

### Odpovídající tabulka v DDL (DDL-PP)

**Table name:** `PasPort_MPGSK.dbo.container_type`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | – |
| name | nvarchar(255) | NOT NULL | – | – |
| description | nvarchar(255) | NULL | – | – |
| tenant_id | int | NOT NULL | tenant(id) | – |
| is_active | bit | NOT NULL | – | 1 |
| created_at | datetimeoffset | NULL | – | getutcdate() |
| updated_at | datetimeoffset | NULL | – | getutcdate() |
| external_id | nvarchar(255) | NULL | – | – |
| created_by | int | NULL | user(id) | – |
| updated_by | int | NULL | user(id) | – |
| creation_source_id | int | NOT NULL | code_modification_source(id) | 1 |
| last_modification_source_id | int | NULL | code_modification_source(id) | – |

### Mapování DS-PP <> DDL-PP

| Atribut (DS) | Persist. ref. | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|---------------|----------|-----------|-------|----------|
| *(implicitní PK)* | – | id | – | int IDENTITY | – | – |
| Externí identifikátor | – | external_id | Řetězec(255) | nvarchar(255) | ✓ | – |
| Název | – | name | Řetězec(50) | nvarchar(255) | ~ | DS: max 50; DDL: 255 |
| Popis | – | description | Řetězec(255) | nvarchar(255) | ✓ | – |
| Provozovna | Ne | *(chybí)* | Reference→Provozovna | – | ✗ | **V DDL chybí FK na organization_unit!** |
| Oblast použití | – | *(chybí)* | Řetězec(15) | – | ✗ | **V DDL chybí sloupec.** |
| Je aktivní | – | is_active | Boolean | bit | ✓ | – |
| Datum vytvoření | – | created_at | Datum a čas | datetimeoffset | ✓ | – |
| Vytvořil | Ne | created_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj vytvoření | – | creation_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Datum poslední změny | – | updated_at | Datum a čas | datetimeoffset | ✓ | – |
| Provedl poslední změnu | Ne | updated_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj poslední změny | – | last_modification_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Tenant | Ne | tenant_id | Reference→Tenant | int FK | ✓ | – |

### Identifikované delty

- **V DS, chybí v DDL:**
  - `Provozovna` (organization_unit_id) — DS definuje vazbu na Provozovnu, DDL tabulka container_type ji nemá. Rozlišení se pravděpodobně řeší přes tenant_id nebo na aplikační úrovni.
  - `Oblast použití` — DS definuje řetězcový atribut pro oblast (RP, Pasport), v DDL chybí. Pravděpodobně řešeno synchronizací mimo DB.
- **V DDL, chybí v DS:** `id` (interní PK)
- **Typové nesrovnalosti:** `name` — DS: Řetězec(50), DDL: nvarchar(255)

---

## 9. Frekvence vývozu

**Priorita: Střední** — Kontext pro rozvrhy.

### Atributový profil z datového slovníku (DS-PP)

| Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis | Persist. ref. | Poznámka |
|---------|-----------|--------|----------|-----------|-------|---------------|----------|
| Externí identifikátor | Řetězec | 255 | – | Ne | Identifikátor instance v externím systému. | – | Jedinečný v rámci tabulky. |
| Název | Řetězec | 50 | – | Ano | Název frekvence vývozu. | – | – |
| Popis | Řetězec | 80 | – | Ne | Popis frekvence vývozu. | – | – |
| Skupina | Řetězec | 80 | – | Ne | Skupina frekvence vývozu. | – | – |
| Provozovna | Reference | – | Provozovna | Ne | Provozovna pro frekvenci vývozu. | Ne | Pokud prázdná, není omezena dostupnost. |
| Je aktivní | Boolean | – | – | Ano | Zda je instance aktivní. | – | Systémový atribut. |
| Datum vytvoření | Datum a čas | – | – | Ano | Datum a čas vytvoření. | – | Systémový atribut. |
| Vytvořil | Reference | – | Uživatel | Ano | Uživatel, který vytvořil záznam. | Ne | Systémový atribut. |
| Zdroj vytvoření | Enumerace | – | Zdroj změny | Ano | Zdroj vytvoření záznamu. | – | Systémový atribut. |
| Datum poslední změny | Datum a čas | – | – | Ne | Datum a čas poslední změny. | – | Systémový atribut. |
| Provedl poslední změnu | Reference | – | Uživatel | Ne | Uživatel poslední změny. | Ne | Systémový atribut. |
| Zdroj poslední změny | Enumerace | – | Zdroj změny | Ne | Zdroj poslední změny. | – | Systémový atribut. |
| Tenant | Reference | – | Tenant | Ano | Rozlišení firmy/organizace. | Ne | Systémový atribut. |

### Odpovídající tabulka v DDL (DDL-PP)

**Table name:** `PasPort_MPGSK.dbo.waste_disposal_frequency`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | – |
| external_id | nvarchar(255) | NULL | – | – |
| name | nvarchar(50) | NOT NULL | – | – |
| description | nvarchar(80) | NULL | – | – |
| [group] | nvarchar(80) | NULL | – | – |
| is_active | bit | NOT NULL | – | 1 |
| tenant_id | int | NOT NULL | tenant(id) | – |
| created_at | datetimeoffset | NOT NULL | – | getutcdate() |
| created_by | int | NOT NULL | user(id) | – |
| creation_source_id | int | NOT NULL | code_modification_source(id) | – |
| updated_at | datetimeoffset | NULL | – | – |
| updated_by | int | NULL | user(id) | – |
| last_modification_source_id | int | NULL | code_modification_source(id) | – |
| external_organization_unit_id | nvarchar(255) | NULL | – | – |

### Mapování DS-PP <> DDL-PP

| Atribut (DS) | Persist. ref. | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|---------------|----------|-----------|-------|----------|
| *(implicitní PK)* | – | id | – | int IDENTITY | – | – |
| Externí identifikátor | – | external_id | Řetězec(255) | nvarchar(255) | ✓ | – |
| Název | – | name | Řetězec(50) | nvarchar(50) | ✓ | – |
| Popis | – | description | Řetězec(80) | nvarchar(80) | ✓ | – |
| Skupina | – | [group] | Řetězec(80) | nvarchar(80) | ✓ | DDL escapuje rezervované slovo [group] |
| Provozovna | Ne | external_organization_unit_id | Reference→Provozovna | nvarchar(255) | ~ | DS: Reference (FK int); DDL: textový ext. ID provozovny |
| Je aktivní | – | is_active | Boolean | bit | ✓ | – |
| Datum vytvoření | – | created_at | Datum a čas | datetimeoffset | ✓ | – |
| Vytvořil | Ne | created_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj vytvoření | – | creation_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Datum poslední změny | – | updated_at | Datum a čas | datetimeoffset | ✓ | – |
| Provedl poslední změnu | Ne | updated_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj poslední změny | – | last_modification_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Tenant | Ne | tenant_id | Reference→Tenant | int FK | ✓ | – |

### Identifikované delty

- **V DS, chybí v DDL:** *(žádné — Provozovna je řešena jinak)*
- **V DDL, chybí v DS:** `id` (interní PK)
- **Typové nesrovnalosti:** `Provozovna` — DS definuje Reference (FK), DDL používá `external_organization_unit_id` jako textový identifikátor (nvarchar(255)), nikoliv int FK. Jedná se o volnou vazbu přes externí identifikátor místo referencí integrity.

---

## 10. Adresa

**Priorita: Vysoká** — V PP chybí lat/lng.

### Atributový profil z datového slovníku (DS-PP)

| Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis | Persist. ref. | Poznámka |
|---------|-----------|--------|----------|-----------|-------|---------------|----------|
| Externí identifikátor | Řetězec | 255 | – | Ne | Identifikátor instance v externím systému. | – | Jedinečný v rámci tabulky. |
| Externí identifikátor v registru adres | Řetězec | 10 | – | Ne | Identifikátor v registru adres (RÚIAN). | – | Společný napříč systémy. Aktuálně není na UI. |
| Ulice | Řetězec | 80 | – | Ne | Název ulice. | – | – |
| Číslo popisné | Řetězec | 15 | – | Ne | Číslo popisné. | – | – |
| Číslo orientační | Řetězec | 15 | – | Ne | Číslo orientační. | – | – |
| Město | Řetězec | 80 | – | Ano | Název města. | – | – |
| PSČ | Řetězec | 15 | – | Ne | Poštovní směrovací číslo. | – | – |
| Stát | Řetězec | 50 | – | Ano | Stát. | – | – |
| Je aktivní | Boolean | – | – | Ano | Zda je instance aktivní. | – | Systémový atribut. |
| Datum vytvoření | Datum a čas | – | – | Ano | Datum a čas vytvoření. | – | Systémový atribut. |
| Vytvořil | Reference | – | Uživatel | Ano | Uživatel, který vytvořil záznam. | Ne | Systémový atribut. |
| Zdroj vytvoření | Enumerace | – | Zdroj změny | Ano | Zdroj vytvoření záznamu. | – | Systémový atribut. |
| Datum poslední změny | Datum a čas | – | – | Ne | Datum a čas poslední změny. | – | Systémový atribut. |
| Provedl poslední změnu | Reference | – | Uživatel | Ne | Uživatel poslední změny. | Ne | Systémový atribut. |
| Zdroj poslední změny | Enumerace | – | Zdroj změny | Ne | Zdroj poslední změny. | – | Systémový atribut. |
| Tenant | Reference | – | Tenant | Ano | Rozlišení firmy/organizace. | Ne | Systémový atribut. |

### Odpovídající tabulka v DDL (DDL-PP)

**Table name:** `PasPort_MPGSK.dbo.address`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | – |
| street | nvarchar(80) | NULL | – | – |
| registry_number | nvarchar(15) | NULL | – | – |
| orientation_number | nvarchar(15) | NULL | – | – |
| city | nvarchar(50) | NOT NULL | – | – |
| postal_code | nvarchar(15) | NULL | – | – |
| country | nvarchar(50) | NOT NULL | – | – |
| created_at | datetimeoffset | NULL | – | getutcdate() |
| updated_at | datetimeoffset | NULL | – | – |
| is_active | bit | NOT NULL | – | 1 |
| external_id | nvarchar(255) | NULL | – | – |
| tenant_id | int | NOT NULL | tenant(id) | – |
| created_by | int | NOT NULL | user(id) | – |
| updated_by | bit | NULL | – | – |
| last_modification_source_id | int | NULL | code_modification_source(id) | – |
| creation_source_id | int | NOT NULL | code_modification_source(id) | – |
| full_address | nvarchar(255) | NULL | – | – |
| address_register_external_id | nvarchar(255) | NULL | – | – |

### Mapování DS-PP <> DDL-PP

| Atribut (DS) | Persist. ref. | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|---------------|----------|-----------|-------|----------|
| *(implicitní PK)* | – | id | – | int IDENTITY | – | – |
| Externí identifikátor | – | external_id | Řetězec(255) | nvarchar(255) | ✓ | – |
| Ext. ID v registru adres | – | address_register_external_id | Řetězec(10) | nvarchar(255) | ~ | DS: max 10; DDL: 255 |
| Ulice | – | street | Řetězec(80) | nvarchar(80) | ✓ | – |
| Číslo popisné | – | registry_number | Řetězec(15) | nvarchar(15) | ✓ | – |
| Číslo orientační | – | orientation_number | Řetězec(15) | nvarchar(15) | ✓ | – |
| Město | – | city | Řetězec(80) | nvarchar(50) | ~ | **DS: max 80; DDL: max 50 — DDL je užší!** |
| PSČ | – | postal_code | Řetězec(15) | nvarchar(15) | ✓ | – |
| Stát | – | country | Řetězec(50) | nvarchar(50) | ✓ | – |
| Je aktivní | – | is_active | Boolean | bit | ✓ | – |
| Datum vytvoření | – | created_at | Datum a čas | datetimeoffset | ✓ | – |
| Vytvořil | Ne | created_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj vytvoření | – | creation_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Datum poslední změny | – | updated_at | Datum a čas | datetimeoffset | ✓ | – |
| Provedl poslední změnu | Ne | updated_by | Reference→Uživatel | int FK→bit | ✗ | **DDL: bit místo int FK!** |
| Zdroj poslední změny | – | last_modification_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Tenant | Ne | tenant_id | Reference→Tenant | int FK | ✓ | – |

### Identifikované delty

- **V DS, chybí v DDL:** *(žádné)*
- **V DDL, chybí v DS:** `id` (interní PK); `full_address` (generovaný/konsolidovaný sloupec pro celou adresu)
- **Typové nesrovnalosti:**
  - `Město` (city) — **DS: Řetězec(80), DDL: nvarchar(50)**. DDL je užší, hrozí ořezání dlouhých názvů.
  - `Provedl poslední změnu` (updated_by) — **DS: Reference na Uživatel (int FK), DDL: bit**. Toto je pravděpodobně chyba v DDL (bit místo int). Potenciální bug.
  - `Ext. ID v registru adres` — DS: Řetězec(10), DDL: nvarchar(255). DDL má výrazně širší rozsah.

---

## 11. Přiřazení nádoby ke stanovišti

**Priorita: Střední** — Kontext pro lokalizaci OS.

### Atributový profil z datového slovníku (DS-PP)

| Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis | Persist. ref. | Poznámka |
|---------|-----------|--------|----------|-----------|-------|---------------|----------|
| Externí identifikátor | Řetězec | 255 | – | Ne | Identifikátor instance v externím systému. | – | Jedinečný v rámci tabulky. |
| Nádoba | Reference | – | Nádoba | Ano | Nádoba v rámci přiřazení. | Ne | – |
| Stanoviště | Reference | – | Stanoviště | Ano | Stanoviště v rámci přiřazení. | Ne | – |
| Datum přistavení | Datum a čas | – | – | Ano | Začátek období přistavení nádoby. | – | – |
| Datum stažení | Datum a čas | – | – | Ne | Konec období přistavení. | – | – |
| Poznámka | Řetězec | 255 | – | Ne | Poznámka k přiřazení. | – | – |
| Je aktivní | Boolean | – | – | Ano | Zda je instance aktivní. | – | Systémový atribut. |
| Datum vytvoření | Datum a čas | – | – | Ano | Datum a čas vytvoření. | – | Systémový atribut. |
| Vytvořil | Reference | – | Uživatel | Ano | Uživatel, který vytvořil záznam. | Ne | Systémový atribut. |
| Zdroj vytvoření | Enumerace | – | Zdroj změny | Ano | Zdroj vytvoření záznamu. | – | Systémový atribut. |
| Datum poslední změny | Datum a čas | – | – | Ne | Datum a čas poslední změny. | – | Systémový atribut. |
| Provedl poslední změnu | Reference | – | Uživatel | Ne | Uživatel poslední změny. | Ne | Systémový atribut. |
| Zdroj poslední změny | Enumerace | – | Zdroj změny | Ne | Zdroj poslední změny. | – | Systémový atribut. |
| Tenant | Reference | – | Tenant | Ano | Rozlišení firmy/organizace. | Ne | Systémový atribut. |

### Odpovídající tabulka v DDL (DDL-PP)

**Table name:** `PasPort_MPGSK.dbo.site_container_assignment`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | – |
| container_id | int | NOT NULL | container(id) | – |
| site_id | int | NOT NULL | site(id) | – |
| valid_from | datetimeoffset | NOT NULL | – | – |
| valid_to | datetimeoffset | NULL | – | – |
| note | nvarchar(255) | NULL | – | – |
| is_active | bit | NOT NULL | – | 1 |
| created_at | datetimeoffset | NOT NULL | – | getutcdate() |
| created_by | int | NOT NULL | user(id) | – |
| updated_at | datetimeoffset | NULL | – | – |
| updated_by | int | NULL | user(id) | – |
| tenant_id | int | NOT NULL | tenant(id) | – |
| external_id | nvarchar(255) | NULL | – | – |
| last_modification_source_id | int | NULL | code_modification_source(id) | – |
| creation_source_id | int | NOT NULL | code_modification_source(id) | – |

### Mapování DS-PP <> DDL-PP

| Atribut (DS) | Persist. ref. | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|---------------|----------|-----------|-------|----------|
| *(implicitní PK)* | – | id | – | int IDENTITY | – | – |
| Externí identifikátor | – | external_id | Řetězec(255) | nvarchar(255) | ✓ | – |
| Nádoba | Ne | container_id | Reference→Nádoba | int FK | ✓ | – |
| Stanoviště | Ne | site_id | Reference→Stanoviště | int FK | ✓ | – |
| Datum přistavení | – | valid_from | Datum a čas | datetimeoffset | ✓ | – |
| Datum stažení | – | valid_to | Datum a čas | datetimeoffset | ✓ | – |
| Poznámka | – | note | Řetězec(255) | nvarchar(255) | ✓ | – |
| Je aktivní | – | is_active | Boolean | bit | ✓ | – |
| Datum vytvoření | – | created_at | Datum a čas | datetimeoffset | ✓ | – |
| Vytvořil | Ne | created_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj vytvoření | – | creation_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Datum poslední změny | – | updated_at | Datum a čas | datetimeoffset | ✓ | – |
| Provedl poslední změnu | Ne | updated_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj poslední změny | – | last_modification_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Tenant | Ne | tenant_id | Reference→Tenant | int FK | ✓ | – |

### Identifikované delty

- **V DS, chybí v DDL:** *(žádné)*
- **V DDL, chybí v DS:** `id` (interní PK)
- **Typové nesrovnalosti:** *(žádné)*

---

## 12. Přiřazení nádoby k položce objednávky

**Priorita: Střední** — Kontext pro generování OS.

### Atributový profil z datového slovníku (DS-PP)

| Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis | Persist. ref. | Poznámka |
|---------|-----------|--------|----------|-----------|-------|---------------|----------|
| Externí identifikátor | Řetězec | 255 | – | Ne | Identifikátor instance v externím systému. | – | Jedinečný v rámci tabulky. |
| Nádoba | Reference | – | Nádoba | Ano | Nádoba v rámci přiřazení. | Ne | – |
| Položka objednávky | Reference | – | Položka objednávky | Ano | Položka objednávky v rámci přiřazení. | Ne | – |
| Platnost od | Datum a čas | – | – | Ano | Začátek období přiřazení. | – | – |
| Platnost do | Datum a čas | – | – | Ne | Konec období přiřazení. | – | – |
| Poznámka | Řetězec | 255 | – | Ne | Poznámka k přiřazení. | – | – |
| Je aktivní | Boolean | – | – | Ano | Zda je instance aktivní. | – | Systémový atribut. |
| Datum vytvoření | Datum a čas | – | – | Ano | Datum a čas vytvoření. | – | Systémový atribut. |
| Vytvořil | Reference | – | Uživatel | Ano | Uživatel, který vytvořil záznam. | Ne | Systémový atribut. |
| Zdroj vytvoření | Enumerace | – | Zdroj změny | Ano | Zdroj vytvoření záznamu. | – | Systémový atribut. |
| Datum poslední změny | Datum a čas | – | – | Ne | Datum a čas poslední změny. | – | Systémový atribut. |
| Provedl poslední změnu | Reference | – | Uživatel | Ne | Uživatel poslední změny. | Ne | Systémový atribut. |
| Zdroj poslední změny | Enumerace | – | Zdroj změny | Ne | Zdroj poslední změny. | – | Systémový atribut. |
| Tenant | Reference | – | Tenant | Ano | Rozlišení firmy/organizace. | Ne | Systémový atribut. |

### Odpovídající tabulka v DDL (DDL-PP)

**Table name:** `PasPort_MPGSK.dbo.container_order_item_assignment`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | – |
| external_id | nvarchar(255) | NULL | – | – |
| container_id | int | NOT NULL | container(id) | – |
| order_item_id | int | NOT NULL | order_item(id) | – |
| valid_from | datetimeoffset | NOT NULL | – | – |
| valid_to | datetimeoffset | NULL | – | – |
| note | nvarchar(255) | NULL | – | – |
| is_active | bit | NOT NULL | – | 1 |
| tenant_id | int | NOT NULL | tenant(id) | – |
| created_at | datetimeoffset | NOT NULL | – | getutcdate() |
| created_by | int | NOT NULL | user(id) | – |
| creation_source_id | int | NOT NULL | code_modification_source(id) | – |
| updated_at | datetimeoffset | NULL | – | – |
| updated_by | int | NULL | user(id) | – |
| last_modification_source_id | int | NULL | code_modification_source(id) | – |

### Mapování DS-PP <> DDL-PP

| Atribut (DS) | Persist. ref. | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|---------------|----------|-----------|-------|----------|
| *(implicitní PK)* | – | id | – | int IDENTITY | – | – |
| Externí identifikátor | – | external_id | Řetězec(255) | nvarchar(255) | ✓ | – |
| Nádoba | Ne | container_id | Reference→Nádoba | int FK | ✓ | – |
| Položka objednávky | Ne | order_item_id | Reference→Položka objednávky | int FK | ✓ | – |
| Platnost od | – | valid_from | Datum a čas | datetimeoffset | ✓ | – |
| Platnost do | – | valid_to | Datum a čas | datetimeoffset | ✓ | – |
| Poznámka | – | note | Řetězec(255) | nvarchar(255) | ✓ | – |
| Je aktivní | – | is_active | Boolean | bit | ✓ | – |
| Datum vytvoření | – | created_at | Datum a čas | datetimeoffset | ✓ | – |
| Vytvořil | Ne | created_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj vytvoření | – | creation_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Datum poslední změny | – | updated_at | Datum a čas | datetimeoffset | ✓ | – |
| Provedl poslední změnu | Ne | updated_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj poslední změny | – | last_modification_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Tenant | Ne | tenant_id | Reference→Tenant | int FK | ✓ | – |

### Identifikované delty

- **V DS, chybí v DDL:** *(žádné)*
- **V DDL, chybí v DS:** `id` (interní PK)
- **Typové nesrovnalosti:** *(žádné)*

---

## 13. Přiřazení dní svozu k RPO

**Priorita: Vysoká** — Současný mechanismus vs. nové Rozvrhy.

> **Poznámka:** V DS-PP je tato entita pojmenována "Přiřazení základních dní svozu k revizi položky objednávky". V DDL: `waste_disposal_day_order_item_revision_assignment`.

### Atributový profil z datového slovníku (DS-PP)

| Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis | Persist. ref. | Poznámka |
|---------|-----------|--------|----------|-----------|-------|---------------|----------|
| Externí identifikátor | Řetězec | 255 | – | Ne | Identifikátor instance v externím systému. | – | Jedinečný v rámci tabulky. |
| Revize položky objednávky | Reference | – | Revize položky objednávky | Ano | RPO v rámci přiřazení. | Ne | – |
| Základní dny svozu | Reference | – | Základní dny svozu | Ano | Základní dny svozu v rámci přiřazení. | Ne | – |
| Je aktivní | Boolean | – | – | Ano | Zda je instance aktivní. | – | Systémový atribut. |
| Datum vytvoření | Datum a čas | – | – | Ano | Datum a čas vytvoření. | – | Systémový atribut. |
| Vytvořil | Reference | – | Uživatel | Ano | Uživatel, který vytvořil záznam. | Ne | Systémový atribut. |
| Zdroj vytvoření | Enumerace | – | Zdroj změny | Ano | Zdroj vytvoření záznamu. | – | Systémový atribut. |
| Datum poslední změny | Datum a čas | – | – | Ne | Datum a čas poslední změny. | – | Systémový atribut. |
| Provedl poslední změnu | Reference | – | Uživatel | Ne | Uživatel poslední změny. | Ne | Systémový atribut. |
| Zdroj poslední změny | Enumerace | – | Zdroj změny | Ne | Zdroj poslední změny. | – | Systémový atribut. |
| Tenant | Reference | – | Tenant | Ano | Rozlišení firmy/organizace. | Ne | Systémový atribut. |

### Referenční entita: Základní dny svozu

**DDL table:** `PasPort_MPGSK.dbo.waste_disposal_day`

Pro kontext, entita "Základní dny svozu" z DS-PP obsahuje:

| Atribut | Datový typ | Rozsah | Asociace | Povinnost |
|---------|-----------|--------|----------|-----------|
| Externí identifikátor | Řetězec | 255 | – | Ne |
| Dny svozu | Řetězec | 50 | – | Ano |
| Počet svozů za měsíc | Číslo (kladné, celé) | – | – | Ano |
| Provozovna | Reference | – | Provozovna | Ne |

### Odpovídající tabulka v DDL (DDL-PP)

**Table name:** `PasPort_MPGSK.dbo.waste_disposal_day_order_item_revision_assignment`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | – |
| external_id | nvarchar(255) | NULL | – | – |
| order_item_revision_id | int | NOT NULL | order_item_revision(id) | – |
| waste_disposal_day_id | int | NOT NULL | waste_disposal_day(id) | – |
| is_active | bit | NOT NULL | – | 1 |
| tenant_id | int | NOT NULL | tenant(id) | – |
| created_at | datetimeoffset | NOT NULL | – | getutcdate() |
| created_by | int | NOT NULL | user(id) | – |
| creation_source_id | int | NOT NULL | code_modification_source(id) | – |
| updated_at | datetimeoffset | NULL | – | – |
| updated_by | int | NULL | user(id) | – |
| last_modification_source_id | int | NULL | code_modification_source(id) | – |

### Mapování DS-PP <> DDL-PP

| Atribut (DS) | Persist. ref. | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|---------------|----------|-----------|-------|----------|
| *(implicitní PK)* | – | id | – | int IDENTITY | – | – |
| Externí identifikátor | – | external_id | Řetězec(255) | nvarchar(255) | ✓ | – |
| Revize položky objednávky | Ne | order_item_revision_id | Reference→RPO | int FK | ✓ | – |
| Základní dny svozu | Ne | waste_disposal_day_id | Reference→Základní dny svozu | int FK | ✓ | – |
| Je aktivní | – | is_active | Boolean | bit | ✓ | – |
| Datum vytvoření | – | created_at | Datum a čas | datetimeoffset | ✓ | – |
| Vytvořil | Ne | created_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj vytvoření | – | creation_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Datum poslední změny | – | updated_at | Datum a čas | datetimeoffset | ✓ | – |
| Provedl poslední změnu | Ne | updated_by | Reference→Uživatel | int FK | ✓ | – |
| Zdroj poslední změny | – | last_modification_source_id | Enumerace→Zdroj změny | int FK | ✓ | – |
| Tenant | Ne | tenant_id | Reference→Tenant | int FK | ✓ | – |

### Identifikované delty

- **V DS, chybí v DDL:** *(žádné)*
- **V DDL, chybí v DS:** `id` (interní PK)
- **Typové nesrovnalosti:** *(žádné)*
- **Poznámka k integraci:** Tato vazební tabulka řeší M:N vztah mezi RPO a dny svozu. V DDL existuje navíc i přímý FK `waste_disposal_day_id` přímo na `order_item_revision` (entita #1), což je redundantní vazba — pravděpodobně starší mechanismus (1:1), zatímco tato tabulka umožňuje M:N.

---

## Souhrnný přehled nalezených delt

### Kritické nálezy

| # | Entita | Delta | Závažnost | Popis |
|---|--------|-------|-----------|-------|
| 1 | Adresa | `updated_by` typ | Vysoká | DDL má `bit` místo `int` FK na user. Pravděpodobně bug v DDL. |
| 2 | Adresa | `city` rozsah | Střední | DS: max 80 znaků, DDL: max 50. Hrozí ořezání. |
| 3 | Typ nádoby | Chybějící `Provozovna` | Střední | DS definuje FK, DDL nemá sloupec organization_unit_id. |
| 4 | Typ nádoby | Chybějící `Oblast použití` | Střední | DS definuje atribut, v DDL chybí. |
| 5 | RPO (order_item_revision) | `waste_disposal_day_id` | Info | Přímý FK v DDL, DS řeší přes vazební tabulku (#13). Redundance. |
| 6 | RPO | `container_count` typ | Nízká | DS: "2 des. místa", DDL: float (nepřesný typ). |

### Technické sloupce v DDL bez protějšku v DS

Tyto sloupce se vyskytují v DDL, ale nemají protějšek v DS-PP. Jedná se o implementační/technické sloupce:

- `id` — interní auto-increment PK (všechny entity)
- `evidence_number_search` — vyhledávací sloupec (Nádoba)
- `name_search` — vyhledávací sloupec (Stanoviště)
- `geom` — geography sloupec pro prostorové dotazy (Stanoviště)
- `full_address` — konsolidovaný textový sloupec (Adresa)

### Typové nesrovnalosti v rozsazích (DS > DDL)

| Entita | Atribut | DS rozsah | DDL rozsah | Směr |
|--------|---------|-----------|------------|------|
| Adresa | Město | 80 | 50 | **DDL je užší** |

### Typové nesrovnalosti v rozsazích (DDL > DS)

| Entita | Atribut | DS rozsah | DDL rozsah | Směr |
|--------|---------|-----------|------------|------|
| Nádoba | Evidenční číslo | 20 | 255 | DDL je širší |
| Skupina odpadu | Název | 50 | 255 | DDL je širší |
| Skupina odpadu | Zkrácený název | 20 | 255 | DDL je širší |
| Typ nádoby | Název | 50 | 255 | DDL je širší |
| Adresa | Ext. ID v registru adres | 10 | 255 | DDL je širší |
| Nádoba | Vyřazena ke dni | Datum | datetimeoffset | DDL je širší |
