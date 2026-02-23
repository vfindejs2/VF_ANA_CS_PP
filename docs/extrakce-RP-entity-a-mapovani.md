# Extrakce entit z DS-RP a mapovani na DDL-RP

> **Datum:** 2026-02-23
> **Zdroje:**
> - DS-RP: `data/input/dokumentace_RP/2026-02-19_technicky-projekt-RP/technicky-projekt/datove-modely/datovy-78681044.html`
> - DDL-RP: `data/input/dokumentace_RP/DDL_RP_DB.txt`
> **Pocet entit:** 11

---

## Obsah

1. [Objednaná služba (Kritická)](#1-objednaná-služba)
2. [Lokace objednané služby (Kritická)](#2-lokace-objednané-služby)
3. [Denní výkon (Vysoká)](#3-denní-výkon)
4. [Položka denního výkonu (Vysoká)](#4-položka-denního-výkonu)
5. [Vozidlo (Vysoká)](#5-vozidlo)
6. [Likvidační místo (Střední)](#6-likvidační-místo)
7. [Druh odpadu likvidačního místa (Střední)](#7-druh-odpadu-likvidačního-místa)
8. [Typ dopravy (Střední)](#8-typ-dopravy)
9. [Typ položky denního výkonu (Vysoká)](#9-typ-položky-denního-výkonu)
10. [Stav objednané služby (Střední)](#10-stav-objednané-služby)
11. [Místo realizace / execution_site (Střední)](#11-místo-realizace)

---

## 1. Objednaná služba

**Priorita: Kritická** | Hlavní entita pro CS -- rozšíření vs. nová tabulka

### Atributový profil z datového slovníku (DS-RP)

| # | Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis |
|---|---------|-----------|--------|----------|-----------|-------|
| 1 | Stav | Enumerace | -- | Stav objednané služby | A | Stav, ve kterém se právě nachází Objednaná služba |
| 2 | Objednaný úkon | Reference | -- | Objednaný úkon | A | Objednaný úkon, ze kterého OS vznikla (vazba na Objednávku) |
| 3 | Konkrétní objednaná nádoba | Celé kladné číslo | 1-255 | -- | A | Označení konkrétní nádoby, pro kterou je OS určena |
| 4 | Datum realizace od | Datum | -- | -- | A | Předpokládané datum realizace -- od |
| 5 | Datum realizace do | Datum | -- | -- | A | Předpokládané datum realizace -- do |
| 6 | Čas realizace od | Čas | -- | -- | A | Předpokládaný čas realizace -- od |
| 7 | Čas realizace do | Čas | -- | -- | A | Předpokládaný čas realizace -- do |
| 8 | Poznámka | Řetězec | 255 | -- | N | Poznámka k OS |
| 9 | Datum vytvoření | Datum a čas | -- | -- | A | Datum a čas vytvoření |
| 10 | Vytvořil | Reference | -- | Uživatel | P | Uživatel, který vytvořil OS |
| 11 | Vytvořeno uživatelem | Boolean | -- | -- | A | TRUE = vytvořena uživatelem, FALSE = generována systémem |
| 12 | Vytvořena na výzvu | Boolean | -- | -- | A | TRUE = vygenerována na výzvu objednavatele |
| 13 | Pořadí v rámci objednávky | Číslo (kladné, celé) | -- | -- | A | Pořadové číslo OS v rámci objednávky |
| 14 | Identifikátor jízdy | Řetězec | 255 | -- | Ne | ID záznamu na straně ERP |
| 15 | Evidenční číslo výchozího vozidla | Řetězec | 255 | -- | Ne | Ev. číslo vozidla z ERP |
| 16 | Výchozí datum realizace | Datum | -- | -- | Ne | Původní datum realizace z ERP |
| 17 | Položka denního výkonu | Reference | -- | Položka denního výkonu | N | Vazba na položku DV |
| 18 | Lokace objednané služby | Reference [] | -- | Lokace objednané služby | P | Posloupnost kroků k provedení |
| 19 | Pořadí v rámci objednané služby | Číslo (kladné, celé) | -- | -- | A | Pořadí části v rámci rozdělené OS |
| 20 | Následující objednaná služba | Reference | -- | Objednaná služba | N | Odkaz na následující rozdělenou OS |
| 21 | Způsob rozdělení následující OS | Enumerace | -- | Způsob rozdělení OS | N | Způsob rozdělení |
| 22 | Předchozí objednaná služba | Reference | -- | Objednaná služba | N | Odkaz na předchozí rozdělenou OS |
| 23 | Způsob rozdělení předchozí OS | Enumerace | -- | Způsob rozdělení OS | N | Způsob rozdělení |
| 24 | Realizace denního výkonu | Reference | -- | Realizace DV | P | Realizace DV, v rámci které byla OS realizována |
| 25 | Pořadí v realizaci DV | Celé kladné číslo | -- | -- | N | Pořadí v kontrole realizace |
| 26 | Realizována | Boolean | -- | -- | N | TRUE = OS skutečně realizována |
| 27 | Způsob přeplánování | Enumerace | -- | Způsob přeplánování OS | N | Jak byla OS dále řešena |
| 28 | Přeplánováno do objednané služby | Reference | -- | Objednaná služba | N | Odkaz na nově vytvořenou OS |
| 29 | Přeplánováno z objednané služby | Reference | -- | Objednaná služba | N | Odkaz na původní OS |
| 30 | Určení | Enumerace | -- | Určení objednané služby | A | (implicitní -- typ OS) |
| 31 | Řetězec ID OS v řetězci | Řetězec | MAX | -- | N | ID OS v řetězci (ordered_service_chain_ids) |

### Odpovídající tabulka v DDL (DDL-RP)

**Table name:** `RoadPlan_MPGSK.dbo.ordered_service`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | -- |
| status | int | NOT NULL | FK -> ordered_service_status(id) | -- |
| container_operation_id | int | NOT NULL | FK -> order_item_container_operation(id) | -- |
| container_id | int | NOT NULL | FK -> order_item_container(id) | -- |
| container_number | int | NOT NULL | -- | -- |
| date_from | date | NULL | -- | -- |
| date_to | date | NULL | -- | -- |
| note | nvarchar(255) | NULL | -- | -- |
| time_from | varchar(150) | NULL | -- | -- |
| time_to | varchar(150) | NULL | -- | -- |
| parent_ordered_service_id | int | NULL | FK -> ordered_service(id) | -- |
| child_ordered_service_id | int | NULL | FK -> ordered_service(id) | -- |
| temporary_storage_ordered_service_id | int | NULL | FK -> ordered_service(id) | -- |
| type | nvarchar(200) | NULL | FK -> ordered_service_type_code(type) | -- |
| day_performance_realization_id | int | NULL | FK -> day_performance_realization(id) | -- |
| is_realized | bit | NOT NULL | -- | -- |
| realization_order | int | NULL | -- | -- |
| is_according_to_plan | bit | NOT NULL | -- | -- |
| confirmed_by | int | NULL | FK -> user(id) | -- |
| rescheduled_to_ordered_service_id | int | NULL | FK -> ordered_service(id) | -- |
| rescheduled_from_ordered_service_id | int | NULL | FK -> ordered_service(id) | -- |
| reschedule_method | nvarchar(200) | NULL | FK -> ordered_service_reschedule_method_code(type) | -- |
| confirmation_datetime | datetimeoffset | NULL | -- | -- |
| created_at | datetimeoffset | NOT NULL | -- | -- |
| updated_at | datetimeoffset | NOT NULL | -- | -- |
| is_created_by_user | bit | NOT NULL | -- | -- |
| created_by | int | NULL | FK -> user(id) | -- |
| parent_ordered_service_split_type | nvarchar(MAX) | NULL | -- | -- |
| child_ordered_service_split_type | nvarchar(MAX) | NULL | -- | -- |
| split_order | int | NULL | -- | -- |
| ordered_service_chain_ids | nvarchar(MAX) | NULL | -- | -- |
| on_demand | bit | NOT NULL | -- | -- |
| ordering_within_order | int | NULL | -- | -- |
| ride_identifier | nvarchar(255) | NULL | -- | -- |
| vehicle_evidence_number | nvarchar(255) | NULL | -- | -- |
| default_realization_date | date | NULL | -- | -- |

### Mapování DS-RP <-> DDL-RP

| Atribut (DS) | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|----------|-----------|-------|----------|
| Stav | status | Enumerace | int (FK) | ok | FK -> ordered_service_status |
| Objednaný úkon | container_operation_id | Reference | int (FK) | ok | FK -> order_item_container_operation |
| Konkrétní objednaná nádoba | container_number | Celé kladné číslo | int | ok | -- |
| -- | container_id | -- | int (FK) | -- | V DDL navíc: FK -> order_item_container |
| Datum realizace od | date_from | Datum | date | ok | -- |
| Datum realizace do | date_to | Datum | date | ok | -- |
| Čas realizace od | time_from | Čas | varchar(150) | ok | Pozor: varchar(150) je netypicky dlouhý pro čas |
| Čas realizace do | time_to | Čas | varchar(150) | ok | Pozor: varchar(150) je netypicky dlouhý pro čas |
| Poznámka | note | Řetězec(255) | nvarchar(255) | ok | -- |
| Datum vytvoření | created_at | Datum a čas | datetimeoffset | ok | -- |
| Vytvořil | created_by | Reference | int (FK) | ok | FK -> user(id) |
| Vytvořeno uživatelem | is_created_by_user | Boolean | bit | ok | -- |
| Vytvořena na výzvu | on_demand | Boolean | bit | ok | -- |
| Pořadí v rámci objednávky | ordering_within_order | Číslo | int | ok | -- |
| Identifikátor jízdy | ride_identifier | Řetězec(255) | nvarchar(255) | ok | -- |
| Evidenční číslo výchozího vozidla | vehicle_evidence_number | Řetězec(255) | nvarchar(255) | ok | -- |
| Výchozí datum realizace | default_realization_date | Datum | date | ok | -- |
| Položka denního výkonu | -- | Reference | -- | CHYBI | Vazba řešena přes day_performance_item (opačná FK) |
| Lokace objednané služby | -- | Reference [] | -- | ok | Řešeno přes ordered_service_location.ordered_service_id |
| Pořadí v rámci OS | split_order | Číslo | int | ok | -- |
| Následující OS | child_ordered_service_id | Reference | int (FK) | ok | -- |
| Způsob rozdělení následující OS | child_ordered_service_split_type | Enumerace | nvarchar(MAX) | ok | -- |
| Předchozí OS | parent_ordered_service_id | Reference | int (FK) | ok | -- |
| Způsob rozdělení předchozí OS | parent_ordered_service_split_type | Enumerace | nvarchar(MAX) | ok | -- |
| Realizace DV | day_performance_realization_id | Reference | int (FK) | ok | -- |
| Pořadí v realizaci DV | realization_order | Celé číslo | int | ok | -- |
| Realizována | is_realized | Boolean | bit | ok | -- |
| Způsob přeplánování | reschedule_method | Enumerace | nvarchar(200) (FK) | ok | FK -> reschedule_method_code |
| Přeplánováno do OS | rescheduled_to_ordered_service_id | Reference | int (FK) | ok | -- |
| Přeplánováno z OS | rescheduled_from_ordered_service_id | Reference | int (FK) | ok | -- |
| Určení | type | Enumerace | nvarchar(200) (FK) | ok | FK -> ordered_service_type_code |
| Řetězec ID OS | ordered_service_chain_ids | Řetězec(MAX) | nvarchar(MAX) | ok | -- |

### Identifikované delty

- **V DS, chybí v DDL:**
  - `Lokace objednané služby` -- řešeno opačnou FK z tabulky `ordered_service_location`, OK
  - `Položka denního výkonu` -- řešeno opačnou FK z tabulky `day_performance_item`, OK

- **V DDL, chybí v DS:**
  - `container_id` (FK -> order_item_container) -- implicitně zahrnuto v DS přes "Objednaný úkon" -> "Objednaná nádoba"
  - `temporary_storage_ordered_service_id` -- odkaz na OS dočasného uskladnění, v DS nespecifikován jako samostatný atribut
  - `is_according_to_plan` -- příznak, zda realizace proběhla dle plánu
  - `confirmed_by` -- uživatel, který potvrdil realizaci
  - `confirmation_datetime` -- čas potvrzení realizace
  - `updated_at` -- systémový sloupec (DS neuvádí explicitně)

- **Typové nesrovnalosti:**
  - `time_from` / `time_to`: DS uvádí typ `Čas`, DDL má `varchar(150)` -- neobvykle dlouhý typ pro čas (formát pravděpodobně JSON/textový)
  - `split_type` sloupce: DS uvádí `Enumerace`, DDL má `nvarchar(MAX)` -- v praxi textová hodnota

---

## 2. Lokace objednané služby

**Priorita: Kritická** | Lokace pro CS -- nové atributy

### Atributový profil z datového slovníku (DS-RP)

| # | Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis |
|---|---------|-----------|--------|----------|-----------|-------|
| 1 | Objednaná služba | Reference | -- | Objednaná služba | N | Reference na odpovídající OS |
| 2 | Pořadí | Kladné celé číslo | -- | -- | A | Pořadí obsluhy lokací |
| 3 | Akce | Enumerace | -- | Akce v lokaci OS | A | Provedená akce |
| 4 | Typ lokace | Enumerace | -- | Typ lokace OS | A | Typ lokace (MR, LM, provozovna...) |
| 5 | Adresa | Reference | -- | Adresa | P | Adresa lokace |
| 6 | Souřadnice | Point | -- | -- | P | GPS souřadnice |
| 7 | Manuální posun souřadnic | Boolean | -- | -- | A | TRUE = manuálně posunuté souřadnice |
| 8 | Místo realizace | Reference | -- | Místo realizace | N | Reference pro aktualizaci |
| 9 | Likvidační místo | Reference | -- | Likvidační místo | P | Povinná pro typ LM |
| 10 | Kód nakládání | Řetězec | 15 | -- | P | Kód nakládání na LM |
| 11 | Provozovna | Reference | -- | Provozovna | P | Povinná pro typ Provozovna |
| 12 | Doba manipulace | Celé kladné číslo | -- | -- | P | Doba manipulace v minutách |
| 13 | Doba trvání | Celé kladné číslo | -- | -- | P | Pro typ Časový interval |
| 14 | Provést | Boolean | -- | -- | A | Bude lokace obsloužena? |
| 15 | Začátek časového okna | Čas | -- | -- | P | Čas od |
| 16 | Konec časového okna | Čas | -- | -- | P | Čas do |
| 17 | Splněno časové okno | Boolean | -- | -- | P | Obslouženo v rámci okna? |
| 18 | Vzdálenost do další lokace | Kladné celé číslo | -- | -- | N | V metrech |
| 19 | Doba jízdy do další lokace | Kladné celé číslo | -- | -- | N | V sekundách |
| 20 | Trasa do další lokace | LineString | -- | -- | N | Geometrie trasy |
| 21 | Poznámka | Řetězec | 255 | -- | N | Poznámka |
| 22 | Monitoring realizace | Boolean | -- | -- | N | Monitorovat realizaci? |
| 23 | Obsluha lokace OS | Reference | -- | Obsluha lokace OS | N | Údaje z monitoringu |
| 24 | Fakturace | Boolean | -- | -- | N | Je vývoz na LM pod paušálem? |
| 25 | Druhy odpadu | Reference [] | -- | Druh odpadu | N | Druhy odpadu na LM |

### Odpovídající tabulka v DDL (DDL-RP)

**Table name:** `RoadPlan_MPGSK.dbo.ordered_service_location`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | -- |
| order | int | NOT NULL | -- | -- |
| action | int | NOT NULL | FK -> ordered_service_location_action(id) | -- |
| type | int | NOT NULL | FK -> ordered_service_location_type(id) | -- |
| address_id | int | NULL | FK -> address(id) | -- |
| lat | float | NULL | -- | -- |
| lng | float | NULL | -- | -- |
| liquidation_site_id | int | NULL | FK -> liquidation_site(id) | -- |
| execute | bit | NULL | -- | -- |
| organization_unit_id | int | NULL | FK -> organization_unit(id) | -- |
| valid | bit | NULL | -- | -- |
| next_location_distance | float | NULL | -- | -- |
| next_location_time | float | NULL | -- | -- |
| route_to_next_location | geometry | NULL | -- | -- |
| ordered_service_id | int | NULL | FK -> ordered_service(id) | -- |
| note | nvarchar(255) | NULL | -- | -- |
| time_window_start | nvarchar(5) | NOT NULL | -- | -- |
| time_window_end | nvarchar(5) | NOT NULL | -- | -- |
| is_time_window_met | bit | NOT NULL | -- | -- |
| duration | int | NULL | -- | -- |
| is_generated | bit | NOT NULL | -- | -- |
| manipulation_time_in_minutes | int | NULL | -- | -- |
| is_monitored | bit | NOT NULL | -- | -- |
| execution_site_id | int | NULL | FK -> execution_site(id) | -- |
| manual_address_change | bit | NOT NULL | -- | -- |
| has_flat_rate | bit | NULL | -- | -- |
| liquidation_garbage_types | nvarchar(MAX) | NULL | -- | -- |
| created_at | datetimeoffset | NOT NULL | -- | -- |
| updated_at | datetimeoffset | NOT NULL | -- | -- |
| loading_code | nvarchar(15) | NULL | -- | -- |

### Mapování DS-RP <-> DDL-RP

| Atribut (DS) | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|----------|-----------|-------|----------|
| Objednaná služba | ordered_service_id | Reference | int (FK) | ok | -- |
| Pořadí | [order] | Kladné celé číslo | int | ok | Rezervované slovo SQL |
| Akce | [action] | Enumerace | int (FK) | ok | FK -> ordered_service_location_action |
| Typ lokace | [type] | Enumerace | int (FK) | ok | FK -> ordered_service_location_type |
| Adresa | address_id | Reference | int (FK) | ok | -- |
| Souřadnice | lat, lng | Point | float, float | ok | Rozpad na 2 sloupce |
| Manuální posun souřadnic | manual_address_change | Boolean | bit | ok | -- |
| Místo realizace | execution_site_id | Reference | int (FK) | ok | -- |
| Likvidační místo | liquidation_site_id | Reference | int (FK) | ok | -- |
| Kód nakládání | loading_code | Řetězec(15) | nvarchar(15) | ok | -- |
| Provozovna | organization_unit_id | Reference | int (FK) | ok | Mapováno na org. unit |
| Doba manipulace | manipulation_time_in_minutes | Celé kladné číslo | int | ok | -- |
| Doba trvání | duration | Celé kladné číslo | int | ok | -- |
| Provést | [execute] | Boolean | bit | ok | Rezervované slovo SQL |
| Začátek časového okna | time_window_start | Čas | nvarchar(5) | ok | Formát "HH:MM" |
| Konec časového okna | time_window_end | Čas | nvarchar(5) | ok | Formát "HH:MM" |
| Splněno časové okno | is_time_window_met | Boolean | bit | ok | -- |
| Vzdálenost do další lokace | next_location_distance | Kladné celé číslo | float | DELTA | DS: celé číslo, DDL: float |
| Doba jízdy do další lokace | next_location_time | Kladné celé číslo | float | DELTA | DS: celé číslo, DDL: float |
| Trasa do další lokace | route_to_next_location | LineString | geometry | ok | -- |
| Poznámka | note | Řetězec(255) | nvarchar(255) | ok | -- |
| Monitoring realizace | is_monitored | Boolean | bit | ok | -- |
| Obsluha lokace OS | -- | Reference | -- | ok | Řešeno tabulkou ordered_service_location_realization |
| Fakturace | has_flat_rate | Boolean | bit | ok | -- |
| Druhy odpadu | liquidation_garbage_types | Reference [] | nvarchar(MAX) | DELTA | DS: reference na entity, DDL: JSON string |

### Identifikované delty

- **V DS, chybí v DDL:**
  - `Obsluha lokace OS` -- řešeno samostatnou tabulkou `ordered_service_location_realization`, OK

- **V DDL, chybí v DS:**
  - `valid` (bit) -- platnost záznamu, v DS nespecifikováno
  - `is_generated` (bit) -- příznak generování systémem
  - `created_at`, `updated_at` -- systémové sloupce

- **Typové nesrovnalosti:**
  - `next_location_distance`: DS = celé číslo, DDL = float
  - `next_location_time`: DS = celé číslo, DDL = float
  - `liquidation_garbage_types`: DS = Reference [] na entity Druh odpadu, DDL = nvarchar(MAX) (JSON)

---

## 3. Denní výkon

**Priorita: Vysoká** | Nový transport_type, vazba na okruhy dne

### Atributový profil z datového slovníku (DS-RP)

| # | Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis |
|---|---------|-----------|--------|----------|-----------|-------|
| 1 | Stav | Enumerace | -- | Stav denního výkonu | A | Stav DV |
| 2 | Provozovna | Reference | -- | Provozovna | A | Provozovna pro DV |
| 3 | Datum realizace | Datum | -- | -- | A | Den realizace |
| 4 | Typ dopravy | Reference | -- | Typ dopravy | A | Typ dopravy pro DV |
| 5 | Vozidlo | Reference | -- | Vozidlo | N | Přiřazené vozidlo |
| 6 | Kód barvy | Řetězec | 7 | -- | N | Barva v aplikaci |
| 7 | Použití přívěsu | Boolean | -- | -- | A | Příznak přívěsu |
| 8 | Použité přívěsy | Reference [] | -- | Přívěs | N | Přiřazené přívěsy |
| 9 | Intervaly omezení | Reference [] | -- | Interval omezení DV | N | Časová omezení |
| 10 | Začátek provozní doby | Čas | -- | -- | A | Čas začátku DV |
| 11 | Délka provozní doby | Celé číslo | -- | -- | A | Délka provozní doby |
| 12 | Počáteční položka DV | Reference | -- | Položka DV | A | Místo začátku DV |
| 13 | Koncová položka DV | Reference | -- | Položka DV | A | Místo konce DV |
| 14 | Položky DV | Reference [] | -- | Položka DV | N | Místa k obsluze |
| 15 | Ujetá vzdálenost celkem | Celé číslo | -- | -- | P | Celková délka trasy |
| 16 | Doba jízdy celkem | Celé číslo | -- | -- | P | Celková doba jízdy |
| 17 | Způsob získání trajektorie | Enumerace | -- | Způsob získání trajektorie DV | Ne | Způsob získání trajektorie |
| 18 | Poznámka | Řetězec | 255 | -- | N | Poznámka |
| 19 | Časové využití | Desetinné číslo | -- | -- | P | Podíl doby jízdy / provozní doba |
| 20 | Podíl km s přívěsem | Desetinné číslo | -- | -- | P | Podíl km s přívěsem |
| 21 | Počet objednaných služeb | Celé číslo | -- | -- | A | Počet OS na DV |
| 22 | Schválený denní výkon | Reference | -- | DV (JSON) | N | Uložený schválený stav |
| 23 | Průběhy editace DV | Reference [] | -- | Průběh editace DV | N | Historie editace |
| 24 | Vytisknuto potvrzení | Boolean | -- | -- | A | Vytištěno potvrzení? |
| 25 | Textová identifikace vozidla | Řetězec | 50 | -- | N | Text. identifikace vozidla |
| 26 | Způsob vyhodnocení realizace | Enumerace | -- | Způsob vyhodnocení realizace | Ano | Zdroj informací o realizaci |
| 27 | Vytvořen uživatelem | Boolean | -- | -- | A | Vytvořen uživatelem? |

### Odpovídající tabulka v DDL (DDL-RP)

**Table name:** `RoadPlan_MPGSK.dbo.day_performance`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | -- |
| status | int | NOT NULL | FK -> day_performance_status(id) | -- |
| organization_unit_id | int | NOT NULL | FK -> organization_unit(id) | -- |
| transport_type | int | NOT NULL | FK -> transport_type(id) | -- |
| vehicle_id | int | NULL | FK -> vehicle(id) | -- |
| realization_date | date | NOT NULL | -- | -- |
| start_time | nvarchar(5) | NOT NULL | -- | -- |
| start_day_performance_item_id | int | NULL | FK -> day_performance_item(id) | -- |
| finish_day_performance_item_id | int | NULL | FK -> day_performance_item(id) | -- |
| route_duration | int | NULL | -- | -- |
| route_distance | int | NULL | -- | -- |
| route_geom | geometry | NULL | -- | -- |
| note | nvarchar(255) | NULL | -- | -- |
| has_trailer | bit | NOT NULL | -- | -- |
| need_reroute | bit | NOT NULL | -- | -- |
| is_confirmation_printed | bit | NOT NULL | -- | -- |
| is_created_by_user | bit | NOT NULL | -- | -- |
| vehicle_identification | nvarchar(MAX) | NULL | -- | -- |
| created_at | datetimeoffset | NOT NULL | -- | -- |
| updated_at | datetimeoffset | NOT NULL | -- | -- |
| realization_type | int | NOT NULL | FK -> vehicle_realizaton_type(id) | -- |
| color | nvarchar(50) | NULL | -- | -- |

### Mapování DS-RP <-> DDL-RP

| Atribut (DS) | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|----------|-----------|-------|----------|
| Stav | status | Enumerace | int (FK) | ok | -- |
| Provozovna | organization_unit_id | Reference | int (FK) | ok | -- |
| Datum realizace | realization_date | Datum | date | ok | -- |
| Typ dopravy | transport_type | Reference | int (FK) | ok | -- |
| Vozidlo | vehicle_id | Reference | int (FK) | ok | -- |
| Kód barvy | color | Řetězec(7) | nvarchar(50) | ok | DDL: delší rozsah |
| Použití přívěsu | has_trailer | Boolean | bit | ok | -- |
| Použité přívěsy | -- | Reference [] | -- | ok | Řešeno tabulkou day_performance_trailers |
| Intervaly omezení | -- | Reference [] | -- | ok | Řešeno tabulkou day_performance_interval_restriction |
| Začátek provozní doby | start_time | Čas | nvarchar(5) | ok | Formát "HH:MM" |
| Délka provozní doby | -- | Celé číslo | -- | CHYBI | V DDL nenalezeno! |
| Počáteční položka DV | start_day_performance_item_id | Reference | int (FK) | ok | -- |
| Koncová položka DV | finish_day_performance_item_id | Reference | int (FK) | ok | -- |
| Položky DV | -- | Reference [] | -- | ok | Řešeno přes day_performance_item.day_performance_id |
| Ujetá vzdálenost celkem | route_distance | Celé číslo | int | ok | -- |
| Doba jízdy celkem | route_duration | Celé číslo | int | ok | -- |
| Způsob získání trajektorie | -- | Enumerace | -- | CHYBI | V DDL nenalezeno |
| Poznámka | note | Řetězec(255) | nvarchar(255) | ok | -- |
| Časové využití | -- | Desetinné číslo | -- | CHYBI | V DDL nenalezeno na úrovni DV |
| Podíl km s přívěsem | -- | Desetinné číslo | -- | CHYBI | V DDL nenalezeno na úrovni DV |
| Počet OS | -- | Celé číslo | -- | CHYBI | V DDL nenalezeno na úrovni DV |
| Schválený DV | -- | Reference (JSON) | -- | ok | Řešeno tabulkou day_performance_snapshot |
| Průběhy editace DV | -- | Reference [] | -- | ok | Pravděpodobně tabulka day_performance_snapshot |
| Vytisknuto potvrzení | is_confirmation_printed | Boolean | bit | ok | -- |
| Textová identifikace vozidla | vehicle_identification | Řetězec(50) | nvarchar(MAX) | ok | DDL: MAX vs DS: 50 |
| Způsob vyhodnocení realizace | realization_type | Enumerace | int (FK) | ok | FK -> vehicle_realizaton_type |
| Vytvořen uživatelem | is_created_by_user | Boolean | bit | ok | -- |

### Identifikované delty

- **V DS, chybí v DDL:**
  - `Délka provozní doby` -- v DDL nenalezen odpovídající sloupec (pravděpodobně odvozen z DV settings)
  - `Způsob získání trajektorie` -- v DDL nenalezen
  - `Časové využití` -- v DDL nenalezeno (ukládáno na úrovni day_performance_realization: percentage_usage)
  - `Podíl km s přívěsem` -- v DDL nenalezeno (ukládáno v day_performance_realization)
  - `Počet objednaných služeb` -- v DDL nenalezeno (odvozeno runtime)

- **V DDL, chybí v DS:**
  - `route_geom` (geometry) -- geometrie trasy (DS uvádí trajektorii na úrovni lokací)
  - `need_reroute` (bit) -- příznak potřeby přetrasování
  - `created_at`, `updated_at` -- systémové sloupce

- **Typové nesrovnalosti:**
  - `vehicle_identification`: DS = Řetězec(50), DDL = nvarchar(MAX)
  - `color`: DS = Řetězec(7), DDL = nvarchar(50) -- DDL umožňuje delší hodnotu

---

## 4. Položka denního výkonu

**Priorita: Vysoká** | Nový typ položky (okruh dne)

### Atributový profil z datového slovníku (DS-RP)

| # | Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis |
|---|---------|-----------|--------|----------|-----------|-------|
| 1 | Pořadí | Kladné celé číslo | -- | -- | A | Pořadí obsluhy |
| 2 | Typ položky | Enumerace | -- | Typ položky DV | A | Typ položky (OS, lokace OS, časový interval, rozdělení) |
| 3 | Denní výkon | Reference | -- | Denní výkon | A | Nadřazený DV |
| 4 | Přívěs | Boolean | -- | -- | A | Použití přívěsu |
| 5 | Objednaná služba | Reference | -- | Objednaná služba | P | OS pro typ položky "Objednaná služba" |
| 6 | Lokace objednané služby | Reference | -- | Lokace objednané služby | P | Lokace OS pro ostatní typy |

### Odpovídající tabulka v DDL (DDL-RP)

**Table name:** `RoadPlan_MPGSK.dbo.day_performance_item`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | -- |
| order | int | NOT NULL | -- | -- |
| day_performance_id | int | NOT NULL | FK -> day_performance(id) ON DELETE CASCADE | -- |
| duration | int | NULL | -- | -- |
| has_trailer | bit | NOT NULL | -- | -- |
| ordered_service_id | int | NULL | FK -> ordered_service(id) | -- |
| ordered_service_location_id | int | NULL | FK -> ordered_service_location(id) | -- |
| day_performance_item_type | int | NOT NULL | FK -> day_performance_item_type(id) | -- |
| note | nvarchar(255) | NULL | -- | -- |

### Mapování DS-RP <-> DDL-RP

| Atribut (DS) | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|----------|-----------|-------|----------|
| Pořadí | [order] | Kladné celé číslo | int | ok | -- |
| Typ položky | day_performance_item_type | Enumerace | int (FK) | ok | FK -> day_performance_item_type |
| Denní výkon | day_performance_id | Reference | int (FK) | ok | -- |
| Přívěs | has_trailer | Boolean | bit | ok | -- |
| Objednaná služba | ordered_service_id | Reference | int (FK) | ok | -- |
| Lokace objednané služby | ordered_service_location_id | Reference | int (FK) | ok | -- |

### Identifikované delty

- **V DS, chybí v DDL:** (nic)

- **V DDL, chybí v DS:**
  - `duration` (int) -- doba trvání položky, v DS nespecifikována na této úrovni
  - `note` (nvarchar(255)) -- poznámka k položce

- **Typové nesrovnalosti:** (žádné)

---

## 5. Vozidlo

**Priorita: Vysoká** | Nové kapacitní atributy pro SP

> **Poznámka:** DS-RP rozlišuje dva koncepty: **Objekt** (obecný -- vozidlo, přívěs) a **Vozidlo** (specializace objektu). V DDL existuje jedna tabulka `vehicle` pro obojí. Níže jsou uvedeny atributy z obou DS sekcí relevantní pro mapování.

### Atributový profil z datového slovníku (DS-RP) -- sekce "Vozidlo"

| # | Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis |
|---|---------|-----------|--------|----------|-----------|-------|
| 1 | Externí identifikátor FLW | Číslo | -- | -- | N | ID ve FLW |
| 2 | Externí identifikátor ERP | Číslo | -- | -- | N | ID v ERP |
| 3 | Název | Řetězec | 50 | -- | A | Název vozidla |
| 4 | RZ | Řetězec | 50 | -- | N | Registrační značka |
| 5 | Skupina | Řetězec | 50 | -- | N | Skupina vozidla |
| 6 | Umístění | Řetězec | 255 | -- | N | Hierarchie skupin |
| 7 | Typ dopravy | Reference | -- | Typ dopravy | N | Typ dopravy vozidla |
| 8 | Provozovna | Reference | -- | Provozovna | N | Provozovna vozidla |

### Atributy z DS-RP sekce "Objekt" (sdílené pro Vozidlo i Přívěs)

| # | Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis |
|---|---------|-----------|--------|----------|-----------|-------|
| 1 | Externí identifikátor FLW | Řetězec | 50 | -- | Ne | ID ve FLWW2 |
| 2 | Externí identifikátor ERP | Řetězec | 50 | -- | Ne | ID v ERP |
| 3 | Název | Řetězec | 50 | -- | Ano | Název objektu |
| 4 | Identifikátor objektu / RZ | Řetězec | 255 | -- | Ne | Textová identifikace (RZ) |
| 5 | Evidenční číslo | Řetězec | 255 | -- | Ne | Ev. číslo |
| 6 | Druh vozidla | Celé číslo | int | -- | Ano | Kódové označení druhu |
| 7 | Skupina | Řetězec | 50 | -- | Ne | Skupina |
| 8 | Umístění | Řetězec | 255 | -- | Ne | Hierarchie skupin |
| 9 | Provozovna | Reference | -- | Provozovna | Ne | Provozovna |
| 10 | Typ objektu | Reference | -- | Typ objektu | Ano | Typ objektu (vozidlo/přívěs) |
| 11 | Datum zařazení | Datum a čas | -- | -- | Ne | Datum zařazení do provozu |
| 12 | Datum vyřazení | Datum a čas | -- | -- | Ne | Datum vyřazení z provozu |
| 13 | Způsob vyhodnocení realizace | Enumerace | -- | Způsob vyhodnocení realizace | Ne | Způsob vyhodnocení realizace DV |
| 14 | Je aktivní | Boolean | -- | -- | Ano | Aktivní záznam |
| 15 | Datum vytvoření | Datum a čas | -- | -- | Ano | Systémový atribut |
| 16 | Datum poslední změny | Datum a čas | -- | -- | Ano | Systémový atribut |

### Odpovídající tabulka v DDL (DDL-RP)

**Table name:** `RoadPlan_MPGSK.dbo.vehicle`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | -- |
| name | nvarchar(100) | NULL | -- | -- |
| registration_plate | nvarchar(255) | NULL | -- | -- |
| organization_unit_id | int | NULL | FK -> organization_unit(id) | -- |
| flww_id | int | NULL | UNIQUE | -- |
| identification | nvarchar(255) | NULL | -- | -- |
| winyx_id | int | NULL | -- | -- |
| object_type | int | NULL | -- | -- |
| device_type_id | int | NULL | -- | -- |
| color | nvarchar(50) | NULL | -- | -- |
| color_inherited | bit | NULL | -- | -- |
| cost_center_code | nvarchar(255) | NULL | -- | -- |
| group_id | int | NULL | -- | -- |
| parent_name | nvarchar(255) | NULL | -- | -- |
| vehicle_full_path | nvarchar(255) | NULL | -- | -- |
| driver_id | nvarchar(10) | NULL | -- | -- |
| has_gps | bit | NULL | -- | -- |
| cost_center_id | nvarchar(10) | NULL | -- | -- |
| active | bit | NULL | -- | -- |
| evidence_number | nvarchar(255) | NULL | -- | -- |
| vin | nvarchar(255) | NULL | -- | -- |
| vehicle_type_id | int | NOT NULL | FK -> vehicle_type_enum(id) | -- |
| created_at | datetimeoffset | NOT NULL | -- | -- |
| updated_at | datetimeoffset | NOT NULL | -- | -- |
| external_id | nvarchar(255) | NULL | -- | -- |
| sort_id | int | NULL | -- | -- |
| date_insertion | datetimeoffset | NULL | -- | -- |
| date_rejection | datetimeoffset | NULL | -- | -- |
| realization_type | int | NULL | FK -> vehicle_realizaton_type(id) | -- |

**Vazební tabulka:** `RoadPlan_MPGSK.dbo.vehicle__transport_type`

| Sloupec | SQL typ | FK |
|---------|---------|-----|
| vehicle_id | int | FK -> vehicle(id) |
| transport_type_id | int | FK -> transport_type(id) |

### Mapování DS-RP <-> DDL-RP

| Atribut (DS) | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|----------|-----------|-------|----------|
| Ext. ID FLW | flww_id | Číslo/Řetězec | int | ok | DS Vozidlo uvádí Číslo, DS Objekt uvádí Řetězec |
| Ext. ID ERP | external_id | Číslo/Řetězec | nvarchar(255) | ok | -- |
| Název | name | Řetězec(50) | nvarchar(100) | ok | DDL: delší rozsah |
| RZ / Identifikátor | registration_plate | Řetězec(50/255) | nvarchar(255) | ok | -- |
| Evidenční číslo | evidence_number | Řetězec(255) | nvarchar(255) | ok | -- |
| Druh vozidla | vehicle_type_id | Celé číslo | int (FK) | ok | FK -> vehicle_type_enum |
| Skupina | parent_name | Řetězec(50) | nvarchar(255) | ok | DDL: delší rozsah |
| Umístění | vehicle_full_path | Řetězec(255) | nvarchar(255) | ok | -- |
| Provozovna | organization_unit_id | Reference | int (FK) | ok | -- |
| Typ objektu | object_type | Reference | int | ok | Žádná FK v DDL |
| Datum zařazení | date_insertion | Datum a čas | datetimeoffset | ok | -- |
| Datum vyřazení | date_rejection | Datum a čas | datetimeoffset | ok | -- |
| Způsob vyhodnocení realizace | realization_type | Enumerace | int (FK) | ok | FK -> vehicle_realizaton_type |
| Je aktivní | active | Boolean | bit | ok | -- |
| Datum vytvoření | created_at | Datum a čas | datetimeoffset | ok | -- |
| Datum poslední změny | updated_at | Datum a čas | datetimeoffset | ok | -- |
| Typ dopravy | vehicle__transport_type | Reference | M:N tabulka | ok | Řešeno vazební tabulkou |

### Identifikované delty

- **V DS, chybí v DDL:**
  - (nic -- vše pokryto)

- **V DDL, chybí v DS:**
  - `identification` (nvarchar(255)) -- další identifikační pole
  - `winyx_id` (int) -- ID starého systému WinyX
  - `device_type_id` (int) -- typ zařízení
  - `color`, `color_inherited` -- barva a příznak dědičnosti barvy
  - `cost_center_code`, `cost_center_id` -- středisko nákladů
  - `group_id` (int) -- ID skupiny
  - `driver_id` (nvarchar(10)) -- ID řidiče
  - `has_gps` (bit) -- má GPS?
  - `vin` (nvarchar(255)) -- VIN číslo
  - `sort_id` (int) -- řadicí klíč

- **Typové nesrovnalosti:**
  - `flww_id`: DS Objekt uvádí Řetězec(50), DDL má int
  - `name`: DS = Řetězec(50), DDL = nvarchar(100) -- DDL umožňuje delší název

---

## 6. Likvidační místo

**Priorita: Střední** | Vazba na skupinu odpadů

### Atributový profil z datového slovníku (DS-RP)

| # | Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis |
|---|---------|-----------|--------|----------|-----------|-------|
| 1 | Externí identifikátor | Řetězec | 50 | -- | Ne | ID v externím systému |
| 2 | Název | Řetězec | 255/150 | -- | Ne/Ano | Název LM |
| 3 | Adresa | Reference | -- | Adresa | Ne/Ano | Adresa LM |
| 4 | Souřadnice | Point | -- | -- | Ne | GPS souřadnice |
| 5 | Zeměpisná šířka | Desetinné číslo | -- | -- | Ne | Latitude |
| 6 | Zeměpisná délka | Desetinné číslo | -- | -- | Ne | Longitude |
| 7 | Kontakt | Řetězec | 255 | -- | Ne | Kontaktní údaje |
| 8 | Poznámka | Řetězec | 255 | -- | Ne | Poznámka |
| 9 | Provozovna | Reference | -- | Provozovna | Ne/Ano | Společnost nad LM |
| 10 | Stav geokódování | Enumerace | -- | Stav geokódování | Ne/Ano | Stav geokódování |
| 11 | Změna adresy | Boolean | -- | -- | Ano | Příznak změny adresy (neimplementováno) |
| 12 | Druhy odpadu | Reference [] | -- | Druh odpadu | Ano | Kolekce podporovaných druhů odpadu |
| 13 | Vlastní LM | Boolean | -- | -- | Ne/Ano | Majetek organizace? |
| 14 | Kód nakládání | Řetězec | 10 | -- | Ano | Co se s odpadem dále děje |
| 15 | Je aktivní | Boolean | -- | -- | Ano | Systémový atribut |
| 16 | Datum vytvoření | Datum a čas | -- | -- | Ano | Systémový atribut |
| 17 | Datum poslední změny | Datum a čas | -- | -- | Ano | Systémový atribut |

### Odpovídající tabulka v DDL (DDL-RP)

**Table name:** `RoadPlan_MPGSK.dbo.liquidation_site`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | -- |
| name | nvarchar(255) | NULL | -- | -- |
| address_id | int | NULL | FK -> address(id) | -- |
| lat | float | NULL | -- | -- |
| lng | float | NULL | -- | -- |
| contact | nvarchar(255) | NULL | -- | -- |
| note | nvarchar(255) | NULL | -- | -- |
| organization_unit_id | int | NULL | FK -> organization_unit(id) | -- |
| active | bit | NOT NULL | -- | -- |
| winyx_id_old | int | NULL | -- | -- |
| owned_liquidation_site | bit | NULL | -- | -- |
| geocoding_state | int | NULL | -- | -- |
| winyx_id | nvarchar(50) | NULL | -- | -- |
| created_at | datetimeoffset | NOT NULL | -- | -- |
| updated_at | datetimeoffset | NOT NULL | -- | -- |
| external_id | nvarchar(255) | NULL | -- | -- |

### Mapování DS-RP <-> DDL-RP

| Atribut (DS) | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|----------|-----------|-------|----------|
| Externí identifikátor | external_id | Řetězec(50) | nvarchar(255) | ok | DDL: delší rozsah |
| Název | name | Řetězec(255) | nvarchar(255) | ok | -- |
| Adresa | address_id | Reference | int (FK) | ok | -- |
| Zeměpisná šířka | lat | Desetinné číslo | float | ok | -- |
| Zeměpisná délka | lng | Desetinné číslo | float | ok | -- |
| Kontakt | contact | Řetězec(255) | nvarchar(255) | ok | -- |
| Poznámka | note | Řetězec(255) | nvarchar(255) | ok | -- |
| Provozovna | organization_unit_id | Reference | int (FK) | ok | -- |
| Stav geokódování | geocoding_state | Enumerace | int | ok | Žádná FK v DDL |
| Vlastní LM | owned_liquidation_site | Boolean | bit | ok | -- |
| Je aktivní | active | Boolean | bit | ok | -- |
| Datum vytvoření | created_at | Datum a čas | datetimeoffset | ok | -- |
| Datum poslední změny | updated_at | Datum a čas | datetimeoffset | ok | -- |
| Druhy odpadu | -- | Reference [] | -- | ok | Řešeno tabulkou liquidation_site_accepted_garbage_type |

### Identifikované delty

- **V DS, chybí v DDL:**
  - `Změna adresy` -- neimplementováno ani v DS
  - `Kód nakládání` -- v DDL je `waste_code` pouze v TMP/BCKUP tabulce; na úrovni `liquidation_site` není, ale je v `liquidation_site_accepted_garbage_type.loading_code`

- **V DDL, chybí v DS:**
  - `winyx_id_old` (int) -- staré ID WinyX
  - `winyx_id` (nvarchar(50)) -- ID WinyX (nový formát)

- **Typové nesrovnalosti:** (žádné)

---

## 7. Druh odpadu likvidačního místa

**Priorita: Střední** | Aktuální vazby LM <-> odpad

### Atributový profil z datového slovníku (DS-RP)

| # | Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis | Persist. ref. |
|---|---------|-----------|--------|----------|-----------|-------|---------------|
| 1 | Externí identifikátor | Řetězec | 50 | -- | Ne | ID v externím systému | -- |
| 2 | Likvidační místo | Reference | -- | Likvidační místo | Ano | LM, pro které je druh odpadu uveden | Ne |
| 3 | Druh odpadu | Reference | -- | Druh odpadu | Ano | Druh odpadu odebíraný na LM | Ne |
| 4 | Kód nakládání | Řetězec | 15 | -- | Ano | Kód způsobu nakládání | -- |
| 5 | Provozovna | Reference | -- | Provozovna | Ano | Provozovna, pro kterou je druh odpadu dostupný | Ne |
| 6 | Je aktivní | Boolean | -- | -- | Ano | Systémový atribut | -- |
| 7 | Datum vytvoření | Datum a čas | -- | -- | Ano | Systémový atribut | -- |
| 8 | Datum poslední změny | Datum a čas | -- | -- | Ano | Systémový atribut | -- |

### Odpovídající tabulka v DDL (DDL-RP)

**Table name:** `RoadPlan_MPGSK.dbo.liquidation_site_accepted_garbage_type`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | -- |
| liquidation_site_id | int | NULL | FK -> liquidation_site(id) | -- |
| garbage_type_id | int | NULL | FK -> garbage_type(id) | -- |
| winyx_id | int | NULL | -- | -- |
| active | bit | NOT NULL | -- | -- |
| loading_code | nvarchar(15) | NULL | -- | -- |
| organization_unit_id | int | NULL | FK -> organization_unit(id) | -- |
| external_id | nvarchar(255) | NULL | -- | -- |

### Mapování DS-RP <-> DDL-RP

| Atribut (DS) | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|----------|-----------|-------|----------|
| Externí identifikátor | external_id | Řetězec(50) | nvarchar(255) | ok | DDL: delší |
| Likvidační místo | liquidation_site_id | Reference | int (FK) | ok | -- |
| Druh odpadu | garbage_type_id | Reference | int (FK) | ok | -- |
| Kód nakládání | loading_code | Řetězec(15) | nvarchar(15) | ok | -- |
| Provozovna | organization_unit_id | Reference | int (FK) | ok | -- |
| Je aktivní | active | Boolean | bit | ok | -- |

### Identifikované delty

- **V DS, chybí v DDL:**
  - `Datum vytvoření` -- v DDL tabulka nemá `created_at`
  - `Datum poslední změny` -- v DDL tabulka nemá `updated_at`

- **V DDL, chybí v DS:**
  - `winyx_id` (int) -- ID starého systému WinyX

- **Typové nesrovnalosti:** (žádné)

---

## 8. Typ dopravy

**Priorita: Střední** | Nový typ pro CS

### Atributový profil z datového slovníku (DS-RP)

| # | Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis | Poznámka |
|---|---------|-----------|--------|----------|-----------|-------|----------|
| 1 | Název | Řetězec | 50 | -- | Ano | Název typu dopravy | Na úrovni DB doplněn pomocný sloupec pro filtrování |
| 2 | Je k dispozici | Boolean | -- | -- | Ano | Dostupný v aplikaci? | Nastavení se liší dle zákazníka |
| 3 | Technická specifikace | Řetězec | MAX | -- | Ano | Technické parametry (JSON) | Struktura dána externím dodavatelem |

### Hodnoty typu dopravy (DS-RP)

| Název | Je k dispozici (MPG SK) | Dostupné typy úkonu (MPG SK) |
|-------|-------------------------|------------------------------|
| Hákový nakladač | TRUE | Přistavení, Vývoz, Výměna, Odvoz, Přeprava, Manipulace |
| Ramenový nakladač | TRUE | Přistavení, Vývoz, Výměna, Odvoz, Přeprava, Manipulace |
| Lisovací vozidlo | TRUE | Sběr |
| Valník | TRUE | Sběr, Rozvoz, Přeprava |
| Komunální vozidlo -- pilot | TRUE | Sběr |
| Cisterna | FALSE | -- |

### Odpovídající tabulka v DDL (DDL-RP)

**Table name:** `RoadPlan_MPGSK.dbo.transport_type`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int | NOT NULL | PK | -- |
| name | nvarchar(50) | NOT NULL | -- | -- |
| name_search | varchar(500) | NULL | -- | -- |
| is_available | bit | NOT NULL | -- | -- |

### Mapování DS-RP <-> DDL-RP

| Atribut (DS) | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|----------|-----------|-------|----------|
| Název | name | Řetězec(50) | nvarchar(50) | ok | -- |
| Je k dispozici | is_available | Boolean | bit | ok | -- |
| (pomocný sloupec) | name_search | -- | varchar(500) | -- | Pomocný sloupec pro filtrování (zmíněn v DS poznámce) |

### Identifikované delty

- **V DS, chybí v DDL:**
  - `Technická specifikace` -- v DDL na tabulce `transport_type` CHYBI! V DS je Řetězec(MAX) s JSON specifikací. Pravděpodobně uložena jinde (např. v konfiguraci aplikace) nebo dosud neimplementována v DB.

- **V DDL, chybí v DS:**
  - `name_search` (varchar(500)) -- pomocný sloupec pro filtrování, v DS zmíněn v poznámce

- **Typové nesrovnalosti:** (žádné)

> **DULEZITE pro CS:** Pokud se bude přidávat nový typ dopravy pro Cyklické svozy, je nutné vyřešit, kde bude uložena technická specifikace (definice generovaných lokací, dostupné typy úkonu). V DDL zatím není odpovídající sloupec.

---

## 9. Typ položky denního výkonu

**Priorita: Vysoká** | Nový kód pro okruh dne

### Hodnoty z datového slovníku (DS-RP)

| Položka | Poznámka |
|---------|----------|
| Objednaná služba | Položka DV určena pro Objednanou službu |
| Lokace objednané služby | Položka DV určena pro Lokaci OS (cesta na LM, bod průjezdu...) |
| Časový interval | Položka DV určena pro přestávku |
| Rozdělení | Speciální případ Lokace OS pro rozdělení směny |

### Odpovídající tabulka v DDL (DDL-RP)

**Table name:** `RoadPlan_MPGSK.dbo.day_performance_item_type`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | -- |
| code | nvarchar(50) | NOT NULL | UNIQUE | -- |

### Mapování DS-RP <-> DDL-RP

| Hodnota (DS) | Kód (DDL) | Poznámka |
|-------------|-----------|----------|
| Objednaná služba | (neznámý) | Mapování DS hodnoty -> DDL code nutno ověřit v datech |
| Lokace objednané služby | (neznámý) | Mapování nutno ověřit v datech |
| Časový interval | (neznámý) | Mapování nutno ověřit v datech |
| Rozdělení | (neznámý) | Mapování nutno ověřit v datech |

### Identifikované delty

- **V DS, chybí v DDL:**
  - DDL definuje pouze strukturu (id + code), ne konkrétní hodnoty. Skutečné kódy nutno ověřit v produkčních datech.

- **Pro CS:** Bude nutné přidat nový kód pro "okruh dne" (circuit / round), který v současném DS ani DDL neexistuje.

---

## 10. Stav objednané služby

**Priorita: Střední** | Stavový automat pro CS OS

### Hodnoty z datového slovníku (DS-RP)

| Stav | Popis |
|------|-------|
| K naplánování | OS vytvořena, připravena k naplánování |
| Naplánovaná | OS vložena na DV, zatím nerealizována |
| Ke kontrole | OS na DV, čeká na kontrolu realizace |
| Zrušená | OS zrušena, není na DV |
| Realizovaná | OS na DV, potvrzena jako realizovaná |
| Nezrealizovaná | OS na DV, nebyla realizována -- nutno přeplánovat |

### Odpovídající tabulka v DDL (DDL-RP)

**Table name:** `RoadPlan_MPGSK.dbo.ordered_service_status`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int | NOT NULL | PK | -- |
| name | nvarchar(25) | NULL | -- | -- |

### Mapování DS-RP <-> DDL-RP

| Stav (DS) | Předpokládané ID (DDL) | Poznámka |
|-----------|----------------------|----------|
| K naplánování | (nutno ověřit) | -- |
| Naplánovaná | (nutno ověřit) | -- |
| Ke kontrole | (nutno ověřit) | -- |
| Zrušená | (nutno ověřit) | -- |
| Realizovaná | (nutno ověřit) | -- |
| Nezrealizovaná | (nutno ověřit) | -- |

### Identifikované delty

- Struktura OK -- DDL obsahuje id + name, což odpovídá DS enumeraci.
- **Pro CS:** Nutno ověřit, zda současné stavy pokrývají workflow Cyklických svozů, nebo bude potřeba přidat nové stavy.

---

## 11. Místo realizace

**Priorita: Střední** | Alternativa k lokaci z PP (execution_site)

### Atributový profil z datového slovníku (DS-RP)

| # | Atribut | Datový typ | Rozsah | Asociace | Povinnost | Popis |
|---|---------|-----------|--------|----------|-----------|-------|
| 1 | Externí identifikátor | Řetězec | 50 | -- | Ne | ID v externím systému |
| 2 | Zákazník | Reference | -- | Zákazník | Ano | Zákazník, ke kterému MR patří |
| 3 | Název | Řetězec | 150/255 | -- | Ne/Ano | Název MR |
| 4 | Adresa | Reference | -- | Adresa | Ano | Adresa MR |
| 5 | Souřadnice | Point | -- | -- | Ne | GPS souřadnice |
| 6 | Zeměpisná šířka | Desetinné číslo | -- | -- | Ne | Latitude |
| 7 | Zeměpisná délka | Desetinné číslo | -- | -- | Ne | Longitude |
| 8 | Kontakt | Řetězec | 255 | -- | Ne | Kontaktní údaje |
| 9 | Poznámka | Řetězec | 255 | -- | Ne | Poznámka |
| 10 | Provozovna | Reference | -- | Provozovna | Ne | Provozovna pro MR |
| 11 | Výchozí | Boolean | -- | -- | Ne | Výchozí MR? |
| 12 | Stav geokódování | Enumerace | -- | Stav geokódování | Ne/Ano | Stav geokódování |
| 13 | Změna adresy | Boolean | -- | -- | Ano | Příznak změny adresy (neimplementováno) |
| 14 | Je aktivní | Boolean | -- | -- | Ano | Systémový atribut |
| 15 | Datum vytvoření | Datum a čas | -- | -- | Ano | Systémový atribut |
| 16 | Datum poslední změny | Datum a čas | -- | -- | Ano | Systémový atribut |

### Odpovídající tabulka v DDL (DDL-RP)

**Table name:** `RoadPlan_MPGSK.dbo.execution_site`

| Sloupec | SQL typ | Nullable | FK | Default |
|---------|---------|----------|----|---------|
| id | int IDENTITY(1,1) | NOT NULL | PK | -- |
| address_id | int | NOT NULL | FK -> address(id) | -- |
| lat | float | NULL | -- | -- |
| lng | float | NULL | -- | -- |
| contact | nvarchar(255) | NULL | -- | -- |
| note | nvarchar(255) | NULL | -- | -- |
| name | nvarchar(255) | NULL | -- | -- |
| customer_id | int | NOT NULL | FK -> customer(id) | -- |
| active | bit | NOT NULL | -- | -- |
| winyx_id_old | int | NULL | -- | -- |
| geocoding_state | int | NULL | -- | -- |
| winyx_id | nvarchar(50) | NULL | -- | -- |
| created_at | datetimeoffset | NOT NULL | -- | -- |
| updated_at | datetimeoffset | NOT NULL | -- | -- |
| organization_unit_id | int | NULL | -- | -- |
| external_id | nvarchar(255) | NULL | -- | -- |

### Mapování DS-RP <-> DDL-RP

| Atribut (DS) | Sloupec (DDL) | Typ (DS) | Typ (DDL) | Shoda | Poznámka |
|-------------|---------------|----------|-----------|-------|----------|
| Externí identifikátor | external_id | Řetězec(50) | nvarchar(255) | ok | DDL: delší |
| Zákazník | customer_id | Reference | int (FK) | ok | -- |
| Název | name | Řetězec(150/255) | nvarchar(255) | ok | -- |
| Adresa | address_id | Reference | int (FK) | ok | -- |
| Zeměpisná šířka | lat | Desetinné číslo | float | ok | -- |
| Zeměpisná délka | lng | Desetinné číslo | float | ok | -- |
| Kontakt | contact | Řetězec(255) | nvarchar(255) | ok | -- |
| Poznámka | note | Řetězec(255) | nvarchar(255) | ok | -- |
| Provozovna | organization_unit_id | Reference | int | ok | Pozor: v DDL CHYBI FK constraint! |
| Stav geokódování | geocoding_state | Enumerace | int | ok | Žádná FK v DDL |
| Je aktivní | active | Boolean | bit | ok | -- |
| Datum vytvoření | created_at | Datum a čas | datetimeoffset | ok | -- |
| Datum poslední změny | updated_at | Datum a čas | datetimeoffset | ok | -- |

### Identifikované delty

- **V DS, chybí v DDL:**
  - `Výchozí` (Boolean) -- příznak výchozího MR; v DDL je řešen jako `df_execution_site_id` na tabulce `customer`
  - `Změna adresy` -- neimplementováno ani v DS

- **V DDL, chybí v DS:**
  - `winyx_id_old` (int) -- staré ID WinyX
  - `winyx_id` (nvarchar(50)) -- ID WinyX (nový formát)

- **Typové nesrovnalosti:** (žádné)

> **Poznámka:** Existuje i tabulka `execution_site_erp_mapping` pro mapování na ERP systém, která má vlastní id, external_id, name, address_id, lat, lng.

---

## Souhrnná tabulka pokrytí

| # | Entita DS-RP | Tabulka DDL-RP | Priorita | Stav pokrytí | Klíčové delty |
|---|-------------|----------------|----------|-------------|---------------|
| 1 | Objednaná služba | ordered_service | Kritická | Vysoké | DDL navíc: container_id, is_according_to_plan, confirmed_by; DS navíc: Položka DV (opačná FK) |
| 2 | Lokace objednané služby | ordered_service_location | Kritická | Vysoké | Typové: distance/time float vs int; liquidation_garbage_types = JSON vs reference |
| 3 | Denní výkon | day_performance | Vysoká | Střední | DS navíc: Délka provozní doby, Časové využití, Podíl km s přívěsem, Počet OS (v DDL buď chybí nebo na jiné tabulce) |
| 4 | Položka denního výkonu | day_performance_item | Vysoká | Vysoké | DDL navíc: duration, note |
| 5 | Vozidlo | vehicle | Vysoká | Vysoké | DDL výrazně bohatší (winyx, driver, GPS, barva, VIN atd.) |
| 6 | Likvidační místo | liquidation_site | Střední | Vysoké | DS Kód nakládání -> v podřízené tabulce |
| 7 | Druh odpadu lik. místa | liquidation_site_accepted_garbage_type | Střední | Vysoké | DDL: chybí created_at, updated_at |
| 8 | Typ dopravy | transport_type | Střední | Střední | DS navíc: Technická specifikace (JSON) -- v DDL CHYBI! |
| 9 | Typ položky DV | day_performance_item_type | Vysoká | OK | Pouze id+code; hodnoty nutno ověřit; CS přidá nový kód |
| 10 | Stav objednané služby | ordered_service_status | Střední | OK | Pouze id+name; hodnoty nutno ověřit |
| 11 | Místo realizace | execution_site | Střední | Vysoké | DS "Výchozí" řešen přes customer.df_execution_site_id |

---

## Klíčová zjištění pro CS projekt

1. **Technická specifikace typu dopravy** -- v DDL `transport_type` CHYBI sloupec pro JSON specifikaci, který DS popisuje. Pro CS bude nutné buď rozšířit tabulku, nebo ukládat specifikaci mimo DB.

2. **Denní výkon -- chybějící sloupce** -- několik atributů z DS (Délka provozní doby, Časové využití, Podíl km s přívěsem, Počet OS) není v DDL tabulce `day_performance`. Některé jsou pravděpodobně na úrovni `day_performance_realization` nebo odvozeny runtime.

3. **Typ položky DV** -- pro okruhy dne (CS) bude nutné přidat nový kód do číselníku `day_performance_item_type`.

4. **Stav objednané služby** -- současných 6 stavů může být dostatečných pro CS, ale je nutné ověřit workflow.

5. **Likvidační místo -- druhy odpadu** -- vazba je implementována přes vazební tabulku `liquidation_site_accepted_garbage_type` s kódem nakládání na úrovni vazby (nikoliv na úrovni LM, jak DS částečně naznačuje).

6. **Lokace objednané služby -- druhy odpadu** -- v DDL uloženy jako JSON (nvarchar(MAX)), nikoliv jako reference na entity. Nutno zohlednit při návrhu integrace.
