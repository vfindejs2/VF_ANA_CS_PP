# Analýza: Cílový koncept CS vs. současný stav datových modelů PP a RP

**Datum:** 2026-02-23
**Zdroje:**
- Příloha č. 3 — Cílový koncept Cyklické svozy MP SK (08/2025)
- DDL_PP_DB.txt — aktuální DDL databáze PasPort_MPGSK
- DDL_RP_DB.txt — aktuální DDL databáze RoadPlan_MPGSK

---

## A) PASPORT (PP) — Požadavky z Cílového konceptu s dopadem na datový model

### A1. Okruh (Circuit) — NOVÁ entita

**Požadavek:**
Okruh sdružuje položky objednávek (RPO), které jsou obsluhovány společně. Nepovinná vazba z RPO na okruh. V Etapě 1 je zdrojem HEN, v Etapě 2 bude PP zdrojem pravdy. Okruh bude mít vazbu na Provozovnu. V UI se uživatel bude moci přiřadit konkrétní vozidlo obsluhující daný okruh.

**Dopad na model:**
- NOVÁ tabulka `circuit` (nebo `okruh`) s atributy:
  - `id`, `external_id` (vazba na HEN), `name`, `description`
  - `organization_unit_id` (FK na provozovnu)
  - `vehicle_id` (FK na preferované vozidlo — Etapa 1)
  - `is_active`, `tenant_id`, audit sloupce (`created_at`, `updated_at`, `created_by`, `updated_by`)
  - `creation_source_id`, `last_modification_source_id`
- NOVÁ vazební tabulka `order_item_revision_circuit_assignment` (M:N vazba RPO na okruh):
  - `id`, `order_item_revision_id`, `circuit_id`
  - `valid_from`, `valid_to` (nepovinné — platnost zařazení)
  - `is_active`, `tenant_id`, audit sloupce

**Současný stav DDL:**
Tabulky `circuit` ani žádná varianta v PP DDL **neexistují**. Zcela nová entita.

---

### A2. Rozvrh (Schedule) — NOVÁ entita

**Požadavek:**
Rozvrh předepisuje konkrétní kalendářní dny obsluhy. Zdrojem pravdy je HEN (i do budoucna). PP bude rozvrhy zobrazovat pro kontrolu. RPO bude mít nepovinnou vazbu na rozvrh (M:N — jedna RPO může mít více vazeb na rozvrhy). Rozvrh má platnost (typicky na kalendářní rok). Každá provozovna může mít svou sadu rozvrhů.

**Dopad na model:**
- NOVÁ tabulka `schedule` (nebo `rozvrh`):
  - `id`, `external_id` (vazba na HEN), `name`, `description`
  - `organization_unit_id` (FK na provozovnu)
  - `valid_from`, `valid_to` (platnost rozvrhu)
  - `is_active`, `tenant_id`, audit sloupce
- NOVÁ tabulka `schedule_calendar_day` (kalendářní dny rozvrhu):
  - `id`, `schedule_id` (FK), `calendar_date` (date)
  - `is_active`, audit sloupce
- NOVÁ vazební tabulka `order_item_revision_schedule_assignment` (M:N vazba RPO na rozvrh):
  - `id`, `order_item_revision_id`, `schedule_id`
  - `valid_from`, `valid_to` (nepovinné)
  - `is_active`, `tenant_id`, audit sloupce

**Současný stav DDL:**
V PP DDL **neexistují** žádné tabulky pro rozvrhy. Existuje tabulka `waste_disposal_day` a `waste_disposal_frequency`, které řeší den a frekvenci svozu na úrovni nádob, ale rozvrh jako entita s kalendáři neexistuje. Vazební tabulka `waste_disposal_day_order_item_revision_assignment` existuje, ale je to jiná granularita (den svozu, ne rozvrh).

---

### A3. Zóna (Zone) — NOVÁ entita

