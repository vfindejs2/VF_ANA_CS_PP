# Nesrovnalosti: Datove slovniky vs. DDL

> **Datum:** 2026-02-23
> **Zdroje:**
> - DS-PP + DDL-PP: `docs/extrakce-PP-entity-a-mapovani.md` (13 entit)
> - DS-RP + DDL-RP: `docs/extrakce-RP-entity-a-mapovani.md` (11 entit)

---

## Shrnuti

| Kategorie | PP pocet | RP pocet | Celkem |
|-----------|----------|----------|--------|
| Atributy v DS bez sloupce v DDL | 4 | 8 | 12 |
| Sloupce v DDL bez popisu v DS | 8 | 20 | 28 |
| Typove nesrovnalosti | 9 | 6 | 15 |
| Potencialni chyby v DDL | 2 | 2 | 4 |

---

## PP -- Nesrovnalosti PasPort

### PP-1. Atributy v DS bez sloupce v DDL

| Entita | Atribut (DS) | Datovy typ (DS) | Zavaznost | Poznamka |
|--------|-------------|----------------|-----------|----------|
| Typ nadoby | Provozovna | Reference -> Provozovna | Vysoka | DS definuje FK na organization_unit, DDL tabulka `container_type` ji nema. Pravdepodobne reseno pres tenant_id nebo aplikacne. |
| Typ nadoby | Oblast pouziti | Retezec(15) | Stredni | DS popisuje atribut pro rozliseni oblasti (RP, Pasport). V DDL chybi. Pravdepodobne reseno synchronizaci mimo DB. |
| Frekvence vyvozu | Provozovna (jako int FK) | Reference -> Provozovna | Stredni | DS definuje referenci (FK int), DDL ma misto ni `external_organization_unit_id` jako nvarchar(255) -- volna vazba pres externi identifikator. Viz take PP-3 typove nesrovnalosti. |
| Druh odpadu | Vazba na Skupinu odpadu | Reference | Nizka | DS nema explicitni vazbu druh -> skupina odpadu. Mapovani pravdepodobne reseno na aplikacni urovni. (Pozn.: neni primo "chybi v DDL", ale chybi v DS i DDL.) |

### PP-2. Sloupce v DDL bez popisu v DS

