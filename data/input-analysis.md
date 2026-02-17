# Analýza vstupních dokumentů: CS-MPG Integrace

**Datum analýzy:** 2026-02-11
**Analyzoval:** Claude (session 1)

---

## 1. Seznam zdrojových dokumentů

| # | Dokument | Typ | Velikost | Popis |
|---|----------|-----|----------|-------|
| 1 | `PRD_Cyklicke_Svozy_MP_SK.md` | Markdown (PRD) | 17 KB | Produktový dokument — kompletní PRD pro projekt Cyklické svozy MP SK, generovaný z HLC konceptu. Pokrývá MVP (Etapa 1) i Etapu 2. |
| 2 | `Komunikace mezi systémy - část HLC CS.docx` | Word (HLC výňatek) | 150 KB | Kapitola z cílového konceptu HLC zaměřená na komunikaci mezi systémy — principiální schéma, integrační mapa, předběžný popis datových toků. |
| 3 | `MDB_schema_DDL.txt` | SQL DDL | 55 KB | Kompletní DDL schema integrační databáze MDB (`MPGSkCommNew`, SQL Server). Obsahuje 22 business entit (BCED_*), 6 candidate/staging tabulek, sync tracking tabulky (*_synced_at) pro Pasport i RoadPlan, a synchronizační views. |

---

## 2. Extrahované požadavky

### 2.1 Funkční požadavky na integrace

#### FR-INT-01: HEN → PP — Přenos číselníků a vazeb
- **Zdroj:** PRD kap. 5.1, HLC kap. Integrační mapa
- **Popis:** Přenos číselníků (okruhy, rozvrhy, kalendáře, zóny) a vazeb mezi předmětem smlouvy (PZ) a entitami okruh/rozvrh/zóna z Helios Nephrite do Pasportu.
- **Kanál:** Služba `PP-sync-external-Helios` (REST + dočasně MDB)
- **Směr:** HEN → PP (jednosměrný)

#### FR-INT-02: PP → RP — Přenos dat RPO a návazností
- **Zdroj:** PRD kap. 5.2.1, HLC kap. Integrační mapa
- **Popis:** Přenos minimálního setu dat o položkách objednávky (RPO) potřebného pro RP — identifikace RPO, příjemce, místo realizace (geo), počty/typy nádob, druh/skupina odpadu, vazby na okruh/rozvrh, kalendářní dny. Dále nádoby, stanoviště, okruhy, rozvrhy, skupiny odpadu, čas obsluhy na typu nádoby.
- **Kanál:** Služba `PP-sync-external-RoadPlan` (REST, nová služba)
- **Směr:** PP → RP (jednosměrný)
- **Datový kontrakt:** Definován v PRD kap. 5.2.1 (JSON schema)

#### FR-INT-03: RP → HEN — Export podkladů pro realizační doklady
- **Zdroj:** PRD kap. 5.2.4, HLC kap. Integrační mapa
- **Popis:** Export hlavičky denního výkonu (DV) a seznamu zařazených položek objednávek (RPO) z RoadPlanu do Heliosu pro sestavu realizačních dokladů.
- **Kanál:** Služba `RP-sync-external-Helios` (REST + dočasně MDB)
- **Směr:** RP → HEN (jednosměrný)
- **Datový kontrakt:** Definován v PRD kap. 5.2.4 (JSON schema)

#### FR-INT-04: RP ↔ FOB — Obousměrná komunikace DV a událostí
- **Zdroj:** PRD kap. 5.2.2, 5.2.3
- **Popis:** RP odesílá do FOB denní výkony s okruhy dne a lokacemi. FOB vrací do RP události realizace (zahájení/přerušení/dokončení okruhu, obsloužení/neobsloužení stanoviště). Události jsou idempotentní (UUID eventId), FOB podporuje offline frontu.
- **Kanál:** HTTP(S)/WebSocket (dle současného FOB 2.0)
- **Směr:** Obousměrný (RP→FOB: DV+lokace; FOB→RP: události)
- **Datové kontrakty:** Definovány v PRD kap. 5.2.2 a 5.2.3 (JSON schema)

#### FR-INT-05: Generování objednaných služeb (OS)
- **Zdroj:** PRD kap. 5.3.1
- **Popis:** Automatické generování OS z vazeb RPO↔Rozvrh/Okruh. RP periodicky (CRON) hledá RPO s kalendářním dnem, dotazuje se PP na RPO + vazby, vytváří/aktualizuje okruhy dne a DV. Idempotentní proces.
- **Sekvence:** RP→PP (GET /rpo) → RP vytvoří OS/OD/DV

