# Inventář entit RP pro Cyklické svozy

> **Účel:** Přehled existujících entit v DS a DDL relevantních pro CS. Čistý inventář bez hodnocení.
> **Zdroje:** DS (datovy-78681044.html, 88 entit), DDL (DDL_RP_DB.txt)
> **Vytvořeno:** 2026-02-23

---

## Obsah

1. [Objednaná služba](#1-objednaná-služba)
2. [Lokace objednané služby](#2-lokace-objednané-služby)
3. [Denní výkon](#3-denní-výkon)
4. [Položka denního výkonu](#4-položka-denního-výkonu)
5. [Realizace denního výkonu](#5-realizace-denního-výkonu)
6. [Objednávka](#6-objednávka)
7. [Položka objednávky](#7-položka-objednávky)
8. [Objednaná nádoba](#8-objednaná-nádoba)
9. [Objednaný úkon](#9-objednaný-úkon)
10. [Vývoz na Likvidační místo](#10-vývoz-na-likvidační-místo)
11. [Typ nádoby](#11-typ-nádoby)
12. [Druh odpadu](#12-druh-odpadu)
13. [Likvidační místo](#13-likvidační-místo)
14. [Druh odpadu likvidačního místa](#14-druh-odpadu-likvidačního-místa)
15. [Objekt (Vozidlo v DS)](#15-objekt)
16. [Typ dopravy objektu](#16-typ-dopravy-objektu)
17. [Přívěs](#17-přívěs)
18. [Typ dopravy](#18-typ-dopravy)
19. [Provozovna](#19-provozovna)
20. [Informace o provozovně](#20-informace-o-provozovně)
21. [Místo realizace](#21-místo-realizace)
22. [Stanoviště](#22-stanoviště)
23. [Přiřazení objektu ke stanovišti](#23-přiřazení-objektu-ke-stanovišti)
24. [Nastavení objektu pro denní výkon](#24-nastavení-objektu-pro-denní-výkon)
25. [Zákazník](#25-zákazník)
26. [Přívěs denního výkonu](#26-přívěs-denního-výkonu)
27. [Interval omezení denního výkonu](#27-interval-omezení-denního-výkonu)
28. [Průběh editace denního výkonu](#28-průběh-editace-denního-výkonu)
29. [Obsluha lokace objednané služby](#29-obsluha-lokace-objednané-služby)
30. [FOB informace o realizaci lokace OS](#30-fob-informace-o-realizaci-lokace-os)
31. [Množstevní jednotka](#31-množstevní-jednotka)
32. [Kód nakládání](#32-kód-nakládání)
33. [Enumerace: Stav objednané služby](#33-stav-objednané-služby)
34. [Enumerace: Určení objednané služby](#34-určení-objednané-služby)
35. [Enumerace: Způsob přeplánování OS](#35-způsob-přeplánování-os)
36. [Enumerace: Způsob rozdělení OS](#36-způsob-rozdělení-os)
37. [Enumerace: Stav denního výkonu](#37-stav-denního-výkonu)
38. [Enumerace: Typ položky denního výkonu](#38-typ-položky-denního-výkonu)
39. [Enumerace: Typ lokace objednané služby](#39-typ-lokace-objednané-služby)
40. [Enumerace: Akce v lokaci objednané služby](#40-akce-v-lokaci-objednané-služby)
41. [Enumerace: Způsob odbavení lokace OS](#41-způsob-odbavení-lokace-os)
42. [Entity neexistující v DS ani DDL (nové pro CS)](#42-entity-neexistující-v-ds-ani-ddl)

---

## 1. Objednaná služba
> Vzniká na základě Objednaného úkonu. Pokud se jedná o opakující se Objednaný úkon, může z něj vzniknout více Objednaných služeb. Objednané služby jsou Dispečerem plánovány do Denních výkonů (tras).

**DS:** Objednaná služba
**DDL:** `ordered_service`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Stav | Enumerace | A | Stav, ve kterém se právě nachází OS (viz Stav objednané služby). |
| Objednaný úkon | Reference | A | Objednaný úkon, ze kterého OS vznikla. |
| Konkrétní objednaná nádoba | Celé kladné číslo (1-255) | A | Označení konkrétní nádoby, pro kterou je OS určena. |
| Datum realizace od | Datum | A | Předpokládané časové okno realizace OS – datum od. |
| Datum realizace do | Datum | A | Předpokládané časové okno realizace OS – datum do. |
| Čas realizace od | Čas | A | Předpokládané časové okno realizace OS – čas od. |
| Čas realizace do | Čas | A | Předpokládané časové okno realizace OS – čas do. |
| Poznámka | Řetězec (255) | N | Poznámka k OS. |
| Datum vytvoření | Datum a čas | A | Datum a čas, kdy byla OS vytvořena. |
| Vytvořil | Reference → Uživatel | P | Uživatel, který vytvořil OS. |
| Vytvořeno uživatelem | Boolean | A | TRUE = vytvořena uživatelem, FALSE = vygenerována systémem. |
| Vytvořena na výzvu | Boolean | A | TRUE = vygenerována na výzvu objednavatele. |
| Pořadí v rámci objednávky | Číslo (kladné, celé) | A | Pořadové číslo OS jedinečné na úrovni Objednávky. |
| Identifikátor jízdy | Řetězec (255) | Ne | Identifikátor záznamu na straně ERP systému. |
| Evidenční číslo výchozího vozidla | Řetězec (255) | Ne | Evidenční číslo vozidla z ERP. |
| Výchozí datum realizace | Datum | Ne | Původní datum realizace z ERP. |
| Položka denního výkonu | Reference → Položka denního výkonu | N | Položka DV, v rámci které je OS naplánována. |
| Lokace objednané služby | Reference [] → Lokace objednané služby | P | Posloupnost kroků v rámci OS. |
| Pořadí v rámci objednané služby | Číslo (kladné, celé) | A | Pořadové číslo části v rámci rozdělené OS. |
| Následující objednaná služba | Reference → Objednaná služba | N | OS vzniklá rozdělením – následující část. |
| Způsob rozdělení následující OS | Enumerace | N | Způsob rozdělení vůči následující části. |
| Předchozí objednaná služba | Reference → Objednaná služba | N | OS vzniklá rozdělením – předchozí část. |
| Způsob rozdělení předchozí OS | Enumerace | N | Způsob rozdělení vůči předchozí části. |
| Realizace denního výkonu | Reference → Realizace denního výkonu | P | Realizace DV, v rámci které byla OS realizována. |
| Pořadí v realizaci denního výkonu | Celé kladné číslo | N | Pořadí zobrazení v kontrole realizace DV. |
| Realizována | Boolean | N | TRUE = OS byla skutečně realizována. |
| Způsob přeplánování | Enumerace | N | Způsob dalšího řešení nezrealizované OS. |
| Přeplánováno do objednané služby | Reference → Objednaná služba | N | OS vytvořená přeplánováním z této instance. |
| Přeplánováno z objednané služby | Reference → Objednaná služba | N | Původní OS, ze které tato instance vznikla přeplánováním. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| status | int | NOT NULL | → ordered_service_status.id |
| container_operation_id | int | NOT NULL | → order_item_container_operation.id |
| container_id | int | NOT NULL | → order_item_container.id |
| container_number | int | NOT NULL | – |
| date_from | date | NULL | – |
| date_to | date | NULL | – |
| note | nvarchar(255) | NULL | – |
| time_from | varchar(150) | NULL | – |
| time_to | varchar(150) | NULL | – |
| parent_ordered_service_id | int | NULL | → ordered_service.id |
| child_ordered_service_id | int | NULL | → ordered_service.id |
| temporary_storage_ordered_service_id | int | NULL | → ordered_service.id |
| type | nvarchar(200) | NULL | → ordered_service_type_code.type |
| day_performance_realization_id | int | NULL | → day_performance_realization.id |
| is_realized | bit | NOT NULL | – |
| realization_order | int | NULL | – |
| is_according_to_plan | bit | NOT NULL | – |
| confirmed_by | int | NULL | → user.id |
| rescheduled_to_ordered_service_id | int | NULL | → ordered_service.id |
| rescheduled_from_ordered_service_id | int | NULL | → ordered_service.id |
| reschedule_method | nvarchar(200) | NULL | → ordered_service_reschedule_method_code.type |
| confirmation_datetime | datetimeoffset | NULL | – |
| created_at | datetimeoffset | NOT NULL | – |
| updated_at | datetimeoffset | NOT NULL | – |
| is_created_by_user | bit | NOT NULL | – |
| created_by | int | NULL | → user.id |
| parent_ordered_service_split_type | nvarchar(MAX) | NULL | – |
| child_ordered_service_split_type | nvarchar(MAX) | NULL | – |
| split_order | int | NULL | – |
| ordered_service_chain_ids | nvarchar(MAX) | NULL | – |
| on_demand | bit | NOT NULL | – |
| ordering_within_order | int | NULL | – |
| ride_identifier | nvarchar(255) | NULL | – |
| vehicle_evidence_number | nvarchar(255) | NULL | – |
| default_realization_date | date | NULL | – |

**Asociace z DS:**
- Objednaný úkon (N:1)
- Stav objednané služby (enumerace)
- Uživatel – Vytvořil (N:1)
- Položka denního výkonu (N:1)
- Lokace objednané služby (1:N)
- Objednaná služba – Následující (1:1, self-ref)
- Objednaná služba – Předchozí (1:1, self-ref)
- Realizace denního výkonu (N:1)
- Objednaná služba – Přeplánováno do (1:1, self-ref)
- Objednaná služba – Přeplánováno z (1:1, self-ref)
- Způsob přeplánování objednané služby (enumerace)
- Způsob rozdělení objednané služby (enumerace)


**Reference v tech. projektu:**
- slovnik-78680658 (slovník pojmů)
- stavove-78680652 (stavové diagramy)
- 101ucxx (případy užití — objednané služby)
- 101uixx (UI specifikace — objednané služby)
- 201ucxx (plánování — objednané služby)
- 200ucxx (plánování — denní výkony; vkládání/odebírání OS na DV)
- 206ucxx (obrazovka plánování DV; panely OS)
- 400ucxx (kontrola realizace DV; přeplánování OS)
- 502uc07–09 (založit/upravit/odebrat obsluhu lokace OS)
- 602uc14 (přijmout FOB informace o realizaci lokace OS)
- 707ucxx (asynchronní export záznamů)

---

## 2. Lokace objednané služby
> Dílčí krok v rámci Objednané služby — popisuje konkrétní místo (Místo realizace, Likvidační místo, Provozovna apod.) a akci, která se v něm provede (Vyzvednout, Umístit, Vyprázdnit, Navštívit).

**DS:** Lokace objednané služby
**DDL:** `ordered_service_location`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Objednaná služba | Reference → Objednaná služba | N | Reference na odpovídající OS. |
| Pořadí | Kladné celé číslo | A | Pořadí obsluhy lokací. |
| Akce | Enumerace → Akce v lokaci OS | A | Provedená akce v lokaci OS. |
| Typ lokace | Enumerace → Typ lokace OS | A | Typ lokace OS. |
| Adresa | Reference → Adresa | P | Adresa lokace OS. |
| Souřadnice | Point | P | Souřadnice lokace OS. |
| Manuální posun souřadnic | Boolean | A | TRUE = došlo k manuálnímu posunutí souřadnic. |
| Místo realizace | Reference → Místo realizace | N | Reference pro účely aktualizace lokace. |
| Likvidační místo | Reference → Likvidační místo | P | Povinná pro Typ lokace Likvidační místo. |
| Kód nakládání | Řetězec (15) | P | Kód způsobu nakládání s odpadem. |
| Provozovna | Reference → Provozovna | P | Povinná pro Typ lokace Provozovna. |
| Doba manipulace | Celé kladné číslo | P | Doba manipulace v místě lokace. |
| Doba trvání | Celé kladné číslo | P | Povinná pro Typ lokace Časový interval. |
| Provést | Boolean | A | TRUE = lokace bude obsloužena. |
| Začátek časového okna | Čas | P | Časové okno realizace: čas od. |
| Konec časového okna | Čas | P | Časové okno realizace: čas do. |
| Splněno časové okno | Boolean | P | TRUE = lokace obsloužena v rámci okna. |
| Vzdálenost do další lokace | Kladné celé číslo | N | Vzdálenost do další lokace. |
| Doba jízdy do další lokace | Kladné celé číslo | N | Doba jízdy do další lokace. |
| Trasa do další lokace | LineString | N | Trasa do další lokace. |
| Poznámka | Řetězec (255) | N | Poznámka k lokaci. |
| Monitoring realizace | Boolean | N | TRUE = monitoring realizace lokace. |
| Obsluha lokace OS | Reference → Obsluha lokace OS | N | Odkaz na instanci Obsluha lokace OS. |
| Fakturace | Boolean | N | TRUE = vývoz na LM pod paušálem. |
| Druhy odpadu | Reference [] → Druh odpadu | N | Druhy odpadu pro vývoz na LM. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| order | int | NOT NULL | – |
| action | int | NOT NULL | → ordered_service_location_action.id |
| type | int | NOT NULL | → ordered_service_location_type.id |
| address_id | int | NULL | → address.id |
| lat | float | NULL | – |
| lng | float | NULL | – |
| liquidation_site_id | int | NULL | → liquidation_site.id |
| execute | bit | NULL | – |
| organization_unit_id | int | NULL | → organization_unit.id |
| valid | bit | NULL | – |
| next_location_distance | float | NULL | – |
| next_location_time | float | NULL | – |
| route_to_next_location | geometry | NULL | – |
| ordered_service_id | int | NULL | → ordered_service.id |
| note | nvarchar(255) | NULL | – |
| time_window_start | nvarchar(5) | NOT NULL | – |
| time_window_end | nvarchar(5) | NOT NULL | – |
| is_time_window_met | bit | NOT NULL | – |
| duration | int | NULL | – |
| is_generated | bit | NOT NULL | – |
| manipulation_time_in_minutes | int | NULL | – |
| is_monitored | bit | NOT NULL | – |
| execution_site_id | int | NULL | → execution_site.id |
| manual_address_change | bit | NOT NULL | – |
| has_flat_rate | bit | NULL | – |
| liquidation_garbage_types | nvarchar(MAX) | NULL | – |
| created_at | datetimeoffset | NOT NULL | – |
| updated_at | datetimeoffset | NOT NULL | – |
| loading_code | nvarchar(15) | NULL | – |

**Asociace z DS:**
- Objednaná služba (N:1)
- Akce v lokaci objednané služby (enumerace)
- Typ lokace objednané služby (enumerace)
- Adresa (N:1)
- Místo realizace (N:1)
- Likvidační místo (N:1)
- Provozovna (N:1)
- Obsluha lokace objednané služby (1:1)
- Druh odpadu (N:M)


**Reference v tech. projektu:**
- 201ucxx (plánování — objednané služby; editace lokací)
- 200ui05 (formulář lokací objednané služby)
- 206uc11 (panel lokace OS na obrazovce plánování)
- 400uc17 (změnit poznámku u obsluhy lokace OS)
- 502uc03 (založit FOB informace o realizaci lokace OS)
- 502uc07–09 (založit/upravit/odebrat obsluhu lokace OS)
- 602uc14 (přijmout informace o realizaci lokace OS)

---

## 3. Denní výkon
> Plán práce jednoho vozidla na jeden den. Obsahuje Položky (objednané služby k obsloužení) a definuje trasu, přívěsy, omezení a provozní dobu. Prochází stavy: Plánovaný, Schválený, Plněný, Ke kontrole, Uzavřený.

**DS:** Denní výkon
**DDL:** `day_performance`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Stav | Enumerace → Stav DV | A | Stav denního výkonu. |
| Provozovna | Reference → Provozovna | A | Provozovna, pro kterou je DV naplánován. |
| Datum realizace | Datum | A | Den realizace DV. |
| Typ dopravy | Reference → Typ dopravy | A | Typ dopravy DV. |
| Vozidlo | Reference → Vozidlo | N | Vozidlo přiřazené k DV. |
| Kód barvy | Řetězec (7) | N | Barva DV v aplikaci. |
| Použití přívěsu | Boolean | A | TRUE = v DV je použit přívěs. |
| Použité přívěsy | Reference [] → Přívěs | N | Přívěsy přiřazené k DV. |
| Intervaly omezení | Reference [] → Interval omezení DV | N | Časová omezení DV. |
| Začátek provozní doby | Čas | A | Čas začátku DV. |
| Délka provozní doby | Celé číslo | A | Doba trvání provozní doby DV. |
| Počáteční položka DV | Reference → Položka DV | A | Místo začátku DV. |
| Koncová položka DV | Reference → Položka DV | A | Místo konce DV. |
| Položky denního výkonu | Reference [] → Položka DV | N | Místa obsloužená v rámci DV. |
| Ujetá vzdálenost celkem | Celé číslo | P | Celková délka trasy. |
| Doba jízdy celkem | Celé číslo | P | Celková doba jízdy. |
| Způsob získání trajektorie | Enumerace | Ne | Způsob získání trajektorie DV. |
| Poznámka | Řetězec (255) | N | Poznámka k DV. |
| Časové využití | Desetinné číslo | P | Podíl odhadované doby jízdy a provozní doby. |
| Podíl kilometrů s přívěsem | Desetinné číslo | P | Podíl ujeté vzdálenosti s přívěsem. |
| Počet objednaných služeb | Celé číslo | A | Počet OS obsloužených v rámci DV. |
| Schválený denní výkon | Reference → Denní výkon (json) | N | Uložený původní stav schváleného DV. |
| Průběhy editace DV | Reference [] → Průběh editace DV | N | Uložené stavy DV při editaci. |
| Vytisknuto potvrzení | Boolean | A | TRUE = vytištěno potvrzení o výkonu OS. |
| Textová identifikace vozidla | Řetězec (50) | N | Textová identifikace Vozidla přiřazeného k DV. |
| Způsob vyhodnocení realizace | Enumerace | Ano | Způsob vyhodnocování realizace. |
| Vytvořen uživatelem | Boolean | A | TRUE = DV vytvořen uživatelem. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| status | int | NOT NULL | → day_performance_status.id |
| organization_unit_id | int | NOT NULL | → organization_unit.id |
| transport_type | int | NOT NULL | → transport_type.id |
| vehicle_id | int | NULL | → vehicle.id |
| realization_date | date | NOT NULL | – |
| start_time | nvarchar(5) | NOT NULL | – |
| start_day_performance_item_id | int | NULL | → day_performance_item.id |
| finish_day_performance_item_id | int | NULL | → day_performance_item.id |
| route_duration | int | NULL | – |
| route_distance | int | NULL | – |
| route_geom | geometry | NULL | – |
| note | nvarchar(255) | NULL | – |
| has_trailer | bit | NOT NULL | – |
| need_reroute | bit | NOT NULL | – |
| is_confirmation_printed | bit | NOT NULL | – |
| is_created_by_user | bit | NOT NULL | – |
| vehicle_identification | nvarchar(MAX) | NULL | – |
| created_at | datetimeoffset | NOT NULL | – |
| updated_at | datetimeoffset | NOT NULL | – |
| realization_type | int | NOT NULL | → vehicle_realizaton_type.id |
| color | nvarchar(50) | NULL | – |

**Asociace z DS:**
- Stav denního výkonu (enumerace)
- Provozovna (N:1)
- Typ dopravy (N:1)
- Vozidlo (N:1)
- Přívěs (N:M)
- Interval omezení denního výkonu (1:N)
- Položka denního výkonu (1:N, + 2x reference na počáteční a koncovou)
- Průběh editace denního výkonu (1:N)
- Způsob vyhodnocení realizace (enumerace)


**Reference v tech. projektu:**
- stavove-78680652 (stavové diagramy)
- 200ucxx (případy užití — plánování DV)
- 200uixx (UI specifikace — plánování DV)
- 202ucxx (plánování — obecné; generování DV)
- 203ucxx (plánování — časové využití vozidel)
- 204ucxx (tiskové výstupy pro DV)
- 205ucxx (generování tras DV)
- 206ucxx (obrazovka plánování DV)
- 207ucxx (zpracování dat o DV)
- 300ucxx (monitoring realizace DV)
- 300uixx (UI specifikace — monitoring realizace DV)
- 400ucxx (kontrola realizace DV)
- 400uixx (UI specifikace — kontrola realizace DV)
- 604uc11 (FLWW2: získat nastavení objektů pro DV)
- 707ucxx (asynchronní export záznamů)

---

## 4. Položka denního výkonu
> Jednotlivý bod v itineráři Denního výkonu — určuje pořadí obsluhy a odkazuje na konkrétní Objednanou službu nebo Lokaci OS. Typ položky rozlišuje, zda jde o OS, lokaci, přestávku nebo rozdělení směny.

**DS:** Položka denního výkonu
**DDL:** `day_performance_item`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Pořadí | Kladné celé číslo | A | Pořadí obsluhy položek DV. |
| Typ položky | Enumerace → Typ položky DV | A | Typ konkrétní položky DV. |
| Denní výkon | Reference → Denní výkon | A | DV, ve kterém je položka obsažena. |
| Přívěs | Boolean | A | TRUE = v rámci položky je použit přívěs. |
| Objednaná služba | Reference → Objednaná služba | P | OS obsloužená v rámci položky. |
| Lokace objednané služby | Reference → Lokace OS | P | Lokace OS obsloužená v rámci položky. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| order | int | NOT NULL | – |
| day_performance_id | int | NOT NULL | → day_performance.id |
| duration | int | NULL | – |
| has_trailer | bit | NOT NULL | – |
| ordered_service_id | int | NULL | → ordered_service.id |
| ordered_service_location_id | int | NULL | → ordered_service_location.id |
| day_performance_item_type | int | NOT NULL | → day_performance_item_type.id |
| note | nvarchar(255) | NULL | – |

**Asociace z DS:**
- Denní výkon (N:1)
- Typ položky denního výkonu (enumerace)
- Objednaná služba (N:1)
- Lokace objednané služby (N:1)


**Reference v tech. projektu:**
- 200ucxx (plánování — denní výkony; vkládání/odebírání položek)
- 200uc34 (rozdělit položku denního výkonu)

---

## 5. Realizace denního výkonu
> Záznam o skutečném průběhu jízdy Denního výkonu — zachycuje začátek, konec, ujetou vzdálenost, trajektorii a stav realizace. Ke každému DV může vzniknout více realizací (aktivní/historická).

**DS:** Realizace denního výkonu
**DDL:** `day_performance_realization`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Denní výkon | Reference → Denní výkon | Ano | DV, ke kterému se záznam vztahuje. |
| Datum a čas začátku | Datum a čas | Ne | Datum a čas začátku realizace DV. |
| Datum a čas konce | Datum a čas | Ne | Datum a čas konce realizace DV. |
| Časové využití | Desetinné číslo | Ano | Podíl skutečné doby jízdy a provozní doby. |
| Ujetá vzdálenost celkem | Celé číslo | Ano | Celková najetá vzdálenost (v metrech). |
| Ujetá vzdálenost s přívěsem | Celé číslo | Ano | Celková najetá vzdálenost s přívěsem (v metrech). |
| Doba jízdy celkem | Celé číslo | Ano | Celková doba jízdy v minutách. |
| Kompletní | Boolean | Ano | TRUE = všechny obsluhy lokací OS byly obslouženy. |
| Trajektorie | LineString | Ne | Uložená trajektorie vozidla. |
| Ukončena | Boolean | Ano | TRUE = realizace již proběhla. |
| Datum a čas potvrzení realizace | Datum a čas | Ne | Datum a čas potvrzení realizace DV. |
| Potvrdil realizaci | Reference → Uživatel | Ne | Uživatel, který potvrdil realizaci. |
| Poznámka | Řetězec (255) | Ne | Poznámka k realizaci DV. |
| Je aktivní | Boolean | Ano | Systémový atribut. |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut. |
| Datum poslední změny | Datum a čas | Ano | Systémový atribut. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| day_performance_id | int | NOT NULL | → day_performance.id |
| active | bit | NOT NULL | – |
| completed | bit | NOT NULL | – |
| traveled_distance | int | NOT NULL | – |
| start_date_time | datetimeoffset | NULL | – |
| finish_date_time | datetimeoffset | NULL | – |
| route | geometry | NULL | – |
| traveled_distance_with_trailer | int | NOT NULL | – |
| percentage_usage | float | NOT NULL | – |
| route_duration_in_minutes | int | NOT NULL | – |
| is_finished | bit | NOT NULL | – |
| note | nvarchar(MAX) | NULL | – |
| confirmed_by | int | NULL | → user.id |
| created_at | datetimeoffset | NOT NULL | – |
| updated_at | datetimeoffset | NOT NULL | – |
| confirmation_datetime | datetimeoffset | NULL | – |

**Asociace z DS:**
- Denní výkon (N:1)
- Uživatel – Potvrdil realizaci (N:1)


**Reference v tech. projektu:**
- 300ucxx (monitoring realizace DV; vyhodnocení, ukončení)
- 400ucxx (kontrola realizace DV; potvrzení)
- 502uc04–06 (založit/upravit/odebrat realizaci DV)

---

## 6. Objednávka
> Požadavek Zákazníka, kterým objednává u Klienta služby. Obsahuje Položky objednávky, z nichž se generují Objednané služby. Prochází stavy: Rozpracovaná, Nová, Aktivní, Uzavřená, Zrušená.

**DS:** Objednávka
**DDL:** `order`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Stav | Enumerace → Stav objednávky | A | Stav Objednávky. |
| Provozovna | Reference → Provozovna | A | Provozovna zajišťující realizaci. |
| Číslo objednávky | Řetězec (20) | A | Prefix provozovny + číslo (např. TRE-012345). |
| Poznámka | Řetězec (255) | N | Poznámka k Objednávce. |
| Statutární zástupce | Reference → Osoba zákazníka | N | Statutární zástupce Objednatele. |
| Objednavatel | Reference → Zákazník | P | Zákazník objednávající službu. |
| Bankovní spojení | Reference → Bankovní spojení | N | Bankovní spojení Objednatele. |
| Zástupce objednatele | Reference → Osoba zákazníka | N | Kontaktní osoba Objednatele. |
| Telefon | Řetězec (30) | N | Telefonní kontakt. |
| E-mail | Řetězec (255) | N | E-mailový kontakt. |
| Položky objednávky | Reference [] → Položka objednávky | P | Sada údajů definujících rozsah služeb. |
| Přijatá záloha | Nezáporné číslo, 2 des. místa (0-100 000) | N | Výše přijaté zálohy. |
| Vytvořeno | Datum a čas | A | Datum a čas vytvoření. |
| Změněno | Datum a čas | N | Datum a čas poslední změny. |
| Vytvořil | Reference → Uživatel | A | Uživatel, který záznam vytvořil. |
| Změnil | Reference → Uživatel | N | Uživatel, který záznam naposledy změnil. |
| Vytisknuta objednávka | Boolean | A | TRUE = Objednávka byla vytisknuta. |
| Vytisknuto potvrzení o vrácení zálohy | Boolean | A | TRUE = potvrzení o vrácení zálohy bylo vytisknuto. |
| Je obsažen objednaný úkon na výzvu | Boolean | A | TRUE = obsahuje alespoň jeden úkon na výzvu. |
| Počet vygenerovaných OS | Číslo (kladné, celé) | A | Počet OS přímo vygenerovaných z Objednávky. |
| Vytvořeno systémem | Boolean | Ano | TRUE = záznam vytvořen systémem. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| created_datetime | datetimeoffset | NULL | – |
| status | int | NOT NULL | → order_status.id |
| organization_unit_id | int | NULL | → organization_unit.id |
| contact_employee_id | int | NULL | → employee.id |
| note | nvarchar(255) | NULL | – |
| deposit | numeric(9,2) | NOT NULL | – |
| customer_id | int | NULL | → customer.id |
| customer_email | nvarchar(255) | NULL | – |
| customer_phone | nvarchar(255) | NULL | – |
| customer_office_address_id | int | NULL | → address.id |
| total_price | numeric(20,2) | NOT NULL | – |
| bank_account_id | int | NULL | → bank_account.id |
| customer_statutory_id | int | NULL | → customer_person.id |
| customer_contact_id | int | NULL | → customer_person.id |
| created_user_id | int | NULL | → user.id |
| updated_datetime | datetimeoffset | NULL | – |
| updated_user_id | int | NULL | → user.id |
| order_number | varchar(20) | NOT NULL | – |
| is_order_printed | bit | NOT NULL | – |
| is_deposit_confirmation_printed | bit | NOT NULL | – |
| on_demand | bit | NOT NULL | – |
| number_of_generated_ordered_services | int | NOT NULL | – |
| created_by_system | bit | NOT NULL | – |

**Asociace z DS:**
- Stav objednávky (enumerace)
- Provozovna (N:1)
- Zákazník (N:1)
- Osoba zákazníka – Statutární zástupce (N:1)
- Osoba zákazníka – Zástupce objednatele (N:1)
- Bankovní spojení (N:1)
- Položka objednávky (1:N)
- Uživatel – Vytvořil, Změnil (N:1)


**Reference v tech. projektu:**
- slovnik-78680658 (slovník pojmů)
- stavove-78680652 (stavové diagramy)
- 100ucxx (případy užití — objednávky)
- 100uixx (UI specifikace — objednávky)
- 102ucxx (šablony objednávek)
- 103ucxx (tiskové výstupy pro objednávky)
- 707ucxx (asynchronní export záznamů)

---

## 7. Položka objednávky
> Dílčí část Objednávky. Vztahuje se k právě jednomu Místu realizace a definuje Objednané nádoby a Objednané úkony pro dané místo.

**DS:** Položka objednávky
**DDL:** `order_item`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Objednávka | Reference → Objednávka | A | – |
| Podle smlouvy | Boolean | A | TRUE = realizováno podle Rámcové smlouvy. |
| Rámcová smlouva | Reference → Rámcová smlouva | P | Rámcová smlouva pro danou objednávku. |
| Místo realizace | Reference → Místo realizace | Ne | Místo, kde objednatel produkuje odpad. |
| Kontakt v místě realizace | Řetězec (255) | N | Kontaktní údaje. |
| Poznámka k místu realizace | Řetězec (255) | N | Poznámka k místu realizace. |
| Objednané nádoby | Reference [] → Objednaná nádoba | A | Specifikace typu a počtu nádob (min. 1). |
| Typ výpočtu ceny | Enumerace | P | Kalkulace / Paušál. |
| Výpočet ceny | Reference [] → Řádek výpočtu ceny | P | Údaje pro výpočet ceny. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| based_on_contract | bit | NULL | – |
| contract_id | int | NULL | → contract.id |
| execution_site_id | int | NULL | → execution_site.id |
| execution_site_contact | nvarchar(255) | NULL | – |
| execution_site_note | nvarchar(255) | NULL | – |
| price_calculation_type_id | int | NULL | → price_calculation_type.id |
| order_id | int | NULL | → order.id |
| item_number | int | NULL | – |

**Asociace z DS:**
- Objednávka (N:1)
- Rámcová smlouva (N:1)
- Místo realizace (N:1)
- Objednaná nádoba (1:N)
- Řádek výpočtu ceny (1:N)


**Reference v tech. projektu:**
- slovnik-78680658 (slovník pojmů)
- persistence-79430071 (persistence dat v dynamických entitách)
- 100ui01 (formulář objednávky — skupina Položky objednávky)

---

## 8. Objednaná nádoba
> Specifikace typu a počtu nádob v rámci Položky objednávky. Definuje Typ nádoby, Typ dopravy a kolekci Objednaných úkonů a Vývozů na Likvidační místo.

**DS:** Objednaná nádoba
**DDL:** `order_item_container`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Položka objednávky | Reference → Položka objednávky | A | – |
| Vývozy na Likvidační místo | Reference [] → Vývoz na Likvidační místo | A | Určení LM pro vývoz (min. 1). |
| Typ dopravy | Reference → Typ dopravy | Ne | Typ dopravy realizace. |
| Typ nádoby | Reference → Typ nádoby | P | Typ nádoby. |
| Počet nádob | Kladné celé číslo (1-255) | P | Počet objednaných nádob. |
| Vlastní nádoba | Boolean | A | TRUE = nádoba je majetkem organizace. |
| Objednané úkony | Reference [] → Objednaný úkon | P | Definice požadovaných úkonů (min. 1). |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| order_item_id | int | NOT NULL | → order_item.id |
| transport_type_id | int | NULL | → transport_type.id |
| container_type_id | int | NULL | → container_type.id |
| quantity | int | NULL | – |
| owned_container | bit | NULL | – |

**Asociace z DS:**
- Položka objednávky (N:1)
- Vývoz na Likvidační místo (1:N)
- Typ dopravy (N:1)
- Typ nádoby (N:1)
- Objednaný úkon (1:N)


**Reference v tech. projektu:**
- 100ucxx (objednávky — editace objednaných nádob)
- persistence-79430071 (persistence dat v dynamických entitách)

---

## 9. Objednaný úkon
> Předpis pro vznik Objednané služby v rámci Objednané nádoby. Vymezuje Typ úkonu (Přistavení, Vývoz, Odvoz, Manipulace, Sběr, Rozvoz), Podmínku vykonání (Výzva, Termín, Pravidlo opakování) a Likvidační místo.

**DS:** Objednaný úkon
**DDL:** `order_item_container_operation`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Objednaná nádoba | Reference → Objednaná nádoba | A | – |
| Typ úkonu | Enumerace → Typ úkonu | A | Přistavení, Vývoz, Odvoz, Manipulace, Sběr, Rozvoz. |
| Opakující se | Boolean | A | TRUE = úkon se opakuje. |
| Podmínka vykonání | Enumerace → Podmínka vykonání | A | Výzva, Termín, Pravidlo opakování. |
| Termín od | Datum | P | Povinné pro Podmínka vykonání = Termín. |
| Termín do | Datum | P | Povinné pro Podmínka vykonání = Termín. |
| Čas od | Čas | N | Začátek časového okna. |
| Čas do | Čas | N | Konec časového okna. |
| Pravidlo opakování | Reference → Pravidlo opakování | P | Povinné pro Podmínku = Pravidlo opakování. |
| Cílové místo realizace | Reference → Místo realizace | Ne | Místo realizace pro Manipulaci, Přepravu. |
| Výchozí likvidační místo | Reference → Likvidační místo | Ne | LM pro vývoz mezi LM. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| order_item_container_id | int | NOT NULL | → order_item_container.id |
| container_operation_type_id | int | NOT NULL | → container_operation_type.id |
| repetitive | bit | NOT NULL | – |
| execution_condition | varchar(50) | NOT NULL | – |
| date_from | date | NULL | – |
| date_to | date | NULL | – |
| unlimited_repetition | bit | NULL | – |
| repetition_rule | varchar(MAX) | NULL | – |
| time_from | varchar(150) | NULL | – |
| time_to | varchar(150) | NULL | – |
| final_execution_site_id | int | NULL | → execution_site.id |
| final_liquidation_site_id | int | NULL | → liquidation_site.id |

**Asociace z DS:**
- Objednaná nádoba (N:1)
- Typ úkonu (enumerace)
- Podmínka vykonání (enumerace)
- Pravidlo opakování (N:1)
- Místo realizace – Cílové (N:1)
- Likvidační místo – Výchozí (N:1)


**Reference v tech. projektu:**
- slovnik-78680658 (slovník pojmů)
- persistence-79430071 (persistence dat v dynamických entitách)
- 100ucxx (objednávky — editace objednaných úkonů)

---

## 10. Vývoz na Likvidační místo
> Vazební entita mezi Objednanou nádobou a Likvidačním místem — určuje, na které LM má být vyvezen konkrétní Druh odpadu. Ke každé Objednané nádobě alespoň jeden.

**DS:** Vývoz na Likvidační místo
**DDL:** `order_item_container_liquidation`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Objednaná nádoba | Reference → Objednaná nádoba | A | – |
| Druh odpadu | Reference → Druh odpadu | A | Druh odpadu předávaný v místě realizace. |
| Likvidační místo | Reference → Likvidační místo | P | Místo likvidace odpadu. |
| Kód nakládání | Řetězec (15) | Ano | Kód způsobu nakládání s odpadem. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| order_item_container_id | int | NOT NULL | → order_item_container.id |
| garbage_type_id | int | NULL | → garbage_type.id |
| liquidation_site_id | int | NULL | → liquidation_site.id |
| loading_code | nvarchar(15) | NULL | – |

**Asociace z DS:**
- Objednaná nádoba (N:1)
- Druh odpadu (N:1)
- Likvidační místo (N:1)


**Reference v tech. projektu:**
- persistence-79430071 (persistence dat v dynamických entitách)

---

## 11. Typ nádoby
> Číselník typů nádob (např. kontejner 5 m3, popelnice 240 l). Každý typ nádoby je vázán na konkrétní Typ dopravy a Provozovnu.

**DS:** Typ nádoby
**DDL:** `container_type`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (50) | Ne | Identifikátor v externím systému. |
| Název | Řetězec (50) | Ano | Název typu nádoby. |
| Popis | Řetězec (255) | Ne | Popis typu nádoby. |
| Typ dopravy | Reference → Typ dopravy | Ano | Typ dopravy pro daný typ nádoby. |
| Provozovna | Reference → Provozovna | Ne | Společnost, pod kterou typ nádoby spadá. |
| Je aktivní | Boolean | Ano | Systémový atribut. |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut. |
| Datum poslední změny | Datum a čas | Ano | Systémový atribut. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| name | nvarchar(50) | NOT NULL | – |
| description | nvarchar(255) | NULL | – |
| transport_type_id | int | NULL | – |
| active | bit | NOT NULL | – |
| winyx_id | int | NULL | – |
| organization_unit_id | int | NULL | – |
| created_at | datetimeoffset | NOT NULL | – |
| updated_at | datetimeoffset | NOT NULL | – |
| external_id | nvarchar(255) | NULL | – |

**Asociace z DS:**
- Typ dopravy (N:1)
- Provozovna (N:1)


**Reference v tech. projektu:**
- 500uc21–23 (založit/upravit/smazat typ nádoby)
- 500ui01 (přehled typů nádob)
- 603ucxx (Helios: synchronizace dat — MPG SK)
- 603uc18 (Helios: získat typy nádob)

---

## 12. Druh odpadu
> Číselník druhů odpadu s kódovým označením, popisem a kategorií (N, O, O/N). Používá se v celém řetězci od objednávky přes OS až po likvidaci.

**DS:** Druh odpadu
**DDL:** `garbage_type`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (50) | Ne | Identifikátor v externím systému. |
| Kód | Řetězec (15) | Ano | Kódové označení druhu odpadu. |
| Popis | Řetězec (255) | Ne | Popis druhu odpadu. |
| Kategorie | Řetězec (4) | Ano | Kategorie (N, O, O/N). |
| Je aktivní | Boolean | Ano | Systémový atribut. |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut. |
| Datum poslední změny | Datum a čas | Ano | Systémový atribut. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| code | varchar(15) | NOT NULL | – |
| name | nvarchar(50) | NOT NULL | – |
| description | nvarchar(255) | NULL | – |
| category | nvarchar(50) | NOT NULL | – |
| active | bit | NOT NULL | – |
| winyx_id | int | NULL | – |
| created_at | datetimeoffset | NOT NULL | – |
| updated_at | datetimeoffset | NOT NULL | – |
| external_id | nvarchar(255) | NULL | – |

**Asociace z DS:**
- (žádné explicitní)


**Reference v tech. projektu:**
- slovnik-78680658 (slovník pojmů)
- 500ui01 (přehled druhů odpadu)
- 603ucxx (Helios: synchronizace dat — MPG SK)
- 603uc21 (Helios: získat druhy odpadu)

---

## 13. Likvidační místo
> Místo, kde probíhá předání odpadu — např. skládka, sběrný dvůr, překladiště, třídírna nebo spalovna. Evidence zahrnuje adresu, souřadnice, podporované druhy odpadu a kód nakládání.

**DS:** Likvidační místo
**DDL:** `liquidation_site`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (50) | Ne | Identifikátor v externím systému. |
| Název | Řetězec (255/150) | Ne/Ano | Název likvidačního místa. |
| Adresa | Reference → Adresa | Ne/Ano | Adresa likvidačního místa. |
| Souřadnice | Point | Ne | Souřadnice LM. |
| Zeměpisná šířka | Desetinné číslo | Ne | Souřadnice LM. |
| Zeměpisná délka | Desetinné číslo | Ne | Souřadnice LM. |
| Kontakt | Řetězec (255) | Ne | Kontaktní údaje. |
| Poznámka | Řetězec (255) | Ne | Poznámka k LM. |
| Provozovna | Reference → Provozovna | Ne/Ano | Společnost, pod kterou je LM zařazeno. |
| Stav geokódování | Enumerace | Ne/Ano | Stav geokódování adresy. |
| Změna adresy | Boolean | Ano | TRUE = při synchronizaci došlo ke změně adresy. |
| Druhy odpadu | Reference [] → Druh odpadu | Ano | Kolekce podporovaných druhů odpadu. |
| Vlastní Likvidační místo | Boolean | Ne/Ano | TRUE = LM je majetkem organizace. |
| Kód nakládání | Řetězec (10) | Ano | Co se s odpadem dále děje. |
| Je aktivní | Boolean | Ano | Systémový atribut. |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut. |
| Datum poslední změny | Datum a čas | Ano | Systémový atribut. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| name | nvarchar(255) | NULL | – |
| address_id | int | NULL | – |
| lat | float | NULL | – |
| lng | float | NULL | – |
| contact | nvarchar(255) | NULL | – |
| note | nvarchar(255) | NULL | – |
| organization_unit_id | int | NULL | – |
| active | bit | NOT NULL | – |
| winyx_id_old | int | NULL | – |
| owned_liquidation_site | bit | NULL | – |
| geocoding_state | int | NULL | – |
| winyx_id | nvarchar(50) | NULL | – |
| created_at | datetimeoffset | NOT NULL | – |
| updated_at | datetimeoffset | NOT NULL | – |
| external_id | nvarchar(255) | NULL | – |

**Asociace z DS:**
- Adresa (N:1)
- Provozovna (N:1)
- Druh odpadu (1:N, via Druh odpadu likvidačního místa)
- Stav geokódování (enumerace)


**Reference v tech. projektu:**
- slovnik-78680658 (slovník pojmů)
- persistence-79430071 (persistence dat v dynamických entitách)
- 500uc19–20 (upravit LM, zobrazit přehled LM)
- 500ui08 (formulář LM)
- 500ui10 (detail LM)
- 603ucxx (Helios: synchronizace dat — MPG SK)
- 603uc13 (Helios: získat druhy odpadu LM)
- 200uc11–12 (vložit/odebrat LM do/z denního výkonu)

---

## 14. Druh odpadu likvidačního místa
> Evidence Druhů odpadu podporovaných v rámci jednotlivých Likvidačních míst. Vazební entita mezi LM, Druhem odpadu a Kódem nakládání, s vazbou na Provozovnu.

**DS:** Druh odpadu likvidačního místa
**DDL:** `liquidation_site_accepted_garbage_type`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (50) | Ne | Identifikátor v externím systému. |
| Likvidační místo | Reference → Likvidační místo | Ano | LM, pro které je uveden podporovaný druh odpadu. |
| Druh odpadu | Reference → Druh odpadu | Ano | Druh odpadu odebíraný v rámci LM. |
| Kód nakládání | Řetězec (15) | Ano | Kód způsobu nakládání s odpovídajícím druhem odpadu. |
| Provozovna | Reference → Provozovna | Ano | Provozovna, pro kterou je druh odpadu dostupný. |
| Je aktivní | Boolean | Ano | Systémový atribut. |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut. |
| Datum poslední změny | Datum a čas | Ano | Systémový atribut. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| liquidation_site_id | int | NULL | – |
| garbage_type_id | int | NULL | – |
| winyx_id | int | NULL | – |
| active | bit | NOT NULL | – |
| loading_code | nvarchar(15) | NULL | – |
| organization_unit_id | int | NULL | – |
| external_id | nvarchar(255) | NULL | – |

**Asociace z DS:**
- Likvidační místo (N:1)
- Druh odpadu (N:1)
- Provozovna (N:1)


**Reference v tech. projektu:**
- 603ucxx (Helios: synchronizace dat — MPG SK)
- 603uc13 (Helios: získat druhy odpadu likvidačního místa)

---

## 15. Objekt
> Entita pro uchování objektů, např. vozidel (odpovídá Objektům ve FLWW2). V DS pojmenováno "Objekt", DDL používá tabulku `vehicle` s rozlišením přes `vehicle_type_id`.

**DS:** Objekt (v DS pojmenováno "Objekt", nikoliv "Vozidlo" — zahrnuje vozidla i přívěsy)
**DDL:** `vehicle`

> Poznámka: DS rozlišuje entity Objekt (obecný) a Vozidlo (DS legacy). DDL používá tabulku `vehicle` pro oba koncepty, s rozlišením přes `vehicle_type_id`.

**Atributy z DS (Objekt):**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor FLW | Řetězec (50) | Ne | Externí identifikátor ve FLWW2. |
| Externí identifikátor ERP | Řetězec (50) | Ne | Externí identifikátor v ERP. |
| Název | Řetězec (50) | Ano | Název objektu. |
| Identifikátor objektu RZ | Řetězec (255) | Ne | Textová identifikace (u vozidel typicky RZ). |
| Evidenční číslo | Řetězec (255) | Ne | Evidenční číslo objektu. |
| Druh vozidla | Celé číslo | Ano | Kódové označení druhu vozidla z FLWW2. |
| Skupina | Řetězec (50) | Ne | Název skupiny. |
| Umístění | Řetězec (255) | Ne | Popis zařazení do hierarchie skupin. |
| Provozovna | Reference → Provozovna | Ne | Provozovna, pod kterou je vozidlo zařazeno. |
| Typ objektu | Reference → Typ objektu | Ano | Typ daného objektu (např. vozidlo). |
| Datum zařazení | Datum a čas | Ne | Datum zařazení do provozu. |
| Datum vyřazení | Datum a čas | Ne | Datum vyřazení z provozu. |
| Způsob vyhodnocení realizace | Enumerace | Ne | Způsob vyhodnocení realizace DV. |
| Je aktivní | Boolean | Ano | Systémový atribut. |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut. |
| Datum poslední změny | Datum a čas | Ano | Systémový atribut. |

**Atributy z DS (Vozidlo — legacy entita):**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor FLW | Číslo | N | Externí identifikátor ve FLW. |
| Externí identifikátor ERP | Číslo | N | Externí identifikátor v ERP. |
| Název | Řetězec (50) | A | Název vozidla. |
| RZ | Řetězec (50) | N | Registrační značka. |
| Skupina | Řetězec (50) | N | Název skupiny. |
| Umístění | Řetězec (255) | N | Popis zařazení do hierarchie skupin. |
| Typ dopravy | Reference → Typ dopravy | N | Typ dopravy vozidla. |
| Provozovna | Reference → Provozovna | N | Provozovna vozidla. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| name | nvarchar(100) | NULL | – |
| registration_plate | nvarchar(255) | NULL | – |
| organization_unit_id | int | NULL | – |
| flww_id | int | NULL | – |
| identification | nvarchar(255) | NULL | – |
| winyx_id | int | NULL | – |
| object_type | int | NULL | – |
| device_type_id | int | NULL | – |
| color | nvarchar(50) | NULL | – |
| color_inherited | bit | NULL | – |
| cost_center_code | nvarchar(255) | NULL | – |
| group_id | int | NULL | – |
| parent_name | nvarchar(255) | NULL | – |
| vehicle_full_path | nvarchar(255) | NULL | – |
| driver_id | nvarchar(10) | NULL | – |
| has_gps | bit | NULL | – |
| cost_center_id | nvarchar(10) | NULL | – |
| active | bit | NULL | – |
| evidence_number | nvarchar(255) | NULL | – |
| vin | nvarchar(255) | NULL | – |
| vehicle_type_id | int | NOT NULL | – |
| created_at | datetimeoffset | NOT NULL | – |
| updated_at | datetimeoffset | NOT NULL | – |
| external_id | nvarchar(255) | NULL | – |
| sort_id | int | NULL | – |
| date_insertion | datetimeoffset | NULL | – |
| date_rejection | datetimeoffset | NULL | – |
| realization_type | int | NULL | – |

**Asociace z DS (Objekt):**
- Provozovna (N:1)
- Typ objektu (N:1)
- Způsob vyhodnocení realizace (enumerace)


**Reference v tech. projektu:**
- 200ucxx (plánování — DV; přiřazení vozidla k DV)
- 203ucxx (časové využití vozidel)
- 500uc26 (zobrazit přehled vozidel)
- 500ui01 (přehled vozidel)
- 603ucxx (Helios: synchronizace dat — MPG SK)
- 604uc03 (FLWW2: získat objekt)
- 604uc10 (FLWW2: získat přiřazení objektu ke stanovišti)
- 604uc11 (FLWW2: získat nastavení objektů pro DV)
- 707ucxx (asynchronní export záznamů)

---

## 16. Typ dopravy objektu
> Vazební entita (N:M) mezi Objektem a Typem dopravy — specifikuje, které typy dopravy jsou daným objektem podporovány.

**DS:** Typ dopravy objektu
**DDL:** `vehicle__transport_type`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Objekt | Reference → Objekt | Ano | Objekt figurující v rámci vazby. |
| Typ dopravy | Reference → Typ dopravy | Ano | Typ dopravy figurující v rámci vazby. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| vehicle_id | int | NOT NULL | – |
| transport_type_id | int | NOT NULL | – |

**Asociace z DS:**
- Objekt (N:1)
- Typ dopravy (N:1)


**Reference v tech. projektu:**
- žádné specifické reference

---

## 17. Přívěs
> Přívěsné vozidlo, v DS samostatná entita. V DDL realizováno v tabulce `vehicle` s rozlišením přes `vehicle_type_id`. Přívěs může mít více typů dopravy a je přiřazován k Denním výkonům.

**DS:** Přívěs
**DDL:** neexistuje jako samostatná tabulka (přívěsy jsou v tabulce `vehicle` s rozlišením přes `vehicle_type_id`)

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor FLW | Číslo | N | Externí identifikátor ve FLW. |
| Externí identifikátor ERP | Číslo | N | Externí identifikátor v ERP. |
| Název | Řetězec (50) | A | Název přívěsu. |
| RZ | Řetězec (50) | N | Registrační značka přívěsu. |
| Skupina | Řetězec (50) | N | Název skupiny. |
| Umístění | Řetězec (255) | N | Popis zařazení do hierarchie skupin. |
| Typ dopravy [] | Reference → Typ dopravy | N | Typy dopravy přívěsu (může jich být více). |
| Provozovna | Reference → Provozovna | N | Provozovna přívěsu. |

**Asociace z DS:**
- Typ dopravy (N:M)
- Provozovna (N:1)


**Reference v tech. projektu:**
- 200uc23–24 (použít/zrušit přívěs u DV)
- 200uc51–53 (přiřadit/změnit/odebrat přívěs DV)

---

## 18. Typ dopravy
> Číselník typů dopravy (např. HNK, RNK, Valník, Lisovací vozidlo). Technická specifikace uložena jako JSON. Určuje dostupné typy nádob, vozidla a způsob plánování tras.

**DS:** Typ dopravy
**DDL:** `transport_type`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Název | Řetězec (50) | Ano | Název typu dopravy. |
| Je k dispozici | Boolean | Ano | TRUE = typ dopravy je dostupný v aplikaci. |
| Technická specifikace | Řetězec (MAX) | Ano | JSON specifikace technických parametrů. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int | NOT NULL | – |
| name | nvarchar(50) | NOT NULL | – |
| name_search | varchar(500) | NULL | – |
| is_available | bit | NOT NULL | – |

**Asociace z DS:**
- (žádné explicitní)


**Reference v tech. projektu:**
- datovy-78681044 (datový slovník — definice entity)

---

## 19. Provozovna
> Dílčí organizační jednotka Organizace. Provozovny přijímají Objednávky a plánují Trasy nezávisle na sobě. Hraje roli při autorizaci — uživatelé přistupují pouze k datům svých provozoven.

**DS:** Provozovna
**DDL:** `organization_unit`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor FLW | Řetězec (50) | Ne | Externí identifikátor ve FLWW2. |
| Externí identifikátor ERP | Řetězec (50) | Ne | Externí identifikátor v ERP. |
| Kód | Řetězec (10) | Ne | Kód provozovny (prefix čísla objednávky). |
| Název | Řetězec (150) | Ano | Název provozovny. |
| Poznámka | Řetězec (255) | Ne | Poznámka k provozovně. |
| Nadřazená provozovna | Reference → Provozovna | Ne | Nejbližší nadřazená provozovna. |
| Úroveň | Celé číslo | Ano | Úroveň ve stromové struktuře (0 = Společnost, 1 = Provozovna). |
| Adresa | Reference → Adresa | Ano | Adresa provozovny. |
| Stav geokódování | Enumerace | Ano | Stav geokódování adresy. |
| Změna adresy | Boolean | Ano | TRUE = při synchronizaci došlo ke změně adresy. |
| Provozní doba od | Čas | Ano | Začátek provozní doby. |
| Provozní doba do | Čas | Ano | Konec provozní doby. |
| Kontakt na dispečink | Řetězec (30) | Ne | Telefonní číslo na dispečink. |
| Souřadnice | Point | Ne | Souřadnice provozovny. |
| Externí identifikátor | Kladné celé číslo | Ne | ID provozovny z WinyX (legacy). |
| Je aktivní | Boolean | Ano | Systémový atribut. |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut. |
| Datum poslední změny | Datum a čas | Ano | Systémový atribut. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int | NOT NULL | – |
| prefix | nvarchar(10) | NULL | – |
| name | nvarchar(150) | NOT NULL | – |
| sequence_name | nvarchar(150) | NOT NULL | – |
| note | nvarchar(255) | NULL | – |
| address_id | int | NULL | – |
| superior_organization_unit | int | NULL | – |
| active | bit | NOT NULL | – |
| winyx_id | int | NULL | – |
| opening_hours_from | nvarchar(5) | NOT NULL | – |
| opening_hours_to | nvarchar(5) | NOT NULL | – |
| dispatcher_contact | nvarchar(30) | NULL | – |
| level | int | NOT NULL | – |
| created_at | datetimeoffset | NOT NULL | – |
| updated_at | datetimeoffset | NOT NULL | – |
| winyx_last_modified_at | datetimeoffset | NULL | – |
| external_id | nvarchar(255) | NULL | – |
| lat | float | NULL | – |
| lng | float | NULL | – |
| geocoding_state | int | NOT NULL | – |

**Asociace z DS:**
- Provozovna – Nadřazená (self-ref, N:1)
- Adresa (N:1)
- Stav geokódování (enumerace)


**Reference v tech. projektu:**
- slovnik-78680658 (slovník pojmů)
- 500uc38 (upravit provozovnu)
- 500ui01 (přehled provozoven)
- 500ui12 (formulář provozovny)
- 603ucxx (Helios: synchronizace dat — MPG SK)
- 604uc02 (FLWW2: získat provozovnu)
- 200uc15–16 (vložit/odebrat provozovnu do/z DV)

---

## 20. Informace o provozovně
> Doplňkové informace o provozovně pro tiskové sestavy (IČO, DIČ, IČ DPH, IBAN). Typicky se jedná o provozovny 1. úrovně (společnosti).

**DS:** Informace o provozovně
**DDL:** `organization_unit_info`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Provozovna | Reference → Provozovna | A | Provozovna, ke které se informace vztahují. |
| IČO | Řetězec (255) | N | Identifikační číslo osoby. |
| DIČ | Řetězec (255) | N | Daňové identifikační číslo. |
| IČ DPH | Řetězec (255) | N | Daňové identifikační číslo plátce DPH. |
| IBAN | Řetězec (255) | N | Číslo bankovního účtu ve formátu IBAN. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| organization_unit_id | int | NOT NULL | – |
| icdph | nvarchar(255) | NULL | – |
| ico | nvarchar(255) | NULL | – |
| dic | nvarchar(255) | NULL | – |
| iban | nvarchar(255) | NULL | – |

**Asociace z DS:**
- Provozovna (1:1)


**Reference v tech. projektu:**
- žádné specifické reference

---

## 21. Místo realizace
> Stanoviště Zákazníka, kde produkuje odpad. Na toto místo se přistavuje/vyváží nádoba. Z pohledu Klienta zde probíhá příjem odpadu.

**DS:** Místo realizace
**DDL:** `execution_site`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (50) | Ne | Identifikátor v externím systému. |
| Zákazník | Reference → Zákazník | Ano | Zákazník, ke kterému je MR přiřazeno. |
| Název | Řetězec (150/255) | Ne/Ano | Název místa realizace. |
| Adresa | Reference → Adresa | Ano | Adresa místa realizace. |
| Souřadnice | Point | Ne | Souřadnice MR. |
| Zeměpisná šířka | Desetinné číslo | Ne | Souřadnice MR. |
| Zeměpisná délka | Desetinné číslo | Ne | Souřadnice MR. |
| Kontakt | Řetězec (255) | Ne | Kontaktní údaje. |
| Poznámka | Řetězec (255) | Ne | Poznámka k MR. |
| Provozovna | Reference → Provozovna | Ne | Provozovna, pro kterou je MR dostupné. |
| Výchozí | Boolean | Ne | TRUE = výchozí místo realizace. |
| Stav geokódování | Enumerace | Ne/Ano | Stav geokódování adresy. |
| Změna adresy | Boolean | Ano | TRUE = při synchronizaci došlo ke změně adresy. |
| Je aktivní | Boolean | Ano | Systémový atribut. |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut. |
| Datum poslední změny | Datum a čas | Ano | Systémový atribut. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| address_id | int | NOT NULL | → address.id |
| lat | float | NULL | – |
| lng | float | NULL | – |
| contact | nvarchar(255) | NULL | – |
| note | nvarchar(255) | NULL | – |
| name | nvarchar(255) | NULL | – |
| customer_id | int | NOT NULL | → customer.id |
| active | bit | NOT NULL | – |
| winyx_id_old | int | NULL | – |
| geocoding_state | int | NULL | – |
| winyx_id | nvarchar(50) | NULL | – |
| created_at | datetimeoffset | NOT NULL | – |
| updated_at | datetimeoffset | NOT NULL | – |
| organization_unit_id | int | NULL | – |
| external_id | nvarchar(255) | NULL | – |

**Asociace z DS:**
- Zákazník (N:1)
- Adresa (N:1)
- Provozovna (N:1)
- Stav geokódování (enumerace)


**Reference v tech. projektu:**
- slovnik-78680658 (slovník pojmů)
- 500uc07–09 (založit/upravit/smazat místo realizace)
- 500ui04 (formulář místa realizace)
- 603ucxx (Helios: synchronizace dat — MPG SK)
- 603uc10–11 (získat/odeslat místa realizace)
- 207uc01 (agregovat OS denního výkonu dle místa realizace)

---

## 22. Stanoviště
> Místo, kde jsou odstavována vozidla (pokud to není přímo v areálu provozovny). Stanoviště mohou tvořit hierarchii (nadřazené/podřízené).

**DS:** Stanoviště
**DDL:** `site`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (50) | Ne | Identifikátor v externím systému. |
| Název | Řetězec (100) | Ano | Název stanoviště. |
| Kód | Řetězec (20) | Ne | Kód stanoviště. |
| Externí identifikace | Řetězec (20) | Ne | Externí identifikace z FLWW2. |
| Nadřazené stanoviště | Reference → Stanoviště | Ne | Nadřazené stanoviště. |
| Adresa | Reference → Adresa | Ne | Adresa stanoviště. |
| Zeměpisná šířka | Desetinné číslo | Ne | Souřadnice stanoviště. |
| Zeměpisná délka | Desetinné číslo | Ne | Souřadnice stanoviště. |
| Je aktivní | Boolean | Ano | Systémový atribut. |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut. |
| Datum poslední změny | Datum a čas | Ano | Systémový atribut. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int | NOT NULL | – |
| name | nvarchar(255) | NOT NULL | – |
| code | nvarchar(255) | NULL | – |
| external_id | nvarchar(255) | NULL | – |
| parent_site_id | int | NULL | – |
| address_id | int | NULL | – |
| lat | float | NULL | – |
| lng | float | NULL | – |
| active | bit | NOT NULL | – |
| created_at | datetimeoffset | NOT NULL | – |
| updated_at | datetimeoffset | NULL | – |

**Asociace z DS:**
- Stanoviště – Nadřazené (self-ref, N:1)
- Adresa (N:1)


**Reference v tech. projektu:**
- slovnik-78680658 (slovník pojmů)
- 604uc09 (FLWW2: získat stanoviště)

---

## 23. Přiřazení objektu ke stanovišti
> Vazba mezi Objektem a Stanovištěm s časovou platností (od–do). Představuje informaci o místě, kde je vozidlo odstavováno v daném období.

**DS:** Přiřazení objektu ke stanovišti
**DDL:** `site_vehicle_assignment`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (50) | Ne | Identifikátor v externím systému. |
| Objekt | Reference → Objekt | Ano | Objekt figurující v dané vazbě. |
| Stanoviště | Reference → Stanoviště | Ano | Stanoviště figurující v dané vazbě. |
| Platnost od | Datum a čas | Ano | Začátek platnosti přiřazení. |
| Platnost do | Datum a čas | Ne | Konec platnosti přiřazení. |
| Je aktivní | Boolean | Ano | Systémový atribut. |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut. |
| Datum poslední změny | Datum a čas | Ano | Systémový atribut. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int | NOT NULL | – |
| site_id | int | NOT NULL | – |
| vehicle_id | int | NOT NULL | – |
| valid_from | date | NOT NULL | – |
| valid_to | date | NULL | – |
| active | bit | NOT NULL | – |
| created_at | datetimeoffset | NOT NULL | – |
| updated_at | datetimeoffset | NULL | – |

**Asociace z DS:**
- Objekt (N:1)
- Stanoviště (N:1)


**Reference v tech. projektu:**
- 604uc10 (FLWW2: získat přiřazení objektu ke stanovišti)

---

## 24. Nastavení objektu pro denní výkon
> Informace o objektu jako podklad pro generování Denních výkonů — barva, provozní doba. Data získána synchronizací s FLWW2, který je zdrojem pravdy pro informace o objektech.

**DS:** Nastavení objektu pro denní výkon
**DDL:** `day_performance_settings`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (50) | Ne | Identifikátor v externím systému. |
| Objekt | Reference → Objekt | Ano | Objekt, ke kterému jsou informace přiřazeny. |
| Barva | Řetězec (8) | Ne | Kód barvy daného objektu. |
| Provozní doba od | Řetězec (10) | Ne | Začátek provozní doby. |
| Provozní doba do | Řetězec (10) | Ne | Konec provozní doby. |
| Platnost od | Datum a čas | Ano | Začátek platnosti přiřazení. |
| Platnost do | Datum a čas | Ne | Konec platnosti přiřazení. |
| Je aktivní | Boolean | Ano | Systémový atribut. |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut. |
| Datum poslední změny | Datum a čas | Ano | Systémový atribut. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int | NOT NULL | – |
| vehicle_id | int | NOT NULL | – |
| valid_from | date | NOT NULL | – |
| valid_to | date | NULL | – |
| opening_hours_from | nvarchar(50) | NULL | – |
| opening_hours_to | nvarchar(50) | NULL | – |
| color | nvarchar(50) | NULL | – |
| active | bit | NOT NULL | – |
| created_at | datetimeoffset | NOT NULL | – |
| updated_at | datetimeoffset | NULL | – |

**Asociace z DS:**
- Objekt (N:1)


**Reference v tech. projektu:**
- 604uc11 (FLWW2: získat nastavení objektů pro denní výkony)

---

## 25. Zákazník
> Zákazník klienta. V kontextu systému RoadPlan vystupuje jako objednatel, který poptává služby. Evidence zákazníků je společná pro všechny Provozovny.

**DS:** Zákazník
**DDL:** `customer`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Řetězec (50) | Ne | Identifikátor v externím systému. |
| Název | Řetězec (255) | Ano | Název organizace / jméno FO. |
| Je podnikatel / Typ zákazníka | Boolean / Enumerace | Ano | Podnikatel / Nepodnikatel. |
| Právní forma | Reference → Právní forma | Ne | Právní forma. |
| IČO | Řetězec (20) | Ne | Identifikační číslo organizace. |
| DIČ | Řetězec (20) | Ne | Daňové identifikační číslo. |
| IČ DPH | Řetězec (20) | Ne | Daňové identifikační číslo plátce DPH. |
| Plátce DPH | Boolean | Ano | TRUE = je plátce DPH. |
| Adresa sídla | Reference → Adresa | Ne | Fakturační adresa. |
| Osoby | Reference [] → Osoba zákazníka | Ne | Osoby evidované k zákazníkovi. |
| Místa realizace | Reference [] → Místo realizace | Ne | Místa realizace zákazníka. |
| Bankovní spojení | Reference [] → Bankovní spojení | Ne | Bankovní spojení zákazníka. |
| Výchozí statutární zástupce | Reference → Osoba zákazníka | Ne | Výchozí statutární zástupce. |
| Výchozí kontaktní osoba | Reference → Osoba zákazníka | Ne | Výchozí kontaktní osoba. |
| Výchozí místo realizace | Reference → Místo realizace | Ne | Výchozí místo realizace. |
| Výchozí bankovní spojení | Reference → Bankovní spojení | Ne | Výchozí bankovní spojení. |
| Poznámka | Řetězec (255) | Ne | Poznámka k zákazníkovi. |
| Bonita | Boolean | Ne | TRUE = zákazník je v prodlení s platbou. |
| Vytvořil | Reference → Uživatel | Ne | Uživatel, který záznam vytvořil. |
| Odeslán ke kontrole | Boolean | Ne | TRUE = zákazník exportován do WinyX. |
| Načten z rejstříku | Boolean | Ano | TRUE = data získána z rejstříku podnikatelských subjektů. |
| Je aktivní | Boolean | Ano | Systémový atribut. |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut. |
| Datum poslední změny | Datum a čas | Ano | Systémový atribut. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| name | nvarchar(255) | NOT NULL | – |
| legal_form | int | NULL | – |
| identification_number | nvarchar(20) | NULL | – |
| tax_number | nvarchar(20) | NULL | – |
| office_address_id | int | NULL | → address.id |
| note | nvarchar(255) | NULL | – |
| vat_number | varchar(20) | NULL | – |
| vat_payer | bit | NULL | – |
| is_business | bit | NOT NULL | – |
| df_statutory_id | int | NULL | → customer_person.id |
| df_contact_id | int | NULL | → customer_person.id |
| df_execution_site_id | int | NULL | → execution_site.id |
| df_bank_account_id | int | NULL | → bank_account.id |
| winyx_id | int | NULL | – |
| created_by | int | NULL | → user.id |
| active | bit | NOT NULL | – |
| is_synchronizing | bit | NOT NULL | – |
| solvency | bit | NULL | – |
| created_at | datetimeoffset | NOT NULL | – |
| updated_at | datetimeoffset | NOT NULL | – |
| is_created_from_registry | bit | NOT NULL | – |
| external_id | nvarchar(255) | NULL | – |

**Asociace z DS:**
- Právní forma (N:1)
- Adresa (N:1)
- Osoba zákazníka (1:N)
- Místo realizace (1:N)
- Bankovní spojení (1:N)
- Uživatel – Vytvořil (N:1)


**Reference v tech. projektu:**
- slovnik-78680658 (slovník pojmů)
- 500ucxx (správa dat — zákazníci, osoby, místa realizace)
- 500uixx (UI specifikace — správa dat)
- 601uc01 (export nových zákazníků)
- 603ucxx (Helios: synchronizace dat — MPG SK)
- 603uc02–05 (Helios: získat/odeslat zákazníky a osoby zákazníka)
- 603uc23 (Helios: získat výsledky schválení zákazníků)

---

## 26. Přívěs denního výkonu
> Vazební entita pro přiřazení přívěsu (Objektu typu přívěs) k Dennímu výkonu.

**DS:** Přívěs denního výkonu
**DDL:** `day_performance_trailers`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Denní výkon | Reference → Denní výkon | Ano | DV, ke kterému je přiřazen přívěs. |
| Přívěs | Reference → Objekt | Ano | Objekt přiřazený jako přívěs k DV. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| day_performance_id | int | NOT NULL | → day_performance.id |
| trailer_id | int | NOT NULL | → vehicle.id |

**Asociace z DS:**
- Denní výkon (N:1)
- Objekt (N:1)


**Reference v tech. projektu:**
- 200uc51–53 (přiřadit/změnit/odebrat přívěs DV)

---

## 27. Interval omezení denního výkonu
> Časové omezení v rámci Denního výkonu — definuje typ omezení a časový interval (od–do), kdy platí.

**DS:** Interval omezení denního výkonu
**DDL:** `day_performance_interval_restriction`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Denní výkon | Reference → Denní výkon | Ano | DV, v rámci kterého se interval vyskytuje. |
| Typ omezení | Enumerace → Typ intervalu omezení DV | Ano | Typ omezení. |
| Čas omezení od | Čas | Ano | Čas začátku omezení. |
| Čas omezení do | Čas | Ano | Čas konce omezení. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| day_performance_id | int | NOT NULL | → day_performance.id |
| restriction_type | int | NOT NULL | → day_performance_restriction_type.id |
| time_from | nvarchar(5) | NOT NULL | – |
| time_to | nvarchar(5) | NOT NULL | – |

**Asociace z DS:**
- Denní výkon (N:1)
- Typ intervalu omezení DV (enumerace)


**Reference v tech. projektu:**
- 206uc12 (zobrazit panel intervalu omezení DV)

---

## 28. Průběh editace denního výkonu
> Snapshot (JSON uložený stav) Denního výkonu zachycený v průběhu editace — umožňuje uložení a načtení rozpracovaného stavu DV.

**DS:** Průběh editace denního výkonu
**DDL:** `day_performance_snapshot`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Vytvořeno | Datum a čas | A | Okamžik vytvoření průběhu editace. |
| Denní výkon | Reference → Denní výkon (json) | A | DV, jehož průběh byl uložen. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| day_performance_id | int | NOT NULL | → day_performance.id |
| day_performance_snapshot | nvarchar(MAX) | NOT NULL | – |
| snapshot_phase | nvarchar(10) | NULL | → day_performance_snapshot_phase.id |
| created_at | datetimeoffset | NOT NULL | – |
| updated_at | datetimeoffset | NOT NULL | – |

**Asociace z DS:**
- Denní výkon (N:1)


**Reference v tech. projektu:**
- 200uc39 (uložit průběh editace DV)
- 200uc40 (načíst uložený průběh editace DV)
- 200uc41 (smazat uložené průběhy editace DV)

---

## 29. Obsluha lokace objednané služby
> Záznam o skutečné obsluze Lokace OS v rámci Realizace DV — zachycuje datum/čas obsluhy, způsob odbavení a zda byla lokace úspěšně realizována.

**DS:** Obsluha lokace objednané služby
**DDL:** `ordered_service_location_realization`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Realizace denního výkonu | Reference → Realizace DV | Ano | Realizace DV, v rámci které byla obsluha vyhodnocena. |
| Lokace objednané služby | Reference → Lokace OS | Ano | Lokace OS, která byla obsloužena. |
| Datum a čas obsluhy | Datum a čas | Ne | Datum a čas vyhodnocení obsluhy. |
| Událost manipulace s nádobou | Reference → Událost manipulace | Ne | OBSOLETE atribut. |
| Je realizována | Boolean | Ne | TRUE = úspěšná realizace lokace. |
| Způsob odbavení | Enumerace | Ano | Způsob odbavení lokace OS. |
| Identifikátor záznamu o realizaci | Celé číslo | Ne | Identifikátor záznamu o realizaci. |
| Správné pořadí | Boolean | Ne | TRUE = obsloužena ve správném pořadí. |
| Poznámka | Řetězec (255) | Ne | Poznámka k obsluze. |
| Zeměpisná šířka | Desetinné číslo | Ne | OBSOLETE. |
| Zeměpisná délka | Desetinné číslo | Ne | OBSOLETE. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| day_performance_realization_id | int | NOT NULL | → day_performance_realization.id |
| operation_date_time | datetimeoffset | NULL | – |
| lat | float | NULL | – |
| lng | float | NULL | – |
| ordered_service_location_id | int | NOT NULL | → ordered_service_location.id |
| note | nvarchar(MAX) | NULL | – |
| is_visited | bit | NOT NULL | – |
| checking_type | int | NOT NULL | → ordered_service_location_checking_type.id |
| realization_record_identifier | int | NULL | – |

**Asociace z DS:**
- Realizace denního výkonu (N:1)
- Lokace objednané služby (N:1)
- Způsob odbavení lokace OS (enumerace)


**Reference v tech. projektu:**
- 502uc07–09 (založit/upravit/odebrat obsluhu lokace OS)
- 400uc17 (změnit poznámku u obsluhy lokace OS)

---

## 30. FOB informace o realizaci lokace OS
> Informace o realizaci lokace OS získané v rámci zprávy z FOB (mobilního zařízení řidiče) — identifikátor zprávy, souřadnice, způsob a čas odbavení.

**DS:** FOB informace o realizaci lokace objednané služby
**DDL:** `fob_ordered_service_location_realization`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Identifikátor | Celé číslo | Ano | Identifikátor záznamu v DB. |
| Identifikátor zprávy | Řetězec | Ano | Identifikátor zprávy s informacemi o odbavení. |
| Externí identifikátor objektu | Řetězec | Ano | Identifikátor objektu ve FLWW2. |
| Identifikátor lokace OS | Celé číslo | Ne/Ano | Identifikátor lokace OS. |
| Způsob odbavení | Enumerace | Ano | Způsob odbavení lokace OS. |
| Datum a čas odbavení | Datum a čas | Ano | Datum a čas odbavení. |
| Zeměpisná šířka | Desetinné číslo | Ano | Souřadnice odbavení. |
| Zeměpisná délka | Desetinné číslo | Ano | Souřadnice odbavení. |
| Adresa odbavení | Reference → Adresa | Ne | Adresa odbavení (geokódováno). |
| Datum vytvoření | Datum a čas | Ano | Systémový atribut. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| message_identifier | nvarchar(MAX) | NOT NULL | – |
| flww_vehicle_id | int | NOT NULL | → vehicle.flww_id |
| ordered_service_location_id | int | NULL | → ordered_service_location.id |
| checking_type | int | NOT NULL | → ordered_service_location_checking_type.id |
| checking_datetime | datetimeoffset | NOT NULL | – |
| lat | float | NOT NULL | – |
| lng | float | NOT NULL | – |
| address_id | int | NULL | → address.id |
| created_at | datetimeoffset | NOT NULL | – |

**Asociace z DS:**
- Lokace objednané služby (N:1)
- Způsob odbavení lokace OS (enumerace)
- Adresa (N:1)


**Reference v tech. projektu:**
- 502uc03 (založit FOB informace o realizaci lokace OS)
- 602uc14 (přijmout informace o realizaci lokace OS z FOB)
- 602ucxx (FOB komunikační rozhraní)

---

## 31. Množstevní jednotka
> Číselník množstevních jednotek (např. m3, kg, ks). Některé jednotky jsou dostupné pro Druhy odpadu.

**DS:** Množstevní jednotka
**DDL:** `measure_unit`

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Název | Řetězec (50) | A | Název jednotky (např. Kubický metr). |
| Značka | Řetězec (15) | A | Značka jednotky (např. m3). |
| Dostupné pro druh odpadu | Boolean | A | TRUE = jednotka je k dispozici pro druhy odpadu. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable | FK |
|---|---|---|---|
| id | int IDENTITY | NOT NULL | – |
| name | nvarchar(50) | NOT NULL | – |
| symbol | nvarchar(15) | NOT NULL | – |

**Asociace z DS:**
- (žádné explicitní)


**Reference v tech. projektu:**
- žádné specifické reference

---

## 32. Kód nakládání
> Evidence kódů pro různé způsoby nakládání s odpadem. V DDL implementováno jako řetězcový atribut v rámci jiných tabulek (nikoliv jako samostatná tabulka).

**DS:** Kód nakládání
**DDL:** neexistuje jako samostatná tabulka (v DDL je kód nakládání uložen jako řetězcový atribut v rámci jiných tabulek)

**Atributy z DS:**

| Atribut | Typ (DS) | Povinný | Popis |
|---|---|---|---|
| Externí identifikátor | Číslo | Ano | Identifikátor v externím systému. |
| Kód | Řetězec (15) | Ano | Kódové označení. |

**Asociace z DS:**
- (žádné explicitní)


**Reference v tech. projektu:**
- 603ucxx (Helios: synchronizace dat — MPG SK)

---

## 33. Stav objednané služby
> Enumerace stavů, kterými prochází Objednaná služba: K naplánování, Naplánovaná, Ke kontrole, Realizovaná, Nezrealizovaná, Zrušená.

**DS:** Stav objednané služby (enumerace)
**DDL:** `ordered_service_status`

| Položka (DS) | Popis |
|---|---|
| K naplánování | OS vytvořena a připravena k naplánování. |
| Naplánovaná | OS vložena na DV, dosud nerealizována. |
| Ke kontrole | OS vložena na DV, čeká na kontrolu realizace. |
| Zrušená | OS byla zrušena. |
| Realizovaná | OS vložena na DV, potvrzena jako realizovaná. |
| Nezrealizovaná | OS vložena na DV, nebyla potvrzena jako realizovaná. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable |
|---|---|---|
| id | int | NOT NULL |
| name | nvarchar(25) | NULL |


**Reference v tech. projektu:**
- stavove-78680652 (stavové diagramy)
- 101ucxx (objednané služby — přechody stavů)

---

## 34. Určení objednané služby
> Enumerace určující původ Objednané služby: přímý vznik z objednávky, rozdělení, místo dočasného uložení, nebo přeplánování.

**DS:** Určení objednané služby (enumerace)
**DDL:** `ordered_service_type_code`

| Položka (DS) | Popis |
|---|---|
| – | OS vznikla přímo z Objednávky. |
| Rozdělení | OS vznikla při rozdělení OS. |
| Místo dočasného uložení | OS vznikla při vytvoření MDU. |
| Přeplánování | OS vznikla přeplánováním nezrealizované OS. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable |
|---|---|---|
| type | nvarchar(200) | NOT NULL |
| description | nvarchar(MAX) | NULL |


**Reference v tech. projektu:**
- 101ucxx (objednané služby)
- 400ucxx (kontrola realizace — přeplánování)

---

## 35. Způsob přeplánování OS
> Enumerace způsobu dalšího řešení nezrealizované Objednané služby: Přeplánováno, Rozděleno, Místo dočasného uložení, Zastaveno.

**DS:** Způsob přeplánování objednané služby (enumerace)
**DDL:** `ordered_service_reschedule_method_code`

| Položka (DS) | Popis |
|---|---|
| – | OS nebyla přeplánována. |
| Přeplánováno | OS byla přeplánována jako celek. |
| Rozděleno | OS byla částečně přeplánována. |
| Místo dočasného uložení | OS byla částečně přeplánována s MDU. |
| Zastaveno | OS byla zastavena. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable |
|---|---|---|
| type | nvarchar(200) | NOT NULL |
| description | nvarchar(MAX) | NULL |


**Reference v tech. projektu:**
- 400ucxx (kontrola realizace — přeplánování OS)

---

## 36. Způsob rozdělení OS
> Enumerace způsobu rozdělení Objednané služby na dvě části: Rozdělení (bez vložení lokace) nebo Místo dočasného uložení (s vložením lokace MDU).

**DS:** Způsob rozdělení objednané služby (enumerace)
**DDL:** neexistuje jako samostatná tabulka (v DDL uloženo jako řetězec v `ordered_service.parent_ordered_service_split_type` / `child_ordered_service_split_type`)

| Položka (DS) | Popis |
|---|---|
| Rozdělení | OS rozdělena na 2 části bez vložení další lokace. |
| Místo dočasného uložení | OS rozdělena na 2 části s vložením lokace MDU. |


**Reference v tech. projektu:**
- 200uc34 (rozdělit položku denního výkonu)
- 200uc35 (sloučit rozdělenou OS)

---

## 37. Stav denního výkonu
> Enumerace stavů Denního výkonu: Plánovaný, Schválený, Plněný, Ke kontrole, Uzavřený, Zrušený, Vypůjčený.

**DS:** Stav denního výkonu (enumerace)
**DDL:** `day_performance_status`

| Položka (DS) | Popis |
|---|---|
| Plánovaný | DV vytvořen, připraven na plánování. |
| Schválený | DV schválen, čeká na realizaci. |
| Plněný | DV je právě realizován. |
| Ke kontrole | DV realizován, čeká na potvrzení. |
| Uzavřený | DV potvrzen jako realizovaný. |
| Zrušený | DV neschválený, zrušen na konci dne realizace. |
| Vypůjčený | DV domovské Provozovny při přiřazení Vozidla k jinému DV. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable |
|---|---|---|
| id | int IDENTITY | NOT NULL |
| name | nvarchar(50) | NOT NULL |


**Reference v tech. projektu:**
- stavove-78680652 (stavové diagramy)
- 200ucxx (plánování DV — přechody stavů)
- 300ucxx (monitoring realizace DV)
- 400ucxx (kontrola realizace DV)

---

## 38. Typ položky denního výkonu
> Enumerace typů Položky DV: Objednaná služba, Lokace objednané služby, Časový interval (přestávka), Rozdělení.

**DS:** Typ položky denního výkonu (enumerace)
**DDL:** `day_performance_item_type`

| Položka (DS) | Popis |
|---|---|
| Objednaná služba | Položka DV určena pro OS. |
| Lokace objednané služby | Položka DV určena pro Lokaci OS. |
| Časový interval | Položka DV určena pro přestávku. |
| Rozdělení | Speciální typ Lokace OS pro rozdělení směny. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable |
|---|---|---|
| id | int IDENTITY | NOT NULL |
| code | nvarchar(50) | NOT NULL |


**Reference v tech. projektu:**
- 200ucxx (plánování DV — vkládání položek)

---

## 39. Typ lokace objednané služby
> Enumerace typů lokace: Místo realizace, Likvidační místo, Výchozí LM, Provozovna, Jiné, Přestávka, Rozdělení, Místo dočasného uložení, Časový interval.

**DS:** Typ lokace objednané služby (enumerace)
**DDL:** `ordered_service_location_type`

| Položka (DS) | Popis |
|---|---|
| Místo realizace | Lokace OS = místo s nádobou zákazníka. |
| Likvidační místo | Lokace OS = místo výsypu nádoby. |
| Výchozí likvidační místo | Lokace OS = LM, z kterého se převáží odpad. |
| Provozovna | Lokace OS = výchozí místo provozovny. |
| Jiné | Lokace OS = bod průjezdu, start či cíl. |
| Přestávka | Lokace OS = přestávka. |
| Rozdělení | Lokace OS = rozdělení směny (cesta na provozovnu). |
| Místo dočasného uložení | Lokace OS = místo dočasného odložení nádoby. |
| Časový interval | Lokace OS daná časovým intervalem. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable |
|---|---|---|
| id | int IDENTITY | NOT NULL |
| name | nvarchar(50) | NOT NULL |


**Reference v tech. projektu:**
- 201ucxx (plánování — objednané služby)

---

## 40. Akce v lokaci objednané služby
> Enumerace akcí v lokaci OS: Vyzvednout, Umístit, Vyprázdnit, Navštívit, Přestávka.

**DS:** Akce v lokaci objednané služby (enumerace)
**DDL:** `ordered_service_location_action`

| Položka (DS) | Popis |
|---|---|
| Vyzvednout | Kontejner je vyzvednut z požadovaného místa. |
| Umístit | Kontejner je umístěn na požadované místo. |
| Vyprázdnit | Kontejner je vyprázdněn na odpovídajícím místě. |
| Navštívit | Nedochází k manipulaci s kontejnerem. |
| Přestávka | Využití stanovené přestávky. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable |
|---|---|---|
| id | int IDENTITY | NOT NULL |
| name | nvarchar(50) | NOT NULL |


**Reference v tech. projektu:**
- 201ucxx (plánování — objednané služby)

---

## 41. Způsob odbavení lokace OS
> Enumerace způsobu odbavení lokace OS: Neodbavena (výchozí stav), Potvrzena realizace, Přeskočena realizace.

**DS:** Způsob odbavení lokace objednané služby (enumerace)
**DDL:** `ordered_service_location_checking_type`

| Položka (DS) | Popis |
|---|---|
| Neodbavena | Nad lokací OS nebyla provedena žádná akce (výchozí stav). |
| Potvrzena realizace | Byla potvrzena realizace lokace OS. |
| Přeskočena realizace | Lokace OS byla přeskočena. |

**Sloupce z DDL:**

| Sloupec | Typ (DDL) | Nullable |
|---|---|---|
| id | int | NOT NULL |
| name | nvarchar(255) | NOT NULL |


**Reference v tech. projektu:**
- 300ucxx (monitoring realizace DV)
- 400ucxx (kontrola realizace DV)
- 502ucxx (FOB: zpracování realizace)

---

## 42. Entity neexistující v DS ani DDL (nové pro CS)

Následující entity jsou identifikovány z Cílového konceptu jako potřebné pro Cyklické svozy, ale v aktuálním DS ani DDL aplikace RoadPlan neexistují. Budou nově vytvořeny.

| Entity | Poznámka |
|---|---|
| **Okruh** (circuit) | Nová entita pro CS — definice okruhu svozové trasy. |
| **Rozvrh** (schedule) | Nová entita pro CS — rozvrh obsluh / plánování okruhů. |
| **Okruh dne** (daily_circuit / circuit_day) | Nová entita pro CS — instance okruhu pro konkrétní den (obdoba DV pro CS). |
| **Skupina odpadu** (garbage_group) | Nová/sdílená entita — seskupení druhů odpadu pro účely CS. |
| **RPO / Položka objednávky CS** | V RP bude importovaná podmnožina dat z PP (revize položek objednávek). Stávající `order_item` je pro kontejnerovou dopravu. |

---

## Doplňkové DDL tabulky bez protějšku v DS

Následující tabulky existují v DDL a souvisí s CS-relevantními entitami, ale nemají přímý protějšek v DS:

| DDL tabulka | Popis |
|---|---|
| `temporary_day_performance` | Dočasný DV s JSON snapshotem. |
| `fob_device_day_performance` | Přihlášené FOB zařízení na DV. |
| `container_operation_type` | Číselník typů úkonů (kontejnerových operací). |
| `execution_condition_type` | Číselník podmínek vykonání. |
| `vehicle_realizaton_type` | Číselník způsobů vyhodnocení realizace. |
| `vehicle_type_enum` | Číselník typů objektů (vozidlo/přívěs). |
| `day_performance_restriction_type` | Číselník typů omezení intervalu DV. |
| `day_performance_snapshot_phase` | Číselník fází snapshotů DV. |
| `execution_site_erp_mapping` | Mapování míst realizace na ERP. |
| `organization_unit_erp_mapping` | Mapování provozoven na ERP pobočky. |