| Tabulka | Sloupec (DDL) | SQL typ | Zavaznost | Poznamka |
|---------|--------------|---------|-----------|----------|
| order_item_revision | waste_disposal_day_id | int FK | Vysoka | Primo FK na waste_disposal_day, ale DS resi dny svozu pres vazebni tabulku (#13). Redundantni vazba -- pravdepodobne starsi 1:1 mechanismus vedle novejsiho M:N. |
| site | name_search | nvarchar(255) | Nizka | Technicky vyhledavaci sloupec (generovany). |
| site | geom | geography | Nizka | Doplnuje lat/lng pro prostorove dotazy. Implementacni rozhodnuti. |
| container | evidence_number_search | nvarchar(255) | Nizka | Technicky vyhledavaci sloupec (generovany). |
| address | full_address | nvarchar(255) | Nizka | Generovany/konsolidovany sloupec pro celou adresu. |
| *(vsechny entity)* | id | int IDENTITY | Nizka | Interni auto-increment PK -- DS jej explicitne neuvadi u vetsiny entit (krome Stanoviste). Standardni implementacni vzor. |
| garbage_type | last_modification_source_id | int NOT NULL | Nizka | V DDL je NOT NULL, ale u jinych entit byva NULL. Nekonzistence v ramci DDL. |
| waste_disposal_frequency | external_organization_unit_id | nvarchar(255) | Stredni | Textovy externi identifikator provozovny misto FK. Viz PP-1 (Frekvence vyvozu). |

### PP-3. Typove nesrovnalosti

| Entita/Tabulka | Atribut/Sloupec | Typ v DS | Typ v DDL | Zavaznost | Poznamka |
|----------------|----------------|----------|----------|-----------|----------|
| RPO / order_item_revision | Pocet nadob / container_count | Cislo (nezaporne, 2 des. mista) | float | Stredni | DS pozaduje presnost na 2 des. mista; `float` je nepresny typ. `decimal(10,2)` by byl vhodnejsi. |
| Stanoviste / site | Souradnice | Point (1 atribut) | lat (float) + lng (float) + geom (geography) | Nizka | DS: jeden Point; DDL: 3 sloupce. Validni implementacni rozhodnuti, ale odchylka od logickeho modelu. |
| Nadoba / container | Evidencni cislo / evidence_number | Retezec(20) | nvarchar(255) | Nizka | DDL je sirsi (255 vs 20). |
| Nadoba / container | Vyrazena ke dni / discard_to_date | Datum | datetimeoffset | Nizka | DS: pouhy Datum; DDL: datetimeoffset (s casem a offset). DDL je sirsi. |
| Skupina odpadu / garbage_group | Nazev / name | Retezec(50) | nvarchar(255) | Nizka | DDL je sirsi (255 vs 50). |
| Skupina odpadu / garbage_group | Zkraceny nazev / short_name | Retezec(20) | nvarchar(255) | Nizka | DDL je sirsi (255 vs 20). |
| Typ nadoby / container_type | Nazev / name | Retezec(50) | nvarchar(255) | Nizka | DDL je sirsi (255 vs 50). |
| Adresa / address | Mesto / city | Retezec(80) | nvarchar(50) | Vysoka | **DDL je UZSI nez DS!** Hrozi orezani nazvu mest delsich nez 50 znaku. |
| Adresa / address | Ext. ID v registru adres / address_register_external_id | Retezec(10) | nvarchar(255) | Nizka | DDL je sirsi (255 vs 10). |

### PP-4. Potencialni chyby v DDL

| Tabulka | Sloupec | Problem | Zavaznost | Doporuceni |
|---------|---------|---------|-----------|------------|
| address | updated_by | **Typ `bit` misto `int` FK na user(id)** | Kriticka | Vsechny ostatni entity pouzivaji `int FK -> user(id)` pro updated_by. Zde je `bit`, coz neumoznuje ukladat referenci na uzivatele. Pravdepodobny bug v DDL -- opravit na `int NULL FK -> user(id)`. |
| address | city | **nvarchar(50) -- uzsi nez DS (80)** | Vysoka | DS pozaduje max 80 znaku. DDL ma pouze 50. U nazvu mest (napr. slovenske slozene nazvy) muze dojit k orezani. Rozsirit na nvarchar(80). |

---

## RP -- Nesrovnalosti RoadPlan

### RP-1. Atributy v DS bez sloupce v DDL

| Entita | Atribut (DS) | Datovy typ (DS) | Zavaznost | Poznamka |
|--------|-------------|----------------|-----------|----------|
| Denni vykon | Delka provozni doby | Cele cislo | Vysoka | V DDL tabulce `day_performance` neexistuje odpovidajici sloupec. Pravdepodobne odvozeno z nastaveni DV nebo ulozeno jinde. |
| Denni vykon | Zpusob ziskani trajektorie | Enumerace | Stredni | V DDL nenalezeno. |
| Denni vykon | Casove vyuziti | Desetinne cislo | Stredni | V DDL nenalezeno na urovni `day_performance`. Pravdepodobne na urovni `day_performance_realization` (percentage_usage). |
| Denni vykon | Podil km s privesem | Desetinne cislo | Stredni | V DDL nenalezeno na urovni `day_performance`. Pravdepodobne na urovni `day_performance_realization`. |
| Denni vykon | Pocet objednanych sluzeb | Cele cislo | Nizka | V DDL nenalezeno. Pravdepodobne odvozeno runtime (COUNT pres FK). |
| Typ dopravy | Technicka specifikace | Retezec(MAX) -- JSON | Vysoka | V DDL tabulce `transport_type` CHYBI sloupec pro JSON specifikaci technickeho nastaveni. Pro CS kriticky -- definuje generovane lokace a dostupne typy ukonu. |
| Misto realizace | Vychozi (Boolean) | Boolean | Nizka | V DDL reseno jako `df_execution_site_id` na tabulce `customer` (inverzni pristup). |
| Druh odpadu likv. mista | Datum vytvoreni | Datum a cas | Stredni | V DDL tabulce `liquidation_site_accepted_garbage_type` chybi `created_at`. |

> **Poznamka:** Nekterym atributum z DS odpovida reseni pres jinou tabulku nebo opacnou FK (napr. Lokace objednane sluzby -> ordered_service_location, Polozka DV -> day_performance_item). Tyto pripady jsou validni a nejsou nesrovnalostmi.

### RP-2. Sloupce v DDL bez popisu v DS

| Tabulka | Sloupec (DDL) | SQL typ | Zavaznost | Poznamka |
|---------|--------------|---------|-----------|----------|
| ordered_service | container_id | int FK | Stredni | FK -> order_item_container. V DS implicitne zahrnuto pres "Objednany ukon" -> "Objednana nadoba". |
| ordered_service | temporary_storage_ordered_service_id | int FK | Stredni | Odkaz na OS docasneho uskladneni. V DS nespecifikovan jako samostatny atribut. |
| ordered_service | is_according_to_plan | bit | Stredni | Priznak, zda realizace probehla dle planu. V DS neni. |
| ordered_service | confirmed_by | int FK | Stredni | Uzivatel, ktery potvrdil realizaci. V DS neni. |
| ordered_service | confirmation_datetime | datetimeoffset | Stredni | Cas potvrzeni realizace. V DS neni. |
| ordered_service | updated_at | datetimeoffset | Nizka | Systemovy sloupec. |
| ordered_service_location | valid | bit | Stredni | Platnost zaznamu. V DS nespecifikovano. |
| ordered_service_location | is_generated | bit | Stredni | Priznak generovani systemem. |
| ordered_service_location | created_at | datetimeoffset | Nizka | Systemovy sloupec. |
| ordered_service_location | updated_at | datetimeoffset | Nizka | Systemovy sloupec. |
| day_performance | route_geom | geometry | Nizka | Geometrie trasy. DS uvadi trajektorii na urovni lokaci. |
| day_performance | need_reroute | bit | Stredni | Priznak potreby pretrasovani. V DS neni. |
| day_performance | created_at, updated_at | datetimeoffset | Nizka | Systemove sloupce. |
| day_performance_item | duration | int | Stredni | Doba trvani polozky. V DS nespecifikovana na teto urovni. |
| day_performance_item | note | nvarchar(255) | Nizka | Poznamka k polozce. V DS neni. |
| vehicle | identification, winyx_id, device_type_id, color, color_inherited, cost_center_code, cost_center_id, group_id, driver_id, has_gps, vin, sort_id | ruzne | Nizka | DDL je vyrazne bohatsi nez DS. Vetsinou jde o legacy (WinyX), provozni nebo GPS atributy. |
| liquidation_site | winyx_id_old, winyx_id | int / nvarchar(50) | Nizka | Legacy ID stareho systemu WinyX. |
| liquidation_site_accepted_garbage_type | winyx_id | int | Nizka | Legacy ID WinyX. |
| execution_site | winyx_id_old, winyx_id | int / nvarchar(50) | Nizka | Legacy ID WinyX. |
| transport_type | name_search | varchar(500) | Nizka | Pomocny vyhledavaci sloupec. DS ho zminuje v poznamce. |

### RP-3. Typove nesrovnalosti

| Entita/Tabulka | Atribut/Sloupec | Typ v DS | Typ v DDL | Zavaznost | Poznamka |
|----------------|----------------|----------|----------|-----------|----------|
| Objednana sluzba / ordered_service | Cas realizace od/do (time_from/time_to) | Cas | varchar(150) | Stredni | Netypicky dlouhy varchar(150) pro casovou hodnotu. Pravdepodobne JSON/textovy format. |
| Objednana sluzba / ordered_service | Zpusob rozdeleni (parent/child_ordered_service_split_type) | Enumerace | nvarchar(MAX) | Nizka | DS uvadi enumeraci, DDL pouziva textovou hodnotu bez FK. |
| Lokace OS / ordered_service_location | Vzdalenost do dalsi lokace (next_location_distance) | Kladne cele cislo | float | Stredni | DS: cele cislo (metry), DDL: float. Poznamka: float muze byt potrebny pro presnejsi hodnoty. |
| Lokace OS / ordered_service_location | Doba jizdy do dalsi lokace (next_location_time) | Kladne cele cislo | float | Stredni | DS: cele cislo (sekundy), DDL: float. Stejna situace jako u vzdalenosti. |
| Lokace OS / ordered_service_location | Druhy odpadu (liquidation_garbage_types) | Reference [] -> Druh odpadu | nvarchar(MAX) | Vysoka | DS: kolekce referenci na entity Druh odpadu. DDL: JSON string. Ztrata referencni integrity. |
| Denni vykon / day_performance | Textova identifikace vozidla (vehicle_identification) | Retezec(50) | nvarchar(MAX) | Nizka | DDL umoznuje vyrazne delsi hodnotu nez DS predpoklada. |

> **Dalsi rozsahove odchylky (DDL sirsi nez DS, nizka zavaznost):**
> - `vehicle.name`: DS = 50, DDL = nvarchar(100)
> - `day_performance.color`: DS = 7, DDL = nvarchar(50)
> - `vehicle.flww_id`: DS Objekt = Retezec(50), DDL = int (typova zmena -- DS Vozidlo uvadi Cislo, odchylka v ramci DS)

### RP-4. Potencialni chyby v DDL

| Tabulka | Sloupec | Problem | Zavaznost | Doporuceni |
|---------|---------|---------|-----------|------------|
| ordered_service | time_from / time_to | **varchar(150) pro casovou hodnotu** | Stredni | Neobvykle dlouhy typ. Pokud uklada jednoduchy cas (HH:MM), staci nvarchar(5) jako na jinych tabulkach (napr. ordered_service_location.time_window_start). Overit format a zvazit standardizaci. |
| execution_site | organization_unit_id | **Chybi FK constraint** | Stredni | Sloupec existuje (int), ale nema definovany FK na organization_unit(id). Na jinych tabulkach (napr. liquidation_site) FK existuje. Overit, zda jde o zamerne vynechani. |

---

## Prehled chybejicich FK constraintu

Nasledujici sloupce v DDL maji charakter referencnich vazeb, ale nemaji definovany FK constraint:

| System | Tabulka | Sloupec | Ocekavana FK | Poznamka |
|--------|---------|---------|--------------|----------|
| RP | execution_site | organization_unit_id | organization_unit(id) | Sloupec existuje, FK chybi |
| RP | liquidation_site | geocoding_state | geocoding_state tabulka | Pouziva int, ale bez FK |
| RP | vehicle | object_type | (nezname) | DS uvadi referenci, DDL nema FK |
| RP | execution_site | geocoding_state | geocoding_state tabulka | Pouziva int, ale bez FK |

---

## Zaver a doporuceni

### Kriticke nalezy (vyzadujici akci)

1. **PP -- address.updated_by: typ `bit` misto `int FK`** -- Toto je s nejvyssi pravdepodobnosti bug v DDL. Vsechny ostatni entity pouzivaji `int NULL FK -> user(id)` pro sloupec updated_by. Hodnota `bit` (0/1) neumoznuje ukladat referenci na uzivatele, ktery provedl posledni zmenu. **Doporuceni:** Opravit datovy typ na `int NULL` s FK na user(id).

2. **PP -- address.city: nvarchar(50) vs DS max 80** -- DDL je uzsi nez logicky model. U slovenskych slozitejsich nazvu mest muze dojit k orezani. **Doporuceni:** Rozsirit na nvarchar(80) v souladu s DS.

3. **RP -- liquidation_garbage_types: JSON misto referenci** -- Na tabulce `ordered_service_location` jsou druhy odpadu ulozeny jako JSON string (nvarchar(MAX)) misto kolekce referenci. To znamena ztratu referencni integrity na urovni DB. **Doporuceni:** Pro CS projekt zohlednit pri navrhu integrace -- validaci druhu odpadu je nutne delat na aplikacni urovni.

### Vysoka priorita (dulezite pro CS projekt)

4. **RP -- transport_type: chybi Technicka specifikace** -- DS popisuje JSON atribut s technickymi parametry typu dopravy (generovane lokace, dostupne typy ukonu). V DDL sloupec chybi. Pro CS bude nutne bud rozsirit tabulku, nebo ukladat specifikaci mimo DB.

5. **RP -- day_performance: chybi Delka provozni doby** -- DS definuje atribut, DDL ho nema. Pro CS planovani okruhu je tato hodnota dulezita.

6. **PP -- container_type: chybi Provozovna** -- DS definuje vazbu na Provozovnu, DDL ji nema. Relevantni pro multi-provozovne prostredi.

7. **PP -- order_item_revision.waste_disposal_day_id** -- Redundantni primo FK vedle M:N vazebni tabulky. Muze vest k nekonzistenci dat.

### Stredni priorita (dokumentovat, sledovat)

8. **PP -- container_count: float misto decimal** -- Nepresny datovy typ pro financne/objednavkove ucely. Zvazit migraci na decimal(10,2).

9. **RP -- time_from/time_to na ordered_service: varchar(150)** -- Neobvykle dlouhy typ pro cas. Zvazit standardizaci na nvarchar(5) jako u jinych casovych sloupcu.

10. **RP -- execution_site.organization_unit_id: chybi FK** -- Sloupec existuje, ale nema constraint. Overit, zda je zamerne.

### Systemove vzory (informativni, neni potreba akce)

- **`id` sloupce:** DDL pridava interní PK (int IDENTITY) ke vsem entitam -- DS jej explicitne neuvadi (krome Stanoviste). Standardni implementacni vzor.
- **Vyhledavaci sloupce:** `name_search`, `evidence_number_search`, `full_address` -- technicke sloupce pro optimalizaci vyhledavani.
- **Legacy sloupce:** `winyx_id`, `winyx_id_old` v RP -- pozustatky migrace ze stareho systemu WinyX.
- **Rozsahove rozdily (DDL sirsi):** Ve vetsine pripadu DDL pouziva sirsi rozsah nez DS (napr. nvarchar(255) vs Retezec(50)). To je bezpecne -- sirsi typ nepovede ke ztrate dat.