#### FR-INT-06: Export realizace do HEN
- **Zdroj:** PRD kap. 5.3.3
- **Popis:** Po dokončení DV RP exportuje podklady do HEN (POST /dv). HEN vytvoří realizační doklady a vrací potvrzení s identifikátory.

### 2.2 Funkční požadavky na integrační služby (Etapa 2+)

#### FR-INT-07: Přechod správy okruhů z HEN na PP
- **Zdroj:** HLC kap. Rozšíření, PRD kap. 3
- **Popis:** Přesun SoT pro okruhy/rozvrhy/zóny z HEN do PP. Vyžaduje obrácení integračního toku — nový přenos PP→HEN a zastavení HEN→PP pro tyto entity.

#### FR-INT-08: Přechod z MDB na API
- **Zdroj:** HLC kap. Rozšíření, PRD kap. 5.1
- **Popis:** Nahrazení stávající komunikace přes MDB novými REST API službami po ověření v praxi.

#### FR-INT-09: Integrace strategické optimalizace (SP)
- **Zdroj:** HLC kap. Rozšíření, PRD kap. 8
- **Popis:** Integrace optimalizačního nástroje do platformy FLW. Přenos výstupů optimalizace do PP pro transformaci na standardní okruhy, rozvrhy a vazby na PZ.

### 2.3 Nefunkční požadavky

#### NFR-INT-01: Výkon a objem
- **Zdroj:** PRD kap. 5.4, 9
- ~150 vozidel, až ~600 OS/stanovišť na 1 DV
- Latence běžných operací v jednotkách sekund
- Horizontální/vertikální škálování

#### NFR-INT-02: Dostupnost
- **Zdroj:** PRD kap. 5.4, 9
- SaaS dle SLA (RPO/RTO k definici)
- FOB offline režim s opakovaným doručováním v rámci aktuálního dne

#### NFR-INT-03: Bezpečnost
- **Zdroj:** PRD kap. 5.4, 9
- TLS/mTLS, OAuth2 (client-credentials — NÁVRH)
- Šifrování dat v klidu i při přenosu
- Audit logy

#### NFR-INT-04: Observabilita
- **Zdroj:** PRD kap. 5.4
- Korelační `X-Correlation-Id` na všech tocích
- Metriky: propustnost, chybovost, latence
- ELK stack

#### NFR-INT-05: Odolnost a idempotence
- **Zdroj:** PRD kap. 5.5
- `eventId` (UUID) na FOB→RP událostech
- `If-None-Match`/`ETag` pro PP→RP synchronizace
- Exponenciální retry s jitterem, dead-letter queue
- Eventual consistency s kompenzačními akcemi

---

## 3. Architektonické principy

| Princip | Popis | Zdroj |
|---------|-------|-------|
| Jeden zdroj pravdy (SoT) | PP = smluvní/statické vstupy; RP = DV/OS/monitoring; HEN = ERP/realizační doklady | PRD kap. 4 |
| Jednosměrné synchronizace | Preferovat jednosměrné toky, minimalizovat obousměrnou komunikaci | PRD kap. 4, HLC |
| Volné párování přes API | Žádné přímé čtení cizí DB | PRD kap. 4 |
| Minimalizace přenášených dat | Synchronizovat jen nezbytné minimum pro danou funkčnost | PRD kap. 4, HLC |
| Centralizace logiky | Např. generování OS výhradně v RP | PRD kap. 4 |
| Přímé integrace (Etapa 1) | Bez integračního hubu/ESB; alternativy k prověření v detailní analýze | HLC |

### Maticová tabulka SoT

| Entita | SoT (Etapa 1) | SoT (Etapa 2) | Konzumenti |
|--------|----------------|----------------|------------|
| RPO (revize položky objednávky) | PP | PP | RP |
| Okruh/Rozvrh/Zóna | HEN → PP | PP | RP, HEN |
| Objednaná služba (OS), Okruh dne, DV | RP | RP | FOB, HEN |
| Stanoviště/Nádoba | PP | PP | RP, FOB |
| Realizace DV (doklady) | HEN | HEN | — |

---

## 4. Integrační služby — přehled