**Požadavek:**
Zóna reprezentuje geografickou oblast. Nepovinná vazba z RPO na zónu je 1:1 (na rozdíl od okruhu a rozvrhu, kde je M:N). Zdrojem v Etapě 1 je HEN. Zóna má vazbu na provozovnu.

**Dopad na model:**
- NOVÁ tabulka `zone` (nebo `zona`):
  - `id`, `external_id`, `name`, `description`
  - `organization_unit_id` (FK na provozovnu)
  - `is_active`, `tenant_id`, audit sloupce
- Rozšíření tabulky `order_item_revision` o sloupec `zone_id` (FK, nullable) — vazba 1:1

**Současný stav DDL:**
Tabulka `zone` ani žádná varianta v PP DDL **neexistuje**. V tabulce `order_item_revision` sloupec `zone_id` **neexistuje**.

---

### A4. Skupina odpadu (Garbage Group) na RPO

**Požadavek:**
Nová vazba mezi druhem odpadu (`garbage_type`) a skupinou odpadu (`garbage_group`). Skupina odpadu bude na RPO určena procedurou PP dle mapování druh -> skupina. Speciální skupina "MIX" pro kombinovaný svoz. Skupina je per provozovna. Číselník bude společný pro RP a PP.

**Dopad na model:**
- Rozšíření tabulky `garbage_type` o sloupec `garbage_group_id` (FK na `garbage_group`, nullable) — mapování druh -> skupina
- Rozšíření tabulky `order_item_revision` o sloupec `garbage_group_id` (FK na `garbage_group`, nullable)
- Tabulka `garbage_group` je rozšířena o `organization_unit_id` (per provozovna)

**Současný stav DDL:**
- Tabulka `garbage_group` **existuje** (id, name, description, short_name, is_active, tenant_id, external_id, garbage_group_color_id, audit sloupce). **CHYBÍ** ale sloupec `organization_unit_id` (per provozovna).
- Tabulka `garbage_type` **existuje** (id, external_id, code, description, category, is_active, tenant_id, audit sloupce). **CHYBÍ** sloupec `garbage_group_id` (vazba druh -> skupina).
- Tabulka `order_item_revision` **existuje**, ale **NEMÁ** sloupec `garbage_group_id`.
- Tabulka `container` **má** sloupec `garbage_group_id` (FK na `garbage_group`) — to odpovídá existující funkčnosti.

---

### A5. Čas obsluhy typu nádoby

**Požadavek:**
Rozšířit číselník `container_type` o atribut "čas obsluhy typu nádoby" — číselná hodnota času ve vteřinách (průměrná doba obsluhy jedné nádoby). Použití pro výpočet doby trvání okruhu dne v RP.

**Dopad na model:**
- Rozšíření tabulky `container_type` o sloupec `service_time_seconds` (int, nullable) — čas obsluhy v sekundách

**Současný stav DDL:**
Tabulka `container_type` **existuje** (id, name, description, tenant_id, is_active, external_id, audit sloupce). Sloupec pro čas obsluhy **NEEXISTUJE**.

---

### A6. Souřadnice místa realizace RPO (geokódování)

**Požadavek:**
Pro zobrazení RPO v mapě je potřeba geokódovat místa realizace. Souřadnice MR budou použity jako fallback, pokud RPO nemá vazbu na nádobu se stanovištěm. Geokódování proběhne automaticky při změně adresy MR. V mapě se vizuálně odliší bod MR vs. bod stanoviště.

**Dopad na model:**
- Rozšíření tabulky `address` o sloupce `lat` (float, nullable) a `lng` (float, nullable) pro geokódované souřadnice
- Případně rozšíření `order_item_revision` o `lat`, `lng` pro cachované souřadnice MR
- Rozšíření `address` o `geocoding_state` (int, FK)

**Současný stav DDL:**
Tabulka `address` **existuje** (id, street, registry_number, orientation_number, city, postal_code, country, full_address, address_register_external_id, audit sloupce). **NEMÁ** sloupce `lat`, `lng` ani `geocoding_state`. Tabulka `site` (stanoviště) **MÁ** sloupce `lat`, `lng`, `geocoding_state` a `geom` — to je v pořádku, stanoviště mají souřadnice. Tabulka `code_geocoding_state` v PP **existuje**.

