# Revize pozadavku: Cilovy koncept CS -- dvouvrstvovy format

**Datum:** 2026-02-23
**Zdroje:**
- Priloha c. 3 -- Cilovy koncept Cyklicke svozy MP SK (08/2025)
- DDL_PP_DB.txt -- aktualni DDL databaze PasPort_MPGSK
- DDL_RP_DB.txt -- aktualni DDL databaze RoadPlan_MPGSK
- DS-PP: datovy slovnik PasPortu (technicky-projekt-PP/datove-modely)
- DS-RP: datovy slovnik RoadPlanu (technicky-projekt-RP/datove-modely)
- Extrakce PP entit a mapovani DS-PP na DDL-PP
- Extrakce RP entit a mapovani DS-RP na DDL-RP

---

## A) PASPORT (PP) -- Pozadavky A1-A11

---

### A1. Okruh (Circuit) -- NOVA entita

**Pozadavek:**
Okruh sdruzuje polozky objednavek (RPO), ktere jsou obsluhovany spolecne. Nepovinna vazba z RPO na okruh. V Etape 1 je zdrojem HEN, v Etape 2 bude PP zdrojem pravdy. Okruh bude mit vazbu na Provozovnu. V UI se uzivatel bude moci priradit konkretni vozidlo obsluhujici dany okruh.

**Dopad na entitni model (datovy slovnik):**
- Nova entita: **Okruh** (circuit) -- dnes v DS-PP neexistuje
- Nove atributy entity Okruh:
  - `Externi identifikator` -- Retezec(255), nepovinna; vazba na HEN
  - `Nazev` -- Retezec(255), povinna; nazev okruhu
  - `Popis` -- Retezec(255), nepovinna
  - `Provozovna` -- Reference na Provozovnu, povinna
  - `Vozidlo` -- Reference na Vozidlo, nepovinna; preferovane vozidlo (Etapa 1)
  - `Je aktivni`, audit atributy, `Tenant` -- standardni systemove atributy
- Nova asociacni entita: **Prirazeni RPO k okruhu** (M:N vazba)
  - `Revize polozky objednavky` -- Reference na RPO, povinna
  - `Okruh` -- Reference na Okruh, povinna
  - `Platnost od` / `Platnost do` -- Datum, nepovinna (nepovinne casove ohraniceni prirazeni)
  - Systemove atributy