| Služba | Směr | Stav | Kanál (Etapa 1) | Kanál (cílový) |
|--------|------|------|------------------|-----------------|
| `PP-sync-external-Helios` | HEN → PP (+ potenciálně PP → HEN) | Existující, rozšiřuje se | MDB + REST API | REST API |
| `RP-sync-external-Helios` | RP ↔ HEN | Existující, rozšiřuje se | MDB + REST API | REST API |
| `PP-sync-external-RoadPlan` | PP → RP | **Nová** | REST API | REST API |
| `RP-sync-external-Pasport` | RP → PP | **Nová** | REST API | REST API |
| RP ↔ FOB | Obousměrný | Existující (FOB 2.0) | HTTP(S)/WebSocket | HTTP(S)/WebSocket |

---

## 5. Stakeholders a role

| Osoba | Organizace | Role |
|-------|-----------|------|
| M. Bulko | MP SK (zadavatel) | Klíčový stakeholder |
| S. Kodajová | MP SK (zadavatel) | Klíčový stakeholder |
| T. Stiksa | MP SK (zadavatel) | Klíčový stakeholder |
| V. Findejs | RADIUM (dodavatel FLW) | Tým dodavatele |
| J. Komenda | RADIUM (dodavatel FLW) | Tým dodavatele |
| M. Slivoně | RADIUM (dodavatel FLW) | Tým dodavatele |
| Z. Slaný | RADIUM (dodavatel FLW) | Tým dodavatele |
| E. Šťastná | RADIUM (dodavatel FLW) | Tým dodavatele |
| M. Taraba | RADIUM (dodavatel FLW) | Tým dodavatele |
| D2B | Dodavatel HEN | Třetí strana (konzultace, integrace) |

**Organizace:**
- **Marius Pedersen, a.s. (MP SK)** — zadavatel projektu
- **RADIUM** — dodavatel platformy FLW (Pasport, RoadPlan, FOB)
- **D2B** — dodavatel Helios Nephrite

---

## 6. Identifikované omezení a termíny

### Termíny (dle konceptu)
| Milník | Datum | Stav |
|--------|-------|------|
| Schůzky k funkčním požadavkům | 06/2025 | Hotovo |
| Schválení funkčních požadavků | 9.7.2025 | Hotovo |
| Příprava konceptu a fází řešení | 29.8.2025 | ? |
| Schválení konceptu | 11.9.2025 | ? |
| Podání nabídky | 6.10.2025 | ? |
| Návrh řešení I. fáze | 12/2025 | ? |

### Omezení
- MDB komunikace zachována v Etapě 1, nová rozhraní pouze přes REST API
- Přechod z MDB na API až po ověření v praxi (Etapa 2+)
- Integrační hub/ESB se v Etapě 1 nepoužívá (přímé integrace)
- Plánování DV paralelně v HEN i RP je problematické — preferováno buď–anebo na úrovni provozovny

---

## 7. Nejasnosti a protichůdné informace

### N-01: Rozsah služby `RP-sync-external-Pasport`
- **Problém:** HLC schéma uvádí službu `RP-sync-external-Pasport` (RP→PP), ale PRD ani HLC nespecifikuje, jaká data by tekla tímto směrem. Veškerá popsaná komunikace RP→PP není v datech explicitně zmíněna.
- **Dopad:** Není jasné, zda je tato služba potřebná v Etapě 1, nebo jde o rezervu pro budoucí potřeby.
- **Doporučení:** Ověřit s architektem, zda existuje use case pro RP→PP v Etapě 1.

### N-02: Přesný formát a rozsah MDB komunikace
- **Problém:** Oba dokumenty zmiňují zachování MDB v Etapě 1 souběžně s novým REST API, ale nespecifikují, které entity přesně zůstanou na MDB a které budou od začátku na API.
- **Dopad:** Riziko duplicitní implementace a nejasné migrace.
- **Doporučení:** Zmapovat stávající MDB entity a definovat migrační plán per entita.

### N-03: Autentizace a autorizace API
- **Problém:** PRD navrhuje OAuth2 (client-credentials), ale jde o NÁVRH — není potvrzeno.
- **Dopad:** Ovlivňuje návrh všech integračních služeb.
- **Doporučení:** Potvrdit bezpečnostní model se všemi stranami (RADIUM, D2B).