---

### A7. Stanoviště (Site) — existující entita, bez změn

**Požadavek:**
Stanoviště je klíčová existující entita. Souřadnice stanoviště se použijí pro zobrazení v mapě a pro lokalizaci objednaných služeb v RP.

**Dopad na model:**
Žádné přímé změny na tabulce `site`. Existující vazba `site_container_assignment` (container -> site) zůstává.

**Současný stav DDL:**
Tabulka `site` **existuje** s `lat`, `lng`, `geom`, `geocoding_state`, `address_id`, `organization_unit_id` — plně odpovídá.

---

### A8. Kontejner (Container) — změna řízení skupiny odpadu

**Požadavek:**
Skupina odpadu na nádobě bude nově řízena z RPO. Pokud nádoba má vazbu na RPO a RPO má definovanou skupinu odpadu, skupina na nádobě se přepíše. Skupinu odpadu na nádobě půjde zadat pouze pokud nádoba nemá vazbu na RPO nebo RPO nemá skupinu definovánu.

**Dopad na model:**
Žádné strukturální změny — sloupec `garbage_group_id` na tabulce `container` již existuje. Jde o změnu business logiky (procedury), nikoliv DDL.

**Současný stav DDL:**
Tabulka `container` **MÁ** `garbage_group_id` (FK na `garbage_group`) — odpovídá.

---

### A9. Vazba RPO na okruhy, rozvrhy, zóny — panel "Přiřazení okruhů"

**Požadavek:**
Nový panel na obrazovce RPO zobrazující zařazení do okruhů, rozvrhů a zón. V Etapě 1 jen zobrazení (read-only), v Etapě 2 editace. Platnost zařazení (valid_from/valid_to) bude nepovinná.

**Dopad na model:**
Pokryto vazebními tabulkami z bodů A1, A2, A3 výše. Žádné další DDL změny.

---

### A10. Rozšíření filtrů na obrazovce RPO

**Požadavek:**
Pokročilý filtr rozšířen o skupiny: Okruh, Rozvrh, Rozhodující kalendářní den, Aktuálně platné.

**Dopad na model:**
Žádné DDL změny — filtry pracují s daty z vazebních tabulek a kalendáře rozvrhu.

---

### A11. Váhové naplnění a objemové zaplnění per skupina odpadu (Etapa 2 / Strategická optimalizace)

**Požadavek:**
Pro strategické plánování je nutné znát:
- Měrnou hmotnost (hustotu) odpadu na 1 litr nádoby per skupina odpadu
- Průměrné objemové zaplnění typu nádoby per skupina odpadu
Tyto hodnoty budou evidovány v PP pro potřeby konfigurace optimalizace.

**Dopad na model:**
- NOVÁ tabulka `garbage_group_container_type_parameters` (nebo podobná):
  - `id`, `garbage_group_id` (FK), `container_type_id` (FK)
  - `weight_density_per_liter` (float) — měrná hmotnost
  - `volume_fill_percentage` (float) — objemové zaplnění
  - `organization_unit_id`, `tenant_id`, audit sloupce

**Současný stav DDL:**
Taková tabulka **NEEXISTUJE**. Jde o zcela novou strukturu pro Etapu 2+.

---

### Souhrn PP — přehled změn