- Zmena entity **Revize polozky objednavky (RPO)**: doplnena nova asociace "Okruh" (M:N pres vazebni entitu)
  - Dnes DS-PP definuje RPO s 18 atributy (viz extrakce #1), zadny z nich neodkazuje na okruh

**Dopad na fyzicky model (DDL):**
- NOVA tabulka `circuit`:
  - `id` int IDENTITY(1,1) NOT NULL PK
  - `external_id` nvarchar(255) NULL
  - `name` nvarchar(255) NOT NULL
  - `description` nvarchar(255) NULL
  - `organization_unit_id` int NOT NULL FK -> organization_unit(id)
  - `vehicle_id` int NULL FK -> vehicle(id)
  - `is_active` bit NOT NULL DEFAULT 1
  - `tenant_id` int NOT NULL FK -> tenant(id)
  - `created_at`, `updated_at`, `created_by`, `updated_by`, `creation_source_id`, `last_modification_source_id` -- standardni audit sloupce
- NOVA vazebni tabulka `order_item_revision_circuit_assignment`:
  - `id` int IDENTITY(1,1) NOT NULL PK
  - `order_item_revision_id` int NOT NULL FK -> order_item_revision(id)
  - `circuit_id` int NOT NULL FK -> circuit(id)
  - `valid_from` date NULL
  - `valid_to` date NULL
  - `is_active` bit NOT NULL DEFAULT 1
  - `tenant_id` int NOT NULL FK -> tenant(id)
  - Audit sloupce
- Doporucene indexy: IX_circuit_organization_unit_id, IX_oirca_order_item_revision_id, IX_oirca_circuit_id

**Soucasny stav:**
- Entitni model (DS): Entita "Okruh" v DS-PP **neexistuje**. RPO (extrakce #1) nema zadny atribut odkazujici na okruh.
- Fyzicky model (DDL): Tabulka `circuit` v PP DDL **neexistuje**. Zadna varianta nazvu.
- Delta: Zcela nova entita + nova vazebni tabulka. Na strane RPO je treba doplnit logiku asociace (pres vazebni tabulku, bez primych FK sloupcu).

---

### A2. Rozvrh (Schedule) -- NOVA entita

**Pozadavek:**
Rozvrh predpisuje konkretni kalendarni dny obsluhy. Zdrojem pravdy je HEN (i do budoucna). PP bude rozvrhy zobrazovat pro kontrolu. RPO bude mit nepovinnou vazbu na rozvrh (M:N -- jedna RPO muze mit vice vazeb na rozvrhy). Rozvrh ma platnost (typicky na kalendarni rok). Kazda provozovna muze mit svou sadu rozvrhu.

**Dopad na entitni model (datovy slovnik):**
- Nova entita: **Rozvrh** (schedule) -- dnes v DS-PP neexistuje
  - `Externi identifikator` -- Retezec(255), nepovinna; vazba na HEN
  - `Nazev` -- Retezec(255), povinna
  - `Popis` -- Retezec(255), nepovinna
  - `Provozovna` -- Reference na Provozovnu, povinna
  - `Platnost od` -- Datum, povinna; zacatek platnosti rozvrhu
  - `Platnost do` -- Datum, nepovinna; konec platnosti rozvrhu
  - Systemove atributy
- Nova entita: **Kalendarni den rozvrhu** (schedule_calendar_day)
  - `Rozvrh` -- Reference na Rozvrh, povinna
  - `Kalendarni datum` -- Datum, povinny
  - Systemove atributy
- Nova asociacni entita: **Prirazeni RPO k rozvrhu** (M:N vazba)
  - `Revize polozky objednavky` -- Reference na RPO, povinna
  - `Rozvrh` -- Reference na Rozvrh, povinny
  - `Platnost od` / `Platnost do` -- Datum, nepovinna
  - Systemove atributy
- Souvislost s existujicimi entitami:
  - DS-PP dnes definuje entity **Zakladni dny svozu** (waste_disposal_day) a **Frekvence vyvozu** (waste_disposal_frequency) -- viz extrakce #9 a #13. Tyto entity resi frekvenci a dny svozu, ale NE kalendarni rozvrh. Rozvrh je nova, vyssi uroven planovani.
  - Vazebni entita **Prirazeni zakladnich dnu svozu k RPO** (#13) je existujici M:N mechanismus pro dny svozu. Novy rozvrh jej doplnuje (nenahrazuje).

**Dopad na fyzicky model (DDL):**
- NOVA tabulka `schedule`:
  - `id` int IDENTITY(1,1) NOT NULL PK
  - `external_id` nvarchar(255) NULL
  - `name` nvarchar(255) NOT NULL
  - `description` nvarchar(255) NULL
  - `organization_unit_id` int NOT NULL FK -> organization_unit(id)
  - `valid_from` date NOT NULL
  - `valid_to` date NULL
  - `is_active` bit NOT NULL DEFAULT 1
  - `tenant_id` int NOT NULL FK -> tenant(id)
  - Audit sloupce
- NOVA tabulka `schedule_calendar_day`:
  - `id` int IDENTITY(1,1) NOT NULL PK
  - `schedule_id` int NOT NULL FK -> schedule(id)
  - `calendar_date` date NOT NULL
  - `is_active` bit NOT NULL DEFAULT 1
  - Audit sloupce
- NOVA vazebni tabulka `order_item_revision_schedule_assignment`:
  - `id` int IDENTITY(1,1) NOT NULL PK
  - `order_item_revision_id` int NOT NULL FK -> order_item_revision(id)
  - `schedule_id` int NOT NULL FK -> schedule(id)
  - `valid_from` date NULL
  - `valid_to` date NULL
  - `is_active` bit NOT NULL DEFAULT 1
  - `tenant_id` int NOT NULL FK -> tenant(id)
  - Audit sloupce

**Soucasny stav:**
- Entitni model (DS): Entita "Rozvrh" v DS-PP **neexistuje**. DS definuje entity **Zakladni dny svozu** (atributy: ext. ID, Dny svozu retezec(50), Pocet svozu za mesic, Provozovna) a **Frekvence vyvozu** (atributy: ext. ID, Nazev(50), Popis(80), Skupina(80), Provozovna). Tyto entity resi logiku "ktere dny v tydnu" a "kolikrat za obdobi", ale neobsahuji konkretni kalendarni dny.
- Fyzicky model (DDL): Existuji tabulky `waste_disposal_day` a `waste_disposal_frequency`, existuje vazebni tabulka `waste_disposal_day_order_item_revision_assignment`. Tabulky `schedule`, `schedule_calendar_day` **neexistuji**.
- Delta: Zcela nove 3 tabulky. Existujici mechanismus dnu svozu zustava, rozvrh ho doplnuje o konkretni kalendar.

---

### A3. Zona (Zone) -- NOVA entita

**Pozadavek:**
Zona reprezentuje geografickou oblast. Nepovinna vazba z RPO na zonu je 1:1 (na rozdil od okruhu a rozvrhu, kde je M:N). Zdrojem v Etape 1 je HEN. Zona ma vazbu na provozovnu.

**Dopad na entitni model (datovy slovnik):**
- Nova entita: **Zona** (zone) -- dnes v DS-PP neexistuje
  - `Externi identifikator` -- Retezec(255), nepovinna; vazba na HEN
  - `Nazev` -- Retezec(255), povinna
  - `Popis` -- Retezec(255), nepovinna
  - `Provozovna` -- Reference na Provozovnu, povinna
  - Systemove atributy
- Zmena entity **Revize polozky objednavky (RPO)**: novy atribut
  - `Zona` -- Reference na Zonu, nepovinna; vazba 1:1 (jedna RPO = max jedna zona)
  - Dnes DS-PP RPO (extrakce #1) nema zadny atribut odkazujici na zonu

**Dopad na fyzicky model (DDL):**
- NOVA tabulka `zone`:
  - `id` int IDENTITY(1,1) NOT NULL PK
  - `external_id` nvarchar(255) NULL
  - `name` nvarchar(255) NOT NULL
  - `description` nvarchar(255) NULL
  - `organization_unit_id` int NOT NULL FK -> organization_unit(id)
  - `is_active` bit NOT NULL DEFAULT 1
  - `tenant_id` int NOT NULL FK -> tenant(id)
  - Audit sloupce
- ALTER TABLE `order_item_revision`: novy sloupec
  - `zone_id` int NULL FK -> zone(id)

**Soucasny stav:**
- Entitni model (DS): Entita "Zona" v DS-PP **neexistuje**. RPO (extrakce #1, 18 atributu) nema zadnou asociaci na zonu.
- Fyzicky model (DDL): Tabulka `zone` **neexistuje**. Sloupec `zone_id` na `order_item_revision` **neexistuje**. Aktualne `order_item_revision` ma 21 sloupcu (viz extrakce #1 DDL) -- zadny z nich neodkazuje na zonu.
- Delta: Nova tabulka + novy sloupec (FK) na existujici tabulce. Migrace: `zone_id` bude NULL pro vsechna existujici data.

---

### A4. Skupina odpadu (Garbage Group) na RPO

**Pozadavek:**
Nova vazba mezi druhem odpadu (`garbage_type`) a skupinou odpadu (`garbage_group`). Skupina odpadu bude na RPO urcena procedurou PP dle mapovani druh -> skupina. Specialni skupina "MIX" pro kombinovany svoz. Skupina je per provozovna. Ciselnik bude spolecny pro RP a PP.

**Dopad na entitni model (datovy slovnik):**
- Zmena entity **Druh odpadu** (extrakce #7): novy atribut
  - `Skupina odpadu` -- Reference na Skupinu odpadu, nepovinna; mapovani druh -> skupina
  - Dnes DS-PP Druh odpadu ma atributy: ext. ID, Kod(15), Popis(255), Kategorie(4), systemove. **Nema vazbu na skupinu odpadu.** (Poznamka z extrakce #7: "DS nema explicitni vazbu druh odpadu na skupinu odpadu.")
- Zmena entity **Revize polozky objednavky (RPO)** (extrakce #1): novy atribut
  - `Skupina odpadu` -- Reference na Skupinu odpadu, nepovinna; odvozena z druhu odpadu procedurou
  - Dnes RPO nema atribut skupiny odpadu
- Zmena entity **Skupina odpadu** (extrakce #6): novy atribut
  - `Provozovna` -- Reference na Provozovnu, nepovinna; per-provozovna segmentace
  - Dnes DS-PP Skupina odpadu ma: ext. ID, Zkraceny nazev(20), Nazev(50), Popis(255), Barva, systemove. **Nema vazbu na Provozovnu.**
- Business pravidla:
  - Skupina odpadu na RPO je vypoctena automaticky dle mapovani druh -> skupina
  - Specialni skupina "MIX" pro kombinovany svoz
  - Ciselnik spolecny pro PP a RP

**Dopad na fyzicky model (DDL):**
- ALTER TABLE `garbage_type`: novy sloupec
  - `garbage_group_id` int NULL FK -> garbage_group(id)
- ALTER TABLE `order_item_revision`: novy sloupec
  - `garbage_group_id` int NULL FK -> garbage_group(id)
- ALTER TABLE `garbage_group`: novy sloupec
  - `organization_unit_id` int NULL FK -> organization_unit(id)

**Soucasny stav:**
- Entitni model (DS):
  - **Skupina odpadu** (extrakce #6): existuje s atributy ext. ID, Zkraceny nazev(20), Nazev(50), Popis(255), Barva skupiny odpadu, systemove. **CHYBI** atribut Provozovna.
  - **Druh odpadu** (extrakce #7): existuje s atributy ext. ID, Kod(15), Popis(255), Kategorie(4), systemove. **CHYBI** vazba na Skupinu odpadu.
  - **RPO** (extrakce #1): nema atribut skupiny odpadu.
- Fyzicky model (DDL):
  - `garbage_group`: existuje (id, name(255), description(255), short_name(255), is_active, tenant_id, external_id, garbage_group_color_id, audit). **CHYBI** `organization_unit_id`.
  - `garbage_type`: existuje (id, external_id, code(15), description(255), category(4), is_active, tenant_id, audit). **CHYBI** `garbage_group_id`.
  - `order_item_revision`: existuje (21 sloupcu). **CHYBI** `garbage_group_id`.
  - `container`: **MA** sloupec `garbage_group_id` (FK) -- to je existujici funkcnost pro nadoby.
- Delta:
  - DS i DDL: chybi vazba druh -> skupina (3 nove sloupce na 3 tabulkach)
  - Entita Skupina odpadu: v DS chybi Provozovna, v DDL chybi `organization_unit_id`
  - Na tabulce `container` uz `garbage_group_id` existuje -- bude treba sladit logiku ridici skupinu z RPO vs. z nadoby

---

### A5. Cas obsluhy typu nadoby

**Pozadavek:**
Rozsirit ciselnik `container_type` o atribut "cas obsluhy typu nadoby" -- ciselna hodnota casu ve vterinych (prumerna doba obsluhy jedne nadoby). Pouziti pro vypocet doby trvani okruhu dne v RP.

**Dopad na entitni model (datovy slovnik):**
- Zmena entity **Typ nadoby** (extrakce #8): novy atribut
  - `Cas obsluhy` -- Cislo (kladne, cele), nepovinna; cas obsluhy v sekundach
  - Dnes DS-PP Typ nadoby ma: ext. ID, Nazev(50), Popis(255), Provozovna, Oblast pouziti(15), systemove. **Nema atribut casu obsluhy.**
- Poznamka: DS-PP jiz dnes definuje atributy `Provozovna` a `Oblast pouziti`, ktere v DDL chybi (viz delta extrakce #8). Novy atribut `Cas obsluhy` bude tedy treti chybejici atribut.

**Dopad na fyzicky model (DDL):**
- ALTER TABLE `container_type`: novy sloupec
  - `service_time_seconds` int NULL

**Soucasny stav:**
- Entitni model (DS): **Typ nadoby** (extrakce #8) ma 12 atributu. Cas obsluhy **neexistuje**. Navic DS definuje `Provozovnu` a `Oblast pouziti`, ktere DDL take nema.
- Fyzicky model (DDL): `container_type` existuje s 12 sloupci (id, name(255), description(255), tenant_id, is_active, created_at, updated_at, external_id, created_by, updated_by, creation_source_id, last_modification_source_id). Sloupec `service_time_seconds` **neexistuje**. Taktez chybi `organization_unit_id` a sloupec pro oblast pouziti.
- Delta: 1 novy sloupec na existujici tabulce. Plus 2 drive identifikovane chybejici sloupce (org_unit, oblast pouziti).

---

### A6. Souradnice mista realizace RPO (geokodovani)

**Pozadavek:**
Pro zobrazeni RPO v mape je treba geokodovat mista realizace. Souradnice MR budou pouzity jako fallback, pokud RPO nema vazbu na nadobu se stanovistem. Geokodovani probehne automaticky pri zmene adresy MR. V mape se vizualne odlisi bod MR vs. bod stanoviste.

**Dopad na entitni model (datovy slovnik):**
- Zmena entity **Adresa** (extrakce #10): nove atributy
  - `Souradnice` -- Point (lat/lng), nepovinna; geokodovane souradnice adresy
  - `Stav geokodovani` -- Enumerace (Stav geokodovani), povinna; stav procesu geokodovani
  - Dnes DS-PP Adresa ma: ext. ID, Ext. ID v registru adres(10), Ulice(80), Cislo popisne(15), Cislo orientacni(15), Mesto(80), PSC(15), Stat(50), systemove. **NEMA** atribut Souradnice ani Stav geokodovani.
  - Pro srovnani: DS-PP **Stanoviste** (extrakce #4) **MA** atributy `Souradnice` (Point) a `Stav geokodovani` (Enumerace).
- Pripadne: zmena entity **RPO** o cachovane souradnice (pokud se rozhodne o cache)
  - `Souradnice MR` -- Point, nepovinna; cachovane souradnice mista realizace

**Dopad na fyzicky model (DDL):**
- ALTER TABLE `address`: nove sloupce
  - `lat` float NULL
  - `lng` float NULL
  - `geocoding_state` int NOT NULL FK -> code_geocoding_state(id) (s DEFAULT hodnotou)
- Pripadne ALTER TABLE `order_item_revision`:
  - `lat` float NULL, `lng` float NULL (cachovane souradnice)

**Soucasny stav:**
- Entitni model (DS):
  - **Adresa** (extrakce #10): ma 14 atributu. **NEMA** souradnice ani stav geokodovani. (Poznamka: DS definuje mesto s rozsahem 80, ale DDL ma jen 50 -- existujici delta.)
  - **Stanoviste** (extrakce #4): **MA** atributy Souradnice (Point) a Stav geokodovani (Enumerace) -- to je vzor pro implementaci na adrese.
- Fyzicky model (DDL):
  - `address`: existuje s 17 sloupci (id, street(80), registry_number(15), orientation_number(15), city(50), postal_code(15), country(50), full_address(255), address_register_external_id(255), audit). **NEMA** sloupce `lat`, `lng`, `geocoding_state`.
  - `site`: **MA** sloupce `lat`(float), `lng`(float), `geom`(geography), `geocoding_state`(int FK) -- stanoviste souradnice funguje spravne.
  - `code_geocoding_state`: ciselnik v PP **existuje** -- lze vyuzit.
  - Poznamka: `address.updated_by` je typ `bit` (misto `int` FK) -- existujici bug identifikovany v extrakci #10.
- Delta: 3 nove sloupce na tabulce `address`. Existujici implementace na `site` slouzi jako vzor. Ciselnik `code_geocoding_state` jiz existuje.

---

### A7. Stanoviste (Site) -- existujici entita, bez zmen

**Pozadavek:**
Stanoviste je klicova existujici entita. Souradnice stanoviste se pouziji pro zobrazeni v mape a pro lokalizaci objednanych sluzeb v RP.

**Dopad na entitni model (datovy slovnik):**
- Zadne zmeny na entite **Stanoviste** (extrakce #4).
- Existujici asociace `Prirazeni nadoby ke stanovisti` (extrakce #11) zustava.

**Dopad na fyzicky model (DDL):**
- Zadne DDL zmeny na tabulce `site` ani `site_container_assignment`.

**Soucasny stav:**
- Entitni model (DS): **Stanoviste** (extrakce #4) ma 17 atributu: Interni ID, Externi ID(255), Nazev(255), Provozovna, Adresa, Souradnice (Point), Stav geokodovani, Je sklad nadob, Poznamka(255), Vychozi fotografie, systemove.
- Fyzicky model (DDL): `site` existuje s 19 sloupci: id, name(255), organization_unit_id(FK), address_id(FK), lat(float), lng(float), note(255), is_active, audit, external_id, geom(geography), geocoding_state(FK), default_photo_id(FK), name_search, creation_source_id, last_modification_source_id, is_container_warehouse. **Plne odpovida** pozadavkum.
- Delta: Zadna. Stanoviste je pripravene pro CS. Mapovani DS <> DDL je kompletni (jedine navic v DDL: `name_search` a `geom` -- technicke sloupce).

---

### A8. Kontejner (Container) -- zmena rizeni skupiny odpadu

**Pozadavek:**
Skupina odpadu na nadobe bude nove rizena z RPO. Pokud nadoba ma vazbu na RPO a RPO ma definovanou skupinu odpadu, skupina na nadobe se prepise. Skupinu odpadu na nadobe pujde zadat pouze pokud nadoba nema vazbu na RPO nebo RPO nema skupinu definovanu.

**Dopad na entitni model (datovy slovnik):**
- Zadne strukturalni zmeny na entite **Nadoba** (extrakce #5).
- Zmena business logiky: atribut `Skupina odpadu` (Reference na Skupinu odpadu, nepovinna) na entite Nadoba bude nyne rizena kaskadove z RPO.
- Entita **Nadoba** (extrakce #5) jiz ma atribut `Skupina odpadu` v DS-PP.
- Dotcene entity pro business logiku:
  - **Prirazeni nadoby k polozce objednavky** (extrakce #12): existujici vazba nadoba <-> polozka objednavky
  - **RPO** (extrakce #1) -> nova `Skupina odpadu` (viz A4)

**Dopad na fyzicky model (DDL):**
- Zadne DDL zmeny. Sloupec `garbage_group_id` na tabulce `container` jiz existuje.
- Zmena je ciste na urovni business logiky / procedur.

**Soucasny stav:**
- Entitni model (DS): **Nadoba** (extrakce #5) ma 22 atributu vcetne `Skupina odpadu` (Reference na Skupinu odpadu, nepovinna). V DS je tato vazba jiz definovana.
- Fyzicky model (DDL): `container` ma 28 sloupcu vcetne `garbage_group_id` (int NULL FK -> garbage_group(id)). Sloupec **existuje a odpovida**.
- Delta: Zadna strukturalni. Zmena je v logice: dnes je `garbage_group_id` na nadobe nastaven rucne; nove bude kaskadove rizeny z RPO.

---

### A9. Vazba RPO na okruhy, rozvrhy, zony -- panel "Prirazeni okruhu"

**Pozadavek:**
Novy panel na obrazovce RPO zobrazujici zarazeni do okruhu, rozvrhu a zon. V Etape 1 jen zobrazeni (read-only), v Etape 2 editace. Platnost zarazeni (valid_from/valid_to) bude nepovinna.

**Dopad na entitni model (datovy slovnik):**
- Zadne nove entity ani atributy nad ramec toho, co je definovano v A1, A2, A3.
- Pozadavek je UI/prezentacni vrstva pracujici s daty z:
  - Prirazeni RPO k okruhu (A1)
  - Prirazeni RPO k rozvrhu (A2)
  - Zona na RPO (A3)

**Dopad na fyzicky model (DDL):**
- Zadne dalsi DDL zmeny. Pokryto vazebnimi tabulkami z A1 (`order_item_revision_circuit_assignment`), A2 (`order_item_revision_schedule_assignment`) a sloupcem z A3 (`order_item_revision.zone_id`).

**Soucasny stav:**
- Entitni model (DS): RPO (extrakce #1) dnes nema zadne asociace na okruh, rozvrh ani zonu.
- Fyzicky model (DDL): `order_item_revision` nema sloupce `zone_id`, vazebni tabulky pro okruh/rozvrh neexistuji.
- Delta: Kompletne pokryto pozadavky A1-A3. Tento pozadavek nepridava dalsi datove zmeny.

---

### A10. Rozsireni filtru na obrazovce RPO

**Pozadavek:**
Pokrocily filtr rozsiren o skupiny: Okruh, Rozvrh, Rozhodujici kalendarni den, Aktualne platne.

**Dopad na entitni model (datovy slovnik):**
- Zadne nove entity ani atributy. Filtry pracuji s existujicimi/novymi asociacemi:
  - Okruh: pres vazebni entitu Prirazeni RPO k okruhu (A1)
  - Rozvrh: pres vazebni entitu Prirazeni RPO k rozvrhu (A2)
  - Kalendarni den: pres entitu Kalendarni den rozvrhu (A2)
  - Platnost: pres atributy valid_from/valid_to na RPO a na vazebních entitach

**Dopad na fyzicky model (DDL):**
- Zadne DDL zmeny. Filtry pracuji s daty z vazebních tabulek a kalendare rozvrhu.
- Pripadne nove indexy pro vykon dotazu:
  - IX_schedule_calendar_day_calendar_date (pro filtr dle kalendarniho dne)
  - IX_oirca_valid_from_valid_to (pro filtr dle platnosti)

**Soucasny stav:**
- Entitni model (DS): Filtrovani je UI logika. DS nedefinuje filtry jako entity.
- Fyzicky model (DDL): Aktualne nelze filtrovat dle okruhu, rozvrhu ani kalendarnich dnu -- tabulky neexistuji.
- Delta: Zadne datove zmeny. Pozadavek je zavislý na A1, A2 a jejich datovych strukturach.

---

### A11. Vahove naplneni a objemove zaplneni per skupina odpadu (Etapa 2 / Strategicka optimalizace)

**Pozadavek:**
Pro strategicke planovani je nutne znat:
- Mernou hmotnost (hustotu) odpadu na 1 litr nadoby per skupina odpadu
- Prumerne objemove zaplneni typu nadoby per skupina odpadu
Tyto hodnoty budou evidovany v PP pro potreby konfigurace optimalizace.

**Dopad na entitni model (datovy slovnik):**
- Nova entita: **Parametry skupiny odpadu a typu nadoby** -- dnes v DS-PP neexistuje
  - `Skupina odpadu` -- Reference na Skupinu odpadu, povinna
  - `Typ nadoby` -- Reference na Typ nadoby, povinna
  - `Merna hmotnost na litr` -- Desetinne cislo, nepovinna; hustota odpadu
  - `Objemove zaplneni` -- Desetinne cislo, nepovinna; prumerne objemove zaplneni (0-1)
  - `Provozovna` -- Reference na Provozovnu, nepovinna; per-provozovna parametry
  - Systemove atributy
- Asociace pres existujici entity:
  - **Skupina odpadu** (extrakce #6): dnes ma atributy pro barvu a zakladni identifikaci, ale zadne kapacitni/fyzikalni parametry
  - **Typ nadoby** (extrakce #8): dnes ma nazev, popis a provozovnu (v DS), ale zadne kapacitni parametry

**Dopad na fyzicky model (DDL):**
- NOVA tabulka `garbage_group_container_type_parameters`:
  - `id` int IDENTITY(1,1) NOT NULL PK
  - `garbage_group_id` int NOT NULL FK -> garbage_group(id)
  - `container_type_id` int NOT NULL FK -> container_type(id)
  - `weight_density_per_liter` float NULL
  - `volume_fill_percentage` float NULL
  - `organization_unit_id` int NULL FK -> organization_unit(id)
  - `is_active` bit NOT NULL DEFAULT 1
  - `tenant_id` int NOT NULL FK -> tenant(id)
  - Audit sloupce
- UNIQUE constraint: (garbage_group_id, container_type_id, organization_unit_id)

**Soucasny stav:**
- Entitni model (DS): Takova entita **neexistuje**. DS-PP Skupina odpadu (extrakce #6) a Typ nadoby (extrakce #8) nemaji zadne atributy pro fyzikalni parametry.
- Fyzicky model (DDL): Tabulka **neexistuje**. Jde o zcela novou strukturu.
- Delta: Nova tabulka, nova entita. Etapa 2+.

---

### Souhrn PP -- prehled zmen (entitni + fyzicky model)

| Entita / Tabulka | DS dnes | DDL dnes | Typ zmeny | Etapa |
|---|---|---|---|---|
| **Okruh** (`circuit`) | Neexistuje | Neexistuje | NOVA entita + tabulka | 1 |
| **Prirazeni RPO k okruhu** (`order_item_revision_circuit_assignment`) | Neexistuje | Neexistuje | NOVA entita + tabulka | 1 |
| **Rozvrh** (`schedule`) | Neexistuje | Neexistuje | NOVA entita + tabulka | 1 |
| **Kalendarni den rozvrhu** (`schedule_calendar_day`) | Neexistuje | Neexistuje | NOVA entita + tabulka | 1 |
| **Prirazeni RPO k rozvrhu** (`order_item_revision_schedule_assignment`) | Neexistuje | Neexistuje | NOVA entita + tabulka | 1 |
| **Zona** (`zone`) | Neexistuje | Neexistuje | NOVA entita + tabulka | 1 |
| RPO . `zone_id` | Neexistuje v DS | Neexistuje v DDL | NOVY atribut + sloupec | 1 |
| RPO . `garbage_group_id` | Neexistuje v DS | Neexistuje v DDL | NOVY atribut + sloupec | 1 |
| Druh odpadu . `garbage_group_id` | Neexistuje v DS | Neexistuje v DDL | NOVY atribut + sloupec | 1 |
| Skupina odpadu . `organization_unit_id` | Neexistuje v DS | Neexistuje v DDL | NOVY atribut + sloupec | 1 |
| Typ nadoby . `service_time_seconds` | Neexistuje v DS | Neexistuje v DDL | NOVY atribut + sloupec | 1 |
| Adresa . `lat`, `lng`, `geocoding_state` | Neexistuji v DS | Neexistuji v DDL | NOVE atributy + sloupce | 1 |
| **Parametry sk. odp. a typu nadoby** (`garbage_group_container_type_parameters`) | Neexistuje | Neexistuje | NOVA entita + tabulka | 2 |
| Nadoba . `garbage_group_id` | Existuje v DS | Existuje v DDL | Bez zmen (zmena logiky) | 1 |
| Stanoviste (`site`) | Existuje v DS | Existuje v DDL s lat/lng | Bez zmen | - |

**Celkem PP:** 6 novych tabulek (Etapa 1) + 1 nova tabulka (Etapa 2), 6 novych sloupcu na existujicich tabulkach, 0 strukturalnich zmen (1x zmena business logiky).

---

## B) ROADPLAN (RP) -- Pozadavky B1-B13

---

### B1. Okruh dne (Day Circuit / Circuit of the Day) -- NOVA entita

**Pozadavek:**
Okruh dne je soubor objednanych sluzeb, ktere v ramci denniho vykonu vystupuji spolecne. Vznika pri generovani objednanych sluzeb. Je to zakladni stavebni prvek DV pro CS. Typ polozky denniho vykonu (`day_performance_item_type`) bude rozsiren o hodnotu "okruh dne". Okruh dne ma:
- Nazev (kombinace nazvu okruhu + datum)
- Stav (pri realizaci nastavovany z FOB)
- Poznamka
- Vazbu na likvidacni misto(a)
- Vazbu na skupinu odpadu (jeden okruh dne = jedna skupina odpadu)
- Vozidlo (pokud okruh ma prirazene preferovane vozidlo)

**Dopad na entitni model (datovy slovnik):**
- Nova entita: **Okruh dne** (day_circuit) -- dnes v DS-RP neexistuje
  - `Nazev` -- Retezec(255), povinna; kombinace nazvu okruhu + datum
  - `Datum` -- Datum, povinny; den realizace
  - `Okruh` -- Reference na Okruh (z PP, sync), nepovinna
  - `Skupina odpadu` -- Reference na Skupinu odpadu, povinna
  - `Provozovna` -- Reference na Provozovnu, povinna
  - `Stav` -- Enumerace, povinny; stav realizace
  - `Vozidlo` -- Reference na Vozidlo, nepovinna
  - `Denni vykon` -- Reference na Denni vykon, nepovinna; prirazeni do DV
  - `Poznamka` -- Retezec(255), nepovinna
  - Systemove atributy
- Nova asociacni entita: **Likvidacni misto okruhu dne** (M:N)
  - `Okruh dne` -- Reference na Okruh dne, povinna
  - `Likvidacni misto` -- Reference na Likvidacni misto, povinna
  - `Poradi` -- Cislo (kladne, cele), povinne
  - `Je zahrnuto do DV` -- Boolean; zda se prenasi do FOB
- Zmena entity **Typ polozky denniho vykonu** (extrakce RP #9):
  - Nova hodnota: "Okruh dne" (CIRCUIT_OF_DAY)
  - Dnes DS-RP definuje 4 hodnoty: Objednana sluzba, Lokace objednane sluzby, Casovy interval, Rozdeleni
- Zmena entity **Polozka denniho vykonu** (extrakce RP #4): novy atribut
  - `Okruh dne` -- Reference na Okruh dne, nepovinna
  - Dnes DS-RP Polozka DV ma 6 atributu: Poradi, Typ polozky, Denni vykon, Prives, Objednana sluzba, Lokace OS. **Nema** vazbu na okruh dne.

**Dopad na fyzicky model (DDL):**
- NOVA tabulka `day_circuit`:
  - `id` int IDENTITY(1,1) NOT NULL PK
  - `name` nvarchar(255) NOT NULL
  - `date` date NOT NULL
  - `circuit_id` int NULL (FK -- vazba na okruh z PP, sync)
  - `garbage_group_id` int NOT NULL FK -> garbage_group(id)
  - `organization_unit_id` int NOT NULL FK -> organization_unit(id)
  - `status` int NOT NULL
  - `vehicle_id` int NULL FK -> vehicle(id)
  - `day_performance_id` int NULL FK -> day_performance(id)
  - `note` nvarchar(255) NULL
  - Audit sloupce
- NOVA tabulka `day_circuit_liquidation_site`:
  - `id` int IDENTITY(1,1) NOT NULL PK
  - `day_circuit_id` int NOT NULL FK -> day_circuit(id)
  - `liquidation_site_id` int NOT NULL FK -> liquidation_site(id)
  - `order` int NOT NULL
  - `is_included_in_dv` bit NOT NULL
- INSERT do `day_performance_item_type`: novy zaznam s kodem "CIRCUIT_OF_DAY"
- ALTER TABLE `day_performance_item`: novy sloupec
  - `day_circuit_id` int NULL FK -> day_circuit(id)

**Soucasny stav:**
- Entitni model (DS):
  - **Okruh dne**: v DS-RP **neexistuje**.
  - **Typ polozky DV** (extrakce RP #9): DS definuje 4 hodnoty (Objednana sluzba, Lokace OS, Casovy interval, Rozdeleni). DDL ma tabulku `day_performance_item_type` (id, code) -- strukturalne OK, hodnota "okruh dne" chybi.
  - **Polozka DV** (extrakce RP #4): DS ma 6 atributu, DDL ma 8 sloupcu (navic duration, note). **Nema** sloupec `day_circuit_id`.
- Fyzicky model (DDL):
  - `day_circuit` **neexistuje**
  - `day_performance_item_type`: existuje (id, code) -- pripravena na novy zaznam
  - `day_performance_item`: existuje (id, order, day_performance_id, duration, has_trailer, ordered_service_id, ordered_service_location_id, day_performance_item_type, note). **CHYBI** `day_circuit_id`.
- Delta: 2 nove tabulky, 1 novy sloupec na existujici tabulce, 1 novy zaznam v ciselniku.

---

### B2. Denni vykon (Day Performance) -- rozsireni

**Pozadavek:**
DV pro CS bude mit novy transport_type. DV bude pracovat s okruhy dne misto jednotlivych OS. Vypocet planovaneho casu: TDV = suma(TOD) + suma(TBP) + suma(TS), kde TOD = suma(TOS), TOS = TN * pocet nadob RPO. Nova zalozka v planovani DV, monitoring, casove vyuziti vozidel.

**Dopad na entitni model (datovy slovnik):**
- Zmena entity **Denni vykon** (extrakce RP #3):
  - Atribut `Typ dopravy` (existujici Reference na Typ dopravy) -- bude pouzit novy typ pro CS
  - Zadne nove atributy na entite DV -- stávajici struktura pokryva potreby
  - Dnes DS-RP Denni vykon ma 27 atributu (viz extrakce RP #3)
- Zmena entity **Typ dopravy** (extrakce RP #8): nova hodnota
  - Novy zaznam: "Cyklicke svozy" (nebo podobny nazev)
  - Dnes DS-RP definuje 6 hodnot: Hakovy nakladac, Ramenovy nakladac, Lisovaci vozidlo, Valnik, Komunalni vozidlo -- pilot, Cisterna

**Dopad na fyzicky model (DDL):**
- INSERT do `transport_type`: novy zaznam pro CS
  - `id` = (novy), `name` = 'Cyklicke svozy', `is_available` = 1
- Zadne ALTER TABLE zmeny na `day_performance` -- stavajici struktura (20 sloupcu) je dostatecna.

**Soucasny stav:**
- Entitni model (DS): **Denni vykon** (extrakce RP #3) ma 27 atributu. Nektera z nich v DDL chybi (Delka provozni doby, Casove vyuziti, Podil km s privesem, Pocet OS -- viz delta extrakce RP #3). Atribut `Typ dopravy` existuje a je Reference na entitu Typ dopravy.
- Fyzicky model (DDL): `day_performance` existuje s 20 sloupci. Sloupec `transport_type` (int NOT NULL FK -> transport_type(id)) existuje. Tabulka `transport_type` existuje s (id, name(50), name_search(500), is_available). **Poznamka:** DS definuje i atribut `Technicka specifikace` (JSON), ktery v DDL `transport_type` **chybi** -- je treba vyresit pro CS.
- Delta: 1 novy zaznam v ciselniku. Zadne strukturalni zmeny na tabulce DV. Existujici delty DS vs. DDL (viz extrakce RP #3) se CS netykaji primo, ale mohou ovlivnit business logiku.

---

### B3. Objednana sluzba (Ordered Service) -- rozsireni pro CS

**Pozadavek:**
OS pro CS vznika generovanim z RPO. Pro CS eviduje minimalni sadu atributu: cislo RPO, konecny prijemce, adresa a souradnice MR, pocet nadob, typ nadoby, druh a skupina odpadu, okruh, rozvrh, kalendarni dny svozu. OS bude mit vazbu na okruh dne.

**Dopad na entitni model (datovy slovnik):**
- Zmena entity **Objednana sluzba** (extrakce RP #1): nove atributy pro CS
  - `Revize polozky objednavky` -- Reference na RPO (z PP), nepovinna; vazba na zdrojovou RPO
  - `Okruh dne` -- Reference na Okruh dne, nepovinna; prirazeni do okruhu dne
  - `Okruh` -- Reference na Okruh (z PP, sync), nepovinna
  - `Rozvrh` -- Reference na Rozvrh (z PP, sync), nepovinna
  - `Skupina odpadu` -- Reference na Skupinu odpadu, nepovinna
  - `Druh odpadu` -- Reference na Druh odpadu, nepovinna
  - `Typ nadoby` -- Reference na Typ nadoby, nepovinna
  - `Pocet nadob` -- Cislo (kladne, cele), nepovinna
  - `Typ dopravy` -- Retezec, nepovinna; pro rozliseni CS vs. kontejnerova
  - Dnes DS-RP Objednana sluzba ma 31 atributu (viz extrakce RP #1). Vsechny jsou navrzene pro kontejnerovou dopravu. Zadny z nich nepokryva atributy specificke pro CS.
- **Architektonicke rozhodnuti:** Cilovy koncept naznacuje, ze datovy model pro OS CS "bude postaven tak, aby v nezbytne mire podporoval potrebnou funkcnost". Moznosti: (a) rozsireni stavajici tabulky o nullable sloupce, (b) nova separatni tabulka pro CS OS.

**Dopad na fyzicky model (DDL):**
- Varianta A (rozsireni): ALTER TABLE `ordered_service`:
  - `order_item_revision_id` int NULL (FK na RPO z PP)
  - `day_circuit_id` int NULL FK -> day_circuit(id)
  - `circuit_id` int NULL (FK na okruh)
  - `schedule_id` int NULL (FK na rozvrh)
  - `garbage_group_id` int NULL FK -> garbage_group(id)
  - `garbage_type_id` int NULL FK -> garbage_type(id)
  - `container_type_id` int NULL FK -> container_type(id)
  - `container_count` int NULL
  - `transport_type` nvarchar(200) NULL
- Varianta B (nova tabulka): NOVA tabulka `cs_ordered_service` s vlastni strukturou

**Soucasny stav:**
- Entitni model (DS): **Objednana sluzba** (extrakce RP #1) ma 31 atributu orientovanych na kontejnerovou dopravu:
  - Klicove existujici: Stav (Enumerace), Objednany ukon (Reference), Konkretni objednana nadoba (cele cislo), Datum realizace od/do, Cas realizace od/do, Poznamka, Identifikator jizdy, Evidencni cislo vychoziho vozidla, atd.
  - Vsechny vazby jsou na kontejnerovy model (Objednany ukon -> order_item_container_operation).
  - **CHYBI** vsechny CS-specificke atributy.
- Fyzicky model (DDL): `ordered_service` ma 29 sloupcu:
  - Klicove: status(int FK), container_operation_id(FK), container_id(FK), container_number(int), date_from, date_to, note, time_from(varchar(150)), time_to(varchar(150)), parent/child_ordered_service_id, type(FK), day_performance_realization_id(FK), is_realized, realization_order, on_demand, ride_identifier, vehicle_evidence_number, default_realization_date, atd.
  - **CHYBI:** order_item_revision_id, day_circuit_id, circuit_id, schedule_id, garbage_group_id, garbage_type_id, container_type_id, container_count
- Delta: 8-9 novych sloupcu (varianta A) nebo cela nova tabulka (varianta B). Architektonicke rozhodnuti je kriticke.

---

### B4. Lokace objednane sluzby (Ordered Service Location) -- rozsireni pro CS

**Pozadavek:**
Lokace OS pro CS bude odpovídat stanovisti nadob nebo souradnicim MR. Kazda lokace bude mit min: prislusnost k OS, souradnice, typ nadoby, cetnost, skupina odpadu, pocet nadob.

**Dopad na entitni model (datovy slovnik):**
- Zmena entity **Lokace objednane sluzby** (extrakce RP #2): nove atributy pro CS
  - `Stanoviste` -- Reference na Stanoviste (z PP), nepovinna; vazba na PP stanoviste
  - `Typ nadoby` -- Reference na Typ nadoby, nepovinna
  - `Skupina odpadu` -- Reference na Skupinu odpadu, nepovinna
  - `Pocet nadob` -- Cislo (kladne, cele), nepovinna
  - `Popis frekvence` -- Retezec(255), nepovinna
  - `Stav odbavenni` -- Enumerace, nepovinna; neobslouzeno / FOB odbaveno / RP potvrzeno
  - Dnes DS-RP Lokace OS ma 25 atributu (viz extrakce RP #2). Zadny z nich nepokryva CS-specificke atributy (stanoviste, typ nadoby, skupina odpadu, pocet nadob, stav odbaveni).

**Dopad na fyzicky model (DDL):**
- ALTER TABLE `ordered_service_location`: nove sloupce
  - `site_id` int NULL (FK na site z PP)
  - `container_type_id` int NULL FK -> container_type(id)
  - `garbage_group_id` int NULL FK -> garbage_group(id)
  - `container_count` int NULL
  - `frequency_description` nvarchar(255) NULL
  - `status` int NULL (stav odbaveni)

**Soucasny stav:**
- Entitni model (DS): **Lokace OS** (extrakce RP #2) ma 25 atributu:
  - Klicove existujici: Objednana sluzba, Poradi, Akce (Enumerace), Typ lokace (Enumerace), Adresa, Souradnice (Point), Likvidacni misto, Provozovna, Doba manipulace, Doba trvani, Provest, Casove okno (start/end), Vzdalenost/Cas do dalsi lokace, Poznamka, Monitoring, Fakturace, Druhy odpadu.
  - **CHYBI** stanoviste, typ nadoby, skupina odpadu, pocet nadob, stav odbaveni.
- Fyzicky model (DDL): `ordered_service_location` ma 26 sloupcu:
  - Klicove: id, order, action(FK), type(FK), address_id(FK), lat(float), lng(float), liquidation_site_id(FK), execute(bit), organization_unit_id(FK), ordered_service_id(FK), note, time_window_start/end(nvarchar(5)), duration(int), manipulation_time_in_minutes(int), is_monitored, execution_site_id(FK), has_flat_rate, liquidation_garbage_types(nvarchar(MAX)), loading_code(nvarchar(15)).
  - **CHYBI:** site_id, container_type_id, garbage_group_id, container_count, status (stav odbaveni), frequency_description.
  - **Existujici delty** (extrakce RP #2): next_location_distance/time jsou float (DS: cele cislo); liquidation_garbage_types je JSON (DS: Reference []).
- Delta: 6 novych sloupcu na existujici tabulce.

---

### B5. Likvidacni misto -- vazba na skupinu odpadu

**Pozadavek:**
Likvidacni misto bude mit vazbu na skupinu odpadu (pro okruh dne).

**Dopad na entitni model (datovy slovnik):**
- Zmena entity **Likvidacni misto** (extrakce RP #6): novy atribut
  - `Skupina odpadu` -- Reference na Skupinu odpadu, nepovinna
  - Dnes DS-RP Likvidacni misto ma 17 atributu: ext. ID, Nazev, Adresa, Souradnice, Kontakt, Poznamka, Provozovna, Stav geokodovani, Zmena adresy, Druhy odpadu (kolekce), Vlastni LM, Kod nakladani, systemove. **Nema** vazbu na skupinu odpadu.
- Alternativa: rozsireni existujici vazebni entity **Druh odpadu likvidacniho mista** (extrakce RP #7) o atribut `Skupina odpadu`

**Dopad na fyzicky model (DDL):**
- Varianta A: ALTER TABLE `liquidation_site`:
  - `garbage_group_id` int NULL FK -> garbage_group(id)
- Varianta B: ALTER TABLE `liquidation_site_accepted_garbage_type`:
  - `garbage_group_id` int NULL FK -> garbage_group(id)

**Soucasny stav:**
- Entitni model (DS): **Likvidacni misto** (extrakce RP #6) -- 17 atributu, **NEMA** skupinu odpadu. Kolekce druhu odpadu je resena asociacni entitou **Druh odpadu likvidacniho mista** (extrakce RP #7) s atributy: ext. ID, Likvidacni misto, Druh odpadu, Kod nakladani(15), Provozovna, systemove.
- Fyzicky model (DDL):
  - `liquidation_site`: 16 sloupcu (id, name, address_id(FK), lat, lng, contact, note, organization_unit_id(FK), active, geocoding_state, external_id, owned_liquidation_site, winyx_id/winyx_id_old, audit). **NEMA** `garbage_group_id`.
  - `liquidation_site_accepted_garbage_type`: 7 sloupcu (id, liquidation_site_id(FK), garbage_type_id(FK), loading_code(15), organization_unit_id(FK), active, external_id, winyx_id). **NEMA** `garbage_group_id`.
- Delta: 1 novy sloupec na jedne z existujicich tabulek.

---

### B6. Vozidlo -- nove atributy pro strategicke planovani

**Pozadavek:**
Pro strategickou optimalizaci jsou potreba nove atributy vozidla:
- Objem nastavby (kapacita)
- Nosnost (vahova kapacita)
- Pracovni doba (po kterou je vozidlo k dispozici)
- "Pouzit pro SP" (priznak, zda vozidlo vstupuje do strategickeho planovani)

**Dopad na entitni model (datovy slovnik):**
- Zmena entity **Vozidlo** (extrakce RP #5): nove atributy
  - `Objem nastavby` -- Desetinne cislo, nepovinna; objem v litrech/m3
  - `Nosnost` -- Desetinne cislo, nepovinna; nosnost v kg
  - `Pracovni doba od` -- Cas, nepovinna
  - `Pracovni doba do` -- Cas, nepovinna
  - `Pouzit pro strategicke planovani` -- Boolean, nepovinna
  - Dnes DS-RP Vozidlo ma atributy z dvou DS sekci ("Vozidlo" + "Objekt"): ext. ID FLW, ext. ID ERP, Nazev(50), RZ(50), Skupina(50), Umisteni(255), Typ dopravy, Provozovna, Evidencni cislo(255), Druh vozidla, Datum zarazeni/vyrazeni, Zpusob vyhodnoceni realizace, Je aktivni, systemove. **NEMA** zadne kapacitni atributy.

**Dopad na fyzicky model (DDL):**
- ALTER TABLE `vehicle`: nove sloupce
  - `body_volume` float NULL -- objem nastavby
  - `load_capacity` float NULL -- nosnost
  - `working_hours_from` nvarchar(5) NULL -- pracovni doba od (format HH:MM)
  - `working_hours_to` nvarchar(5) NULL -- pracovni doba do (format HH:MM)
  - `use_for_strategic_planning` bit NULL -- priznak pro SP

**Soucasny stav:**
- Entitni model (DS): **Vozidlo** (extrakce RP #5) -- DS kombinuje atributy z "Vozidlo" (8 atributu) a "Objekt" (16 atributu). **NEMA** zadne kapacitni atributy.
- Fyzicky model (DDL): `vehicle` ma 31 sloupcu (id, name(100), registration_plate(255), organization_unit_id(FK), flww_id(int), identification(255), object_type, device_type_id, color, cost_center_code, group_id, parent_name, vehicle_full_path, vehicle_type_id(FK), evidence_number, vin, sort_id, date_insertion, date_rejection, realization_type(FK), external_id, active, atd.). **NEMA** zadny ze zminenych sloupcu (body_volume, load_capacity, working_hours_*, use_for_strategic_planning).
  - **Existujici delty DDL vs DS** (extrakce RP #5): DDL ma navic identification, winyx_id, device_type_id, color, color_inherited, cost_center_*, group_id, driver_id, has_gps, vin, sort_id. Typova: flww_id je int v DDL vs. Retezec(50) v DS.
- Delta: 5 novych sloupcu na existujici tabulce. Etapa 2 (strategicke planovani).

---

### B7. Konfigurace generovani OS

**Pozadavek:**
Konfigurace RP pro automaticke generovani OS:
- Frekvence a cas spusteni
- Obdobi vzniku OS (kolik dnu dopredu)
- Volba vzniku fiktivniho okruhu dne (ANO/NE)

**Dopad na entitni model (datovy slovnik):**
- Nova entita: **Konfigurace generovani CS** -- dnes v DS-RP neexistuje
  - `Provozovna` -- Reference na Provozovnu, povinna
  - `Frekvence spusteni` -- Retezec (CRON vyraz), povinna
  - `Obdobi generovani (dny)` -- Cislo (kladne, cele), povinne; kolik dnu dopredu
  - `Vytvorit fiktivni okruh` -- Boolean, povinne
  - Systemove atributy
- Alternativa: vyuziti existujici entity/tabulky `default_settings` (klicova-hodnotova konfigurace)

**Dopad na fyzicky model (DDL):**
- Varianta A (nova tabulka): NOVA tabulka `cs_generation_config`:
  - `id` int IDENTITY(1,1) NOT NULL PK
  - `organization_unit_id` int NOT NULL FK -> organization_unit(id)
  - `run_frequency_cron` nvarchar(100) NOT NULL
  - `generation_period_days` int NOT NULL
  - `create_fictive_circuit` bit NOT NULL DEFAULT 0
  - Audit sloupce
- Varianta B (existujici tabulka): nove zaznamy v `default_settings` (name, val)

**Soucasny stav:**
- Entitni model (DS): Takova entita v DS-RP **neexistuje**.
- Fyzicky model (DDL): Tabulka `default_settings` **existuje** (name nvarchar, val nvarchar) -- jednoducha klicova-hodnotova tabulka. Pro per-provozovna konfiguraci je nedostatecna (nema organization_unit_id).
- Delta: Nova tabulka (varianta A) nebo nove zaznamy v existujici tabulce (varianta B). Per-provozovna konfigurace vyzaduje variantu A.

---

### B8. Konfigurace strategicke optimalizace (Etapa 2)

**Pozadavek:**
Parametry pro optimalizacni algoritmus:
- Kriteria a omezeni okruhu SP (max cas, max vaha, max objem)
- Hlavni optimalizacni kriterium (cas)
- Distancni matice a dopravni sit

**Dopad na entitni model (datovy slovnik):**
- Nove entity (vsechny dnes v DS-RP neexistuji):
  - **Verze strategickeho planovani** -- kontejner pro jeden beh SP
    - `Provozovna`, `Nazev`, `Stav`, systemove
  - **Konfigurace SP** -- parametry behu
    - `Verze SP` -- Reference, povinna
    - `Max doba trasy (min)`, `Max hmotnost trasy (kg)`, `Max objem trasy (l)`
    - `Optimalizacni kriterium` -- Retezec
    - Parametry vozidel, depo, LM
  - **Strategicka objednana sluzba (SOS)** -- obdoba OS pro SP
  - **Strategicky okruh dne (SOD)** -- obdoba okruhu dne pro SP

**Dopad na fyzicky model (DDL):**
- NOVA tabulka `strategic_planning_version`:
  - `id`, `organization_unit_id`(FK), `name`, `status`, `created_at`, audit sloupce
- NOVA tabulka `strategic_planning_config`:
  - `id`, `strategic_planning_version_id`(FK)
  - `max_route_duration_minutes`(int), `max_route_weight_kg`(float), `max_route_volume_liters`(float)
  - `optimization_criterion`(nvarchar)
  - Parametry vozidel, depo, LM
- NOVA tabulka `strategic_ordered_service`: obdoba ordered_service pro SP
- NOVA tabulka `strategic_day_circuit`: obdoba day_circuit pro SP

**Soucasny stav:**
- Entitni model (DS): Zadna z techto entit v DS-RP **neexistuje**. Zcela nova domena.
- Fyzicky model (DDL): Zadna z techto tabulek **neexistuje**.
- Delta: 4+ novych tabulek. Etapa 2+. Zcela nova domena bez existujicich zakladu v DS ani DDL.

---

### B9. Skupina odpadu v RP -- sdileny ciselnik

**Pozadavek:**
Ciselnik skupin odpadu bude spolecny pro RP a PP. V RP se bude pouzivat pro filtraci OS, okruhu dne. Nastaveni vazby druh -> skupina bude mozne jen na strane PP, RP bude cerpat.

**Dopad na entitni model (datovy slovnik):**
- Nova entita v RP: **Skupina odpadu** -- synchronizovana z PP
  - `Externi identifikator` -- Retezec(255), nepovinna; id z PP
  - `Nazev` -- Retezec(50/255), povinna
  - `Zkraceny nazev` -- Retezec(20/255), nepovinna
  - `Popis` -- Retezec(255), nepovinna
  - `Provozovna` -- Reference na Provozovnu, nepovinna
  - `Je aktivni`, systemove
  - (Vzor: DS-PP Skupina odpadu, extrakce PP #6)
- Zmena entity **Druh odpadu** v RP: novy atribut
  - `Skupina odpadu` -- Reference na Skupinu odpadu, nepovinna
  - Dnes DS-RP nedefinuje explicitni entitu Druh odpadu jako plnohodnotnou entitu s DS profilem (v extrakci RP je soucasti vazebni entity #7). V DDL-RP existuje tabulka `garbage_type` (code, name, description, id, active, category, external_id).

**Dopad na fyzicky model (DDL):**
- NOVA tabulka `garbage_group` v RP:
  - `id` int IDENTITY(1,1) NOT NULL PK
  - `external_id` nvarchar(255) NULL -- id z PP
  - `name` nvarchar(255) NOT NULL
  - `short_name` nvarchar(255) NULL
  - `description` nvarchar(255) NULL
  - `organization_unit_id` int NULL FK -> organization_unit(id)
  - `is_active` bit NOT NULL DEFAULT 1
  - Audit sloupce
- ALTER TABLE `garbage_type` v RP: novy sloupec
  - `garbage_group_id` int NULL FK -> garbage_group(id)

**Soucasny stav:**
- Entitni model (DS): V DS-RP **neexistuje** entita Skupina odpadu. DS-RP definuje Druh odpadu na urovni vazebni entity (Druh odpadu likvidacniho mista, extrakce RP #7).
- Fyzicky model (DDL):
  - `garbage_group` v RP DDL **neexistuje**.
  - `garbage_type` v RP existuje (code(50), name(100), description(255), id, active, category(4), external_id(255)). **NEMA** sloupec `garbage_group_id`.
- Delta: 1 nova tabulka + 1 novy sloupec na existujici tabulce. Synchronizacni mechanismus z PP.

---

### B10. Okruh (Circuit) v RP -- synchronizovany z PP

**Pozadavek:**
RP potrebuje pracovat s okruhy pro generovani objednanych sluzeb a okruhu dne.

**Dopad na entitni model (datovy slovnik):**
- Nova entita v RP: **Okruh** -- synchronizovana z PP
  - `Externi identifikator` -- Retezec(255); id z PP
  - `Nazev` -- Retezec(255), povinna
  - `Provozovna` -- Reference na Provozovnu
  - `Vozidlo` -- Reference na Vozidlo, nepovinna; preferovane vozidlo
  - `Je aktivni`, systemove
  - (Vzor: nova entita Okruh v PP, viz A1)
- V DS-RP dnes **neexistuje** zadna entita Okruh.

**Dopad na fyzicky model (DDL):**
- NOVA tabulka `circuit` v RP:
  - `id` int IDENTITY(1,1) NOT NULL PK
  - `external_id` nvarchar(255) NULL -- id z PP
  - `name` nvarchar(255) NOT NULL
  - `organization_unit_id` int NOT NULL FK -> organization_unit(id)
  - `vehicle_id` int NULL FK -> vehicle(id)
  - `is_active` bit NOT NULL DEFAULT 1
  - Audit sloupce

**Soucasny stav:**
- Entitni model (DS): V DS-RP **neexistuje**.
- Fyzicky model (DDL): `circuit` v RP **neexistuje**.
- Delta: Zcela nova tabulka. Synchronizace z PP.

---

### B11. Rozvrh (Schedule) v RP -- synchronizovany z PP

**Pozadavek:**
RP potrebuje rozvrhy a jejich kalendare pro generovani OS.

**Dopad na entitni model (datovy slovnik):**
- Nova entita v RP: **Rozvrh** -- synchronizovana z PP
  - Atributy: ext. ID, Nazev, Popis, Provozovna, Platnost od/do, systemove
  - (Vzor: nova entita Rozvrh v PP, viz A2)
- Nova entita v RP: **Kalendarni den rozvrhu** -- synchronizovana z PP
  - Atributy: Rozvrh (Reference), Kalendarni datum, systemove
- V DS-RP dnes **neexistuji** zadne entity pro rozvrhy.

**Dopad na fyzicky model (DDL):**
- NOVA tabulka `schedule` v RP (struktura odpovidajici PP):
  - `id`, `external_id`, `name`, `description`, `organization_unit_id`(FK), `valid_from`, `valid_to`, `is_active`, `tenant_id`, audit sloupce
- NOVA tabulka `schedule_calendar_day` v RP:
  - `id`, `schedule_id`(FK), `calendar_date`, `is_active`, audit sloupce

**Soucasny stav:**
- Entitni model (DS): V DS-RP **neexistuji**.
- Fyzicky model (DDL): Tabulky `schedule` a `schedule_calendar_day` v RP **neexistuji**.
- Delta: 2 nove tabulky. Synchronizace z PP.

---

### B12. Vahy a objemy na okruhu dne (pro strategicke planovani)

**Pozadavek:**
Ridic zadava pres FOB hmotnost navezenou na LM a objemove vyuziti. Hodnoty se ukladaji k okruhu dne v RP. Dispecer je muze editovat.

**Dopad na entitni model (datovy slovnik):**
- Nova entita: **Zaznam vahy okruhu dne** -- dnes v DS-RP neexistuje
  - `Okruh dne` -- Reference na Okruh dne, povinna
  - `Hmotnost (kg)` -- Desetinne cislo, povinna
  - `Objemove vyuziti` -- Desetinne cislo, nepovinna; hodnoty 0.25, 0.5, 0.75, 1.0
  - `Cas zaznamu` -- Datum a cas, povinny
  - `Zdroj` -- Retezec, povinny; "FOB" nebo "manual"
  - Systemove atributy

**Dopad na fyzicky model (DDL):**
- NOVA tabulka `day_circuit_weight_record`:
  - `id` int IDENTITY(1,1) NOT NULL PK
  - `day_circuit_id` int NOT NULL FK -> day_circuit(id)
  - `weight_kg` float NOT NULL
  - `volume_utilization` float NULL
  - `recorded_at` datetimeoffset NOT NULL
  - `source` nvarchar(50) NOT NULL -- 'FOB' / 'manual'
  - Audit sloupce

**Soucasny stav:**
- Entitni model (DS): V DS-RP **neexistuje** takova entita. Okruh dne jako celek neexistuje (viz B1).
- Fyzicky model (DDL): Tabulka **neexistuje**.
- Delta: Nova tabulka. Etapa 2. Zavisla na existenci tabulky `day_circuit` (B1).

---

### B13. Typ nadoby v RP -- rozsireni

**Pozadavek:**
Cas obsluhy typu nadoby bude synchronizovan z PP do RP.

**Dopad na entitni model (datovy slovnik):**
- V DS-RP neni Typ nadoby explicitne definovan jako plnohodnotna entita s DS profilem. V DDL-RP existuje tabulka `container_type`.
- Novy atribut (na urovni DS):
  - `Cas obsluhy` -- Cislo (kladne, cele), nepovinna; cas obsluhy v sekundach (synchronizovany z PP)
  - (Vzor: novy atribut na PP Typ nadoby, viz A5)

**Dopad na fyzicky model (DDL):**
- ALTER TABLE `container_type` v RP: novy sloupec
  - `service_time_seconds` int NULL

**Soucasny stav:**
- Entitni model (DS): DS-RP nedefinuje plnohodnotny profil entity Typ nadoby. V extrakci RP se tabulka `container_type` vyskytuje jen v kontextu jinych entit.
- Fyzicky model (DDL): `container_type` v RP **existuje** (id, name, description, transport_type_id(FK), active, organization_unit_id(FK), external_id). Na rozdil od PP verze ma RP tabulka navic sloupce `transport_type_id` a `organization_unit_id`. Sloupec `service_time_seconds` **neexistuje**.
- Delta: 1 novy sloupec. Synchronizace z PP.

---

### Souhrn RP -- prehled zmen (entitni + fyzicky model)

| Entita / Tabulka | DS dnes | DDL dnes | Typ zmeny | Etapa |
|---|---|---|---|---|
| **Okruh dne** (`day_circuit`) | Neexistuje | Neexistuje | NOVA entita + tabulka | 1 |
| **Likvidacni misto okruhu dne** (`day_circuit_liquidation_site`) | Neexistuje | Neexistuje | NOVA entita + tabulka | 1 |
| Polozka DV . `day_circuit_id` | Neexistuje v DS | Neexistuje v DDL | NOVY atribut + sloupec | 1 |
| Typ polozky DV -- novy kod | Existuje (4 hodnoty) | Existuje (id, code) | NOVY zaznam | 1 |
| Objednana sluzba -- CS sloupce | 31 atributu (kontejner.) | 29 sloupcu (kontejner.) | ROZSIRENI (8+ sloupcu) | 1 |
| Lokace OS -- CS sloupce | 25 atributu | 26 sloupcu | ROZSIRENI (6 sloupcu) | 1 |
| **Skupina odpadu** (`garbage_group`) | Neexistuje v DS-RP | Neexistuje v DDL-RP | NOVA entita + tabulka (sync z PP) | 1 |
| Druh odpadu . `garbage_group_id` | Neexistuje v DS-RP | Neexistuje v DDL-RP | NOVY atribut + sloupec | 1 |
| **Okruh** (`circuit`) | Neexistuje v DS-RP | Neexistuje v DDL-RP | NOVA entita + tabulka (sync z PP) | 1 |
| **Rozvrh** (`schedule`) + **Kal. den** | Neexistuje v DS-RP | Neexistuje v DDL-RP | NOVE entity + tabulky (sync z PP) | 1 |
| Typ nadoby . `service_time_seconds` | Neexistuje v DS-RP | Neexistuje v DDL-RP | NOVY atribut + sloupec | 1 |
| **Konfigurace gen. CS** (`cs_generation_config`) | Neexistuje | `default_settings` (klicova-hodnotova) | NOVA entita + tabulka | 1 |
| Vozidlo -- SP sloupce | Neexistuji v DS-RP | Neexistuji v DDL-RP | ROZSIRENI (5 sloupcu) | 2 |
| Likvidacni misto . `garbage_group_id` | Neexistuje | Neexistuje | NOVY sloupec | 1 |
| **Zaznam vahy okruhu dne** (`day_circuit_weight_record`) | Neexistuje | Neexistuje | NOVA entita + tabulka | 2 |
| **Verze SP** (`strategic_planning_version`) | Neexistuje | Neexistuje | NOVA entita + tabulka | 2 |
| **Konfigurace SP** (`strategic_planning_config`) | Neexistuje | Neexistuje | NOVA entita + tabulka | 2 |
| **Strat. OS** (`strategic_ordered_service`) | Neexistuje | Neexistuje | NOVA entita + tabulka | 2 |
| **Strat. okruh dne** (`strategic_day_circuit`) | Neexistuje | Neexistuje | NOVA entita + tabulka | 2 |
| Typ dopravy -- novy zaznam CS | Existuje (6 hodnot) | Existuje (id, name, is_available) | NOVY zaznam | 1 |

**Celkem RP Etapa 1:** 7 novych tabulek, ~14+ novych sloupcu na existujicich tabulkach, 2 nove zaznamy v ciselnících.
**Celkem RP Etapa 2:** 5 novych tabulek, 5 novych sloupcu na existujici tabulce (vehicle).