### N-04: Testovací prostředí HEN
- **Problém:** PRD uvádí jako otevřený bod potřebu testovacích přístupů do HEN pro dodavatele.
- **Dopad:** Blocker pro integrační testování.
- **Doporučení:** Eskalovat na MP SK / D2B.

### N-05: Komunikační kanál RP ↔ FOB
- **Problém:** PRD označuje kanál jako HTTP(S)/WebSocket s poznámkou "NÁVRH dle současného FOB 2.0". Není jasné, zda jde o stávající protokol nebo nový návrh.
- **Dopad:** Ovlivňuje design integrace RP↔FOB.
- **Doporučení:** Ověřit aktuální stav FOB 2.0 komunikačního protokolu.

### N-06: Paralelní plánování DV v HEN a RP
- **Problém:** PRD kap. 11 označuje jako otevřený bod. Preferováno buď–anebo na úrovni provozovny, jinak vysoká komplexita konfliktů.
- **Dopad:** Zásadně ovlivňuje integrační architekturu (RP→HEN tok a zpětná vazba).
- **Doporučení:** Rozhodnutí musí padnout před začátkem implementace integrace RP→HEN.

---

## 8. Otevřené otázky

| # | Otázka | Priorita | Stav | Odpověď / Poznámka |
|---|--------|----------|------|---------------------|
| OQ-01 | Jaký je přesný seznam entit, které zůstanou na MDB v Etapě 1 vs. přejdou na REST API? | Vysoká | **Částečně zodpovězeno** | Z DDL známe kompletní seznam MDB entit (22 BCED_ + 6 candidate). Některé entity mohou být migrovány na API již v Etapě 1, zejména ty s těsnou vazbou na nové toky nebo nové entity rozšiřující stávající. Konkrétní rozřazení nutno upřesnit při detailní specifikaci toků. |
| OQ-02 | Je služba `RP-sync-external-Pasport` potřebná v Etapě 1? | Střední | **Zodpovězeno** | **Ne, až Etapa 2+.** V Etapě 1 RP nebude zapisovat data zpět do PP. |
| OQ-03 | Je potvrzen OAuth2 client-credentials jako bezpečnostní model pro API? | Vysoká | **Otevřeno** | **Nepotvrzeno.** Je třeba projednat se všemi stranami (RADIUM, D2B, MP SK). |
| OQ-04 | Kdy budou k dispozici testovací přístupy do HEN? | Vysoká | **Otevřeno (blocker)** | **Nevyřešeno.** Je třeba eskalovat na MP SK / D2B. Blocker pro integrační testy. |
| OQ-05 | Bude plánování DV probíhat výhradně v RP, nebo paralelně i v HEN? | Kritická | **Otevřeno** | Za RADIUM preferujeme výhradně RP. Realisticky se očekává, že zákazník prosadí paralelní alternativu. Rozhodnutí musí padnout před implementací. Nutno navrhnout architekturu pro oba scénáře. |
| OQ-06 | Jaký je aktuální komunikační protokol FOB 2.0? | Střední | **Mimo scope** | Proprietární komunikace. Integrace RP↔FOB je mimo scope této části projektu — řeší Michal Taraba. |
| OQ-07 | Existují OpenAPI specifikace pro stávající služby? | Střední | **Zodpovězeno** | **Ano, existují.** Dodat jako vstupní dokument pro detailní specifikaci. |
| OQ-08 | Jaké jsou konkrétní hodnoty SLA (RPO/RTO)? | Střední | **Částečně zodpovězeno** | Existuje obecné SLA platformy, ne per integrační služba. V business zadání navrhneme doporučené hodnoty per tok. |
| OQ-09 | Jsou definovány limity na velikost payloadu? | Nízká | **Zodpovězeno** | **Vyřeší implementace.** Limity nejsou předdefinovány, necháme na implementačním návrhu. |
| OQ-10 | Jak bude řešena správa verzí API? | Střední | **Zodpovězeno** | Detail necháme na implementaci, ale **verzování API je explicitní požadavek** — musí být součástí NFR. |

---

## 9. Analýza MDB schématu (MPGSkCommNew)

**Zdroj:** `data/input/MDB_schema_DDL.txt`
**Databáze:** MPGSkCommNew (SQL Server, collation Slovak_CI_AS)

### 9.1 Business entity tabulky (prefix BCED_)