| Entita / Tabulka | Stav | Typ změny |
|---|---|---|
| `circuit` | NEEXISTUJE | NOVÁ tabulka |
| `order_item_revision_circuit_assignment` | NEEXISTUJE | NOVÁ tabulka |
| `schedule` | NEEXISTUJE | NOVÁ tabulka |
| `schedule_calendar_day` | NEEXISTUJE | NOVÁ tabulka |
| `order_item_revision_schedule_assignment` | NEEXISTUJE | NOVÁ tabulka |
| `zone` | NEEXISTUJE | NOVÁ tabulka |
| `order_item_revision.zone_id` | NEEXISTUJE | NOVÝ sloupec |
| `order_item_revision.garbage_group_id` | NEEXISTUJE | NOVÝ sloupec |
| `garbage_type.garbage_group_id` | NEEXISTUJE | NOVÝ sloupec |
| `garbage_group.organization_unit_id` | NEEXISTUJE | NOVÝ sloupec |
| `container_type.service_time_seconds` | NEEXISTUJE | NOVÝ sloupec |
| `address.lat`, `address.lng`, `address.geocoding_state` | NEEXISTUJÍ | NOVÉ sloupce |
| `garbage_group_container_type_parameters` | NEEXISTUJE | NOVÁ tabulka (Etapa 2) |
| `container.garbage_group_id` | EXISTUJE | Bez DDL změn, změna logiky |
| `site` (stanoviště) | EXISTUJE s lat/lng | Bez změn |

---

## B) ROADPLAN (RP) — Požadavky z Cílového konceptu s dopadem na datový model

### B1. Okruh dne (Day Circuit / Circuit of the Day) — NOVÁ entita

**Požadavek:**
Okruh dne je soubor objednaných služeb, které v rámci denního výkonu vystupují společně. Vzniká při generování objednaných služeb. Je to základní stavební prvek DV pro CS. Typ položky denního výkonu (`day_performance_item_type`) bude rozšířen o hodnotu "okruh dne". Okruh dne má:
- Název (kombinace názvu okruhu + datum)
- Stav (při realizaci nastavovaný z FOB)
- Poznámka
- Vazbu na likvidační místo(a)
- Vazbu na skupinu odpadu (jeden okruh dne = jedna skupina odpadu)
- Vozidlo (pokud okruh má přiřazené preferované vozidlo)

**Dopad na model:**
- NOVÁ tabulka `day_circuit` (nebo `circuit_of_day`):
  - `id`, `name`, `date` (date — den realizace)
  - `circuit_id` (FK — vazba na okruh z PP, nullable)
  - `garbage_group_id` (FK — skupina odpadu)
  - `organization_unit_id` (FK)
  - `status` (int — stav realizace)
  - `vehicle_id` (FK, nullable)
  - `day_performance_id` (FK, nullable — přiřazení do DV)
  - `note` (nvarchar)
  - audit sloupce
- NOVÁ tabulka `day_circuit_liquidation_site` (M:N vazba okruh dne -> likvidační místa):
  - `id`, `day_circuit_id` (FK), `liquidation_site_id` (FK)
  - `order` (int), `is_included_in_dv` (bit — zda se přenáší do FOB)
- Rozšíření tabulky `day_performance_item_type` o nový kód "CIRCUIT_OF_DAY"
- Rozšíření tabulky `day_performance_item`:
  - `day_circuit_id` (FK, nullable) — odkaz na okruh dne

**Současný stav DDL:**
- Tabulka `day_circuit` **NEEXISTUJE**.
- Tabulka `day_performance_item_type` **existuje** (id, code). Aktuální hodnoty neznáme, ale typ "CIRCUIT_OF_DAY" pravděpodobně chybí.
- Tabulka `day_performance_item` **existuje** (id, order, day_performance_id, duration, has_trailer, ordered_service_id, ordered_service_location_id, day_performance_item_type, note). Sloupec `day_circuit_id` **NEEXISTUJE**.

---

### B2. Denní výkon (Day Performance) — rozšíření

**Požadavek:**
DV pro CS bude mít nový transport_type. DV bude pracovat s okruhy dne místo jednotlivých OS. Výpočet plánovaného času: TDV = suma(TOD) + suma(TBP) + suma(TS), kde TOD = suma(TOS), TOS = TN * počet nádob RPO. Nová záložka v plánování DV, monitoring, časové využití vozidel.

**Dopad na model:**
- Rozšíření číselníku `transport_type` (pokud existuje jako tabulka) o nový typ pro CS
- Tabulka `day_performance` samotná nevyžaduje nové sloupce — stávající struktura (status, organization_unit_id, transport_type, vehicle_id, realization_date, start_time, route_duration, route_distance, note) pokrývá potřeby. Hodnota `transport_type` bude rozšířena.

**Současný stav DDL:**
Tabulka `day_performance` **existuje** se sloupci: id, status, organization_unit_id, transport_type (int), vehicle_id, realization_date, start_time, start/finish_day_performance_item_id, route_duration, route_distance, route_geom, note, has_trailer, need_reroute, is_confirmation_printed, is_created_by_user, vehicle_identification, realization_type, color. Struktura je **dostatečná**, stačí přidat hodnotu do číselníku transport_type.

---

### B3. Objednaná služba (Ordered Service) — rozšíření pro CS

**Požadavek:**
OS pro CS vzniká generováním z RPO. Pro CS eviduje minimální sadu atributů: číslo RPO, konečný příjemce, adresa a souřadnice MR, počet nádob, typ nádoby, druh a skupina odpadu, okruh, rozvrh, kalendářní dny svozu. OS bude mít vazbu na okruh dne.

**Dopad na model:**
- Rozšíření tabulky `ordered_service` o sloupce:
  - `order_item_revision_id` (int, FK — vazba na RPO z PP, nullable)
  - `day_circuit_id` (int, FK — vazba na okruh dne, nullable)
  - `circuit_id` (int, FK — vazba na okruh/circuit z PP, nullable)
  - `schedule_id` (int, FK — vazba na rozvrh, nullable)
  - `garbage_group_id` (int, FK — skupina odpadu, nullable)
  - `garbage_type_id` (int, FK — druh odpadu, nullable)
  - `container_type_id` (int, FK — typ nádoby, nullable)
  - `container_count` (int — počet nádob)
  - `transport_type` (nvarchar — typ dopravy, pro rozlišení CS vs. kontejnerová)

**Současný stav DDL:**
Tabulka `ordered_service` **existuje** a je primárně navržena pro kontejnerovou dopravu. Obsahuje: id, status, container_operation_id, container_id, container_number, date_from, date_to, note, time_from, time_to, parent/child_ordered_service_id, type, day_performance_realization_id, is_realized, realization_order, confirmed_by, reschedule*, on_demand, ride_identifier, vehicle_evidence_number, default_realization_date.

**CHYBÍ:**
- `order_item_revision_id` (vazba na RPO)
- `day_circuit_id` (vazba na okruh dne)
- `circuit_id`, `schedule_id` (vazby na okruh a rozvrh)
- `garbage_group_id`, `garbage_type_id` (odpady)
- `container_type_id`, `container_count` (typ a počet nádob)

Pozn.: Datový model pro OS CS "bude postaven tak, aby v nezbytné míře podporoval potřebnou funkčnost" — tzn. může být buď rozšíření stávající tabulky, nebo separátní tabulka pro CS OS. Detailní analýza rozhodne.

---

### B4. Lokace objednané služby (Ordered Service Location) — rozšíření pro CS

**Požadavek:**
Lokace OS pro CS bude odpovídat stanovišti nádob nebo souřadnicím MR. Každá lokace bude mít min: příslušnost k OS, souřadnice, typ nádoby, četnost, skupina odpadu, počet nádob.

**Dopad na model:**
- Rozšíření tabulky `ordered_service_location` o sloupce:
  - `site_id` (int, FK — vazba na stanoviště z PP, nullable)
  - `container_type_id` (int, FK, nullable)
  - `garbage_group_id` (int, FK, nullable)
  - `container_count` (int, nullable)
  - `frequency_description` (nvarchar, nullable)
  - `status` (int — stav odbavení: neobslouženo, FOB odbaveno, RP potvrzeno)