| Tabulka | Popis | PK | Klíčové FK |
|---------|-------|----|------------|
| `BCED_Address` | Adresy (sídla, provozovny, org. jednotky) | Id (bigint) | WinyXId, KodSidelniJednotky |
| `BCED_Bank` | Číselník bank | Id (int) | WinyXId |
| `BCED_ContainterType` | Typy nádob (kontejnerů) | Id (int) | Group, ApplicationArea, WinyXId |
| `BCED_ContractSubjectType` | Typy předmětu smlouvy | Id (int) | Code, WinyXId |
| `BCED_WasteType` | Typy odpadu | Id (int) | Code, WasteCategory, WinyXId |
| `BCED_Container` | Nádoby/kontejnery | Id (int) | → ContainerType |
| `BCED_Customer` | Zákazníci | Id (int) | → Address (OfficeAddress), WinyXId |
| `BCED_CustomerPerson` | Kontaktní osoby zákazníka | (Id, CustomerId) | → Customer |
| `BCED_BankAccount` | Bankovní účty zákazníka | (Id, CustomerId) | → Bank, → Customer |
| `BCED_OrganizationalUnit` | Organizační jednotky (provozovny MPG) | Id (int) | → Address, Code, WinyXId |
| `BCED_CustomerContract` | Smlouvy zákazníka | Id (bigint) | → OrganizationalUnit, → Customer (2×: Customer + Purchaser) |
| `BCED_CustomerContractWasteType` | Přiřazení typů odpadu ke smlouvě | (CustomerContractId, WasteTypeId) | → CustomerContract, → WasteType |
| `BCED_CustomerSite` | Stanoviště/provozovny zákazníka | Id (int) | → Address, → Customer, → OrganizationalUnit, GPS (Lat/Lon) |
| `BCED_CustomerContractCustomerSite` | Vazba smlouva↔stanoviště | (CustomerContractId, CustomerSiteId) | → CustomerContract, → CustomerSite |
| `BCED_CollectionDaysOfWeek` | Dny svozu v týdnu | Id (int) | → OrganizationalUnit, DaysOfWeek, CountPerMonth |
| `BCED_CollectionFrequency` | Frekvence svozu | Id (int) | → OrganizationalUnit, Group |
| `BCED_ContractItem` | Položky smlouvy (řádky objednávky) | (Id, Sequence, ContractSubjectTypeId, CustomerSiteId) | → CustomerContract, → WasteType, → ContainerType, → CollectionFrequency, → CollectionDaysOfWeek, → CustomerSite, → ContractSubjectType, → OrganizationalUnit |
| `BCED_ContractItemContainer` | Přiřazení kontejnerů k položkám smlouvy | LocationId (int) | → Container, → ContractItem (composite FK), → CustomerSite |
| `BCED_ContractItemDaysOfWeek` | Přiřazení dnů svozu k položkám smlouvy | (CustomerSiteId, CollectionDaysOfWeekId, ContractItemId, Sequence, ContractSubjectTypeId) | → CollectionDaysOfWeek, → ContractItem |
| `BCED_DisposalSite` | Místa zpracování/likvidace odpadu | Id (int) | → Address, → OrganizationalUnit, GPS (Lat/Lon) |
| `BCED_DisposalSiteWasteType` | Přiřazení typů odpadu k místu zpracování | (DisposalSiteId, WasteTypeId, LoadingCode) | → DisposalSite, → WasteType, → OrganizationalUnit |
| `BCED_CustomerApprovalResult` | Výsledky schválení zákazníka | CustomerToBeApprovedId | ApprovalResult, CustomerId |

### 9.2 Candidate/staging tabulky

Pattern: Entity založené v RoadPlanu, přenesené přes MDB do HEN ke kontrole a zavedení. HEN je pro tyto entity source of truth — uživatel v HEN kandidáta zkontroluje a schválí/zamítne. Nemají sloupce Status/Created — jen LastModified.

| Tabulka | Popis | FK |
|---------|-------|----|
| `AddressCandidate` | Kandidáti adres | — |
| `BankCandidate` | Kandidáti bank | — |
| `CustomerCandidate` | Kandidáti zákazníků | → AddressCandidate |
| `CustomerPersonCandidate` | Kandidáti kontaktních osob | → CustomerCandidate |
| `CustomerSiteCandidate` | Kandidáti stanovišť | → AddressCandidate, → CustomerCandidate |
| `BankAccountCandidate` | Kandidáti bankovních účtů | → BankCandidate, → CustomerCandidate |