**Současný stav DDL:**
Tabulka `ordered_service_location` **existuje** (id, order, action, type, address_id, lat, lng, liquidation_site_id, execute, organization_unit_id, valid, next_location_distance/time, route_to_next_location, ordered_service_id, note, time_window_start/end, is_time_window_met, duration, is_generated, manipulation_time_in_minutes, is_monitored, execution_site_id, manual_address_change, has_flat_rate, liquidation_garbage_types, loading_code).

**CHYBÍ:** `site_id`, `container_type_id`, `garbage_group_id`, `container_count`, `status` (stav odbavení pro CS).

---

### B5. Likvidační místo — vazba na skupinu odpadu

**Požadavek:**
Likvidační místo bude mít vazbu na skupinu odpadu (pro okruh dne).

**Dopad na model:**
- Tabulka `liquidation_site_accepted_garbage_type` **existuje** a řeší vazbu LM -> druh odpadu.
- Potřeba doplnit tabulku `liquidation_site` o sloupec `garbage_group_id` (FK, nullable), nebo rozšířit existující vazební tabulku `liquidation_site_accepted_garbage_type` o `garbage_group_id`.

**Současný stav DDL:**
Tabulka `liquidation_site` **existuje** (id, name, address_id, lat, lng, contact, note, organization_unit_id, active, owned_liquidation_site, geocoding_state, external_id). **NEMÁ** sloupec `garbage_group_id`.

---

### B6. Vozidlo — nové atributy pro strategické plánování

**Požadavek:**
Pro strategickou optimalizaci jsou potřeba nové atributy vozidla:
- Objem nástavby (kapacita)
- Nosnost (váhová kapacita)
- Pracovní doba (po kterou je vozidlo k dispozici)
- "Použít pro SP" (příznak, zda vozidlo vstupuje do strategického plánování)

**Dopad na model:**
- Rozšíření tabulky `vehicle` o sloupce:
  - `body_volume` (float, nullable) — objem nástavby v litrech/m3
  - `load_capacity` (float, nullable) — nosnost v kg
  - `working_hours_from` (nvarchar(5), nullable)
  - `working_hours_to` (nvarchar(5), nullable)
  - `use_for_strategic_planning` (bit, nullable)

**Současný stav DDL:**
Tabulka `vehicle` **existuje** (id, name, registration_plate, organization_unit_id, flww_id, identification, color, cost_center_code, group_id, vehicle_type_id, evidence_number, vin, sort_id, realization_type, date_insertion, date_rejection). **NEMÁ** žádný ze zmíněných sloupců.

---

### B7. Konfigurace generování OS

**Požadavek:**
Konfigurace RP pro automatické generování OS:
- Frekvence a čas spuštění
- Období vzniku OS (kolik dnů dopředu)
- Volba vzniku fiktivního okruhu dne (ANO/NE)

**Dopad na model:**
- Nové záznamy v tabulce `default_settings` (nebo nová konfigurační tabulka per provozovna)
- Případně NOVÁ tabulka `cs_generation_config`:
  - `id`, `organization_unit_id`
  - `run_frequency_cron` (nvarchar)
  - `generation_period_days` (int)
  - `create_fictive_circuit` (bit)
  - audit sloupce

**Současný stav DDL:**
Tabulka `default_settings` **existuje** (name, val). Pro per-provozovna konfiguraci by byla potřeba nová tabulka.

---

### B8. Konfigurace strategické optimalizace (Etapa 2)

**Požadavek:**
Parametry pro optimalizační algoritmus:
- Kritéria a omezení okruhu SP (max čas, max váha, max objem)
- Hlavní optimalizační kritérium (čas)
- Distanční matice a dopravní síť

**Dopad na model:**
- NOVÁ tabulka `strategic_planning_version`:
  - `id`, `organization_unit_id`, `name`, `status`, `created_at`, audit sloupce
- NOVÁ tabulka `strategic_planning_config`:
  - `id`, `strategic_planning_version_id`
  - `max_route_duration_minutes`, `max_route_weight_kg`, `max_route_volume_liters`
  - `optimization_criterion` (nvarchar)
  - parametry vozidel, dep, LM
- NOVÁ tabulka `strategic_ordered_service` (SOS):
  - obdoba `ordered_service`, ale pro strategické plánování
- NOVÁ tabulka `strategic_day_circuit` (SOD):
  - obdoba `day_circuit`, ale pro strategické plánování

**Současný stav DDL:**
Žádná z těchto tabulek **NEEXISTUJE**. Zcela nová doména pro Etapu 2.

---

### B9. Skupina odpadu v RP — sdílený číselník

**Požadavek:**
Číselník skupin odpadu bude společný pro RP a PP. V RP se bude používat pro filtraci OS, okruhů dne. Nastavení vazby druh -> skupina bude možné jen na straně PP, RP bude čerpat.

**Dopad na model:**
- NOVÁ tabulka `garbage_group` v RP (synchronizovaná z PP):
  - `id`, `external_id` (id z PP), `name`, `short_name`, `description`
  - `organization_unit_id`, `is_active`, audit sloupce
- Rozšíření tabulky `garbage_type` v RP o `garbage_group_id` (FK)

**Současný stav DDL:**
V RP DDL **NEEXISTUJE** tabulka `garbage_group`. Tabulka `garbage_type` v RP **existuje** (code, name, description, id, active, category, external_id), ale **NEMÁ** sloupec `garbage_group_id`.

---

### B10. Okruh (Circuit) v RP — synchronizovaný z PP

**Požadavek:**
RP potřebuje pracovat s okruhy pro generování objednaných služeb a okruhů dne.

**Dopad na model:**
- NOVÁ tabulka `circuit` v RP (synchronizovaná z PP):
  - `id`, `external_id`, `name`, `organization_unit_id`, `vehicle_id` (preferované vozidlo)
  - `is_active`, audit sloupce

**Současný stav DDL:**
Tabulka `circuit` v RP **NEEXISTUJE**.

---

### B11. Rozvrh (Schedule) v RP — synchronizovaný z PP

**Požadavek:**
RP potřebuje rozvrhy a jejich kalendáře pro generování OS.

**Dopad na model:**
- NOVÁ tabulka `schedule` v RP
- NOVÁ tabulka `schedule_calendar_day` v RP

**Současný stav DDL:**
Tabulky v RP **NEEXISTUJÍ**.

---

### B12. Váhy a objemy na okruhu dne (pro strategické plánování)

**Požadavek:**
Řidič zadává přes FOB hmotnost naveženou na LM a objemové využití. Hodnoty se ukládají k okruhu dne v RP. Dispečer je může editovat.

**Dopad na model:**
- NOVÁ tabulka `day_circuit_weight_record`:
  - `id`, `day_circuit_id` (FK)
  - `weight_kg` (float) — navážená hmotnost
  - `volume_utilization` (float, nullable) — objemové využití (např. 0.25, 0.5, 0.75, 1.0)
  - `recorded_at` (datetimeoffset)
  - `source` (nvarchar — FOB / manual)
  - audit sloupce

**Současný stav DDL:**
Taková tabulka **NEEXISTUJE**.

---

### B13. Typ nádoby v RP — rozšíření

**Požadavek:**
Čas obsluhy typu nádoby bude synchronizován z PP do RP.

**Dopad na model:**
- Rozšíření tabulky `container_type` v RP o `service_time_seconds` (int, nullable)

**Současný stav DDL:**
Tabulka `container_type` v RP **existuje** (id, name, description, transport_type_id, active, organization_unit_id, external_id). Sloupec `service_time_seconds` **NEEXISTUJE**.

---

### Souhrn RP — přehled změn