### 9.3 Sync tracking tabulky a views

**Pattern:** Pro každou synchronizovanou entitu existuje:
- `BCED_{Entity}_pasport_synced_at` — tracking syncu do Pasportu (PK: deploy_env + pasport_id)
- `BCED_{Entity}_roadplan_synced_at` — tracking syncu do RoadPlanu (PK: deploy_env + external_id, + retry_count)
- Odpovídající VIEW `BCED_{Entity}_{pasport|roadplan}_sync` — CROSS JOIN entity × deploy_envs, LEFT JOIN synced_at

**Entity synchronizované do Pasportu (11):**
Address, ContainerType, CollectionDaysOfWeek, CollectionFrequency, ContractItem (+ ContractItemSequence), ContractItemContainer, ContractItemDaysOfWeek, Customer, CustomerContract, WasteType

**Entity synchronizované do RoadPlanu (13):**
Address, Bank, BankAccount, ContainerType, Customer, CustomerPerson, CustomerContract, CustomerContractWasteType, CustomerContractCustomerSite, CustomerSite, DisposalSite, DisposalSiteWasteType, WasteType

**Infrastruktura:**
- `sync_deploy_envs` — seznam deployment prostředí (PK: deploy_env)
- `TMP_BCKUP_BCED_Container_20251126` — dočasná záloha (ignorovat)

### 9.4 Klíčové poznatky pro integrační analýzu

1. **WinyXId** — sloupec na většině BCED_ entit. Reference na legacy systém WinyX. Relevantní pro mapování identit při migraci.
2. **Composite PK** — ContractItem má 4-sloupcový PK (Id, Sequence, ContractSubjectTypeId, CustomerSiteId). Komplexní vazby na ContractItemContainer a ContractItemDaysOfWeek.
3. **Multi-environment sync** — `sync_deploy_envs` umožňuje trackovat synchronizaci do více prostředí (např. staging, production).
4. **Candidate workflow (RP → HEN)** — Candidate tabulky slouží pro přenos entit založených v RoadPlanu do HEN. Uživatel v HEN kandidáta zkontroluje a schválí → entita se zavede do HEN (SoT). Tabulka `BCED_CustomerApprovalResult` trackuje výsledek schválení.
5. **Pasport vs. RoadPlan asymetrie** — RoadPlan přijímá více entit (13 vs. 11). RoadPlan navíc dostává: Bank, BankAccount, CustomerPerson, CustomerContractWasteType, CustomerContractCustomerSite, CustomerSite, DisposalSite, DisposalSiteWasteType. Pasport navíc dostává: CollectionDaysOfWeek, CollectionFrequency, ContractItem*, ContractItemContainer, ContractItemDaysOfWeek.
6. **Retry pattern** — roadplan sync tabulky mají `retry_count` (pasport sync ne) — RoadPlan sync má robustnější retry mechanismus.
7. **ContainerType.ApplicationArea** — view `BCED_ContainerType_roadplan_sync` filtruje `WHERE ApplicationArea = 'RoadPlan'`. Typy nádob jsou tedy segmentovány dle cílového systému.
8. **Relevance k OQ-01** — DDL poskytuje odpověď na otázku „které entity jsou na MDB". Všechny BCED_ entity + candidate tabulky tvoří stávající MDB rozhraní.

---

## 10. Shrnutí pro SPEC.md

Projekt CS-MPG Integrace se zaměřuje na **6 integračních toků** v rámci Etapy 1:

1. **HEN → PP** (číselníky, vazby) — rozšíření existující služby
2. **PP → RP** (RPO, nádoby, stanoviště, okruhy, rozvrhy, skupiny odpadu) — **nová služba**
3. **RP → HEN** (podklady pro realizační doklady) — rozšíření existující služby
4. **RP → FOB** (DV + okruhy dne) — existující komunikace, rozšíření
5. **FOB → RP** (události realizace) — existující komunikace, rozšíření
6. **RP → PP** (dotaz na RPO dle kalendáře) — **nová služba** (součást generování OS)

Plus 3 budoucí toky pro Etapu 2+ (přesun správy okruhů, migrace MDB→API, integrace SP).

Scope tohoto projektu (business analýza) zahrnuje:
- Analýzu požadavků na všechny integrační toky
- Návrh architektury a technického řešení
- Přípravu business zadání pro implementaci
- **Nikoli** samotnou implementaci