| Entita / Tabulka | Stav | Typ změny | Etapa |
|---|---|---|---|
| `day_circuit` | NEEXISTUJE | NOVÁ tabulka | 1 |
| `day_circuit_liquidation_site` | NEEXISTUJE | NOVÁ tabulka | 1 |
| `day_performance_item.day_circuit_id` | NEEXISTUJE | NOVÝ sloupec | 1 |
| `day_performance_item_type` — nový kód | EXISTUJE | NOVÝ záznam | 1 |
| `ordered_service` — nové sloupce CS | EXISTUJE | ROZŠÍŘENÍ tabulky (6+ sloupců) | 1 |
| `ordered_service_location` — nové sloupce CS | EXISTUJE | ROZŠÍŘENÍ tabulky (5+ sloupců) | 1 |
| `garbage_group` | NEEXISTUJE | NOVÁ tabulka | 1 |
| `garbage_type.garbage_group_id` | NEEXISTUJE | NOVÝ sloupec | 1 |
| `circuit` | NEEXISTUJE | NOVÁ tabulka | 1 |
| `schedule` + `schedule_calendar_day` | NEEXISTUJE | NOVÉ tabulky | 1 |
| `container_type.service_time_seconds` | NEEXISTUJE | NOVÝ sloupec | 1 |
| `cs_generation_config` | NEEXISTUJE | NOVÁ tabulka | 1 |
| `vehicle` — nové sloupce SP | EXISTUJE | ROZŠÍŘENÍ (5 sloupců) | 2 |
| `liquidation_site.garbage_group_id` | NEEXISTUJE | NOVÝ sloupec | 1 |
| `day_circuit_weight_record` | NEEXISTUJE | NOVÁ tabulka | 2 |
| `strategic_planning_*` tabulky | NEEXISTUJE | NOVÉ tabulky (4+) | 2 |

---

## C) Integrační dopady na datový model (PP <-> RP)

### C1. Nové integrační entity (PP -> RP)

Následující entity budou přenášeny z PP do RP:
1. **RPO** — elementární údaje (číslo, konečný příjemce, adresa MR, souřadnice, počet nádob, typ nádoby, druh a skupina odpadu, okruh, rozvrh, kalendáře)
2. **Nádoby a stanoviště** — základní údaje (pro lokalizaci OS)
3. **Okruhy** — číselník + vazby na RPO
4. **Rozvrhy** — číselník + kalendářní dny + vazby na RPO
5. **Skupiny odpadu** — číselník + vazba druh -> skupina
6. **Čas obsluhy na typu nádoby**

### C2. Nové integrační entity (RP -> HEN)

1. **Hlavička DV** (trasa dne) — pro realizační doklady
2. **Seznam RPO v DV** (předměty smluv) — pro realizační doklady

### C3. Nové integrační entity (HEN -> PP)

1. **Okruhy** — číselník (Etapa 1)
2. **Rozvrhy** — číselník + kalendáře (Etapa 1)
3. **Zóny** — číselník (Etapa 1)
4. **Vazby předmět smlouvy -> okruh, rozvrh, zóna**

---

## D) Klíčová rizika z pohledu datového modelu

1. **Rozhodnutí o separátní vs. rozšířené tabulce `ordered_service` pro CS** — Cílový koncept naznačuje, že datový model RP pro OS CS "bude navržen v rámci detailní analýzy". Rozšíření stávající tabulky (přidání nullable sloupců) vs. nová tabulka — zásadní architektonické rozhodnutí.

2. **Sdílení číselníků mezi PP a RP** — `garbage_group`, `circuit`, `schedule` musí být konzistentní. Otázka synchronizace a master systému.

3. **Objem dat** — Denní výkon pro CS může mít až 600 OS / 1000 stanovišť. Dopad na indexy, výkon dotazů a velikost tabulek `ordered_service_location`.

4. **Platnost zařazení RPO do okruhu/rozvrhu** — Nepovinná platnost (valid_from/valid_to) komplikuje dotazy a synchronizaci. Je třeba definovat výchozí chování.

5. **Geokódování adres v PP** — Nutnost doplnit lat/lng na PP adresu, protože PP adresy aktuálně nemají souřadnice (na rozdíl od RP, kde adresy souřadnice mají).
