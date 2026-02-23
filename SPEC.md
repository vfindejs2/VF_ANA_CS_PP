# Specifikace: CS-MPG Analýza

## Kontext
Projekt Cyklické svozy MP SK zavádí plánování a realizaci cyklických svozů malých nádob od smluvních zákazníků. Zahrnuje rozšíření systémů Pasport (PP), RoadPlan (RP), FleetwareOnBoard (FOB) a napojení na Helios Nephrite (HEN).

Tento dílčí projekt poskytuje **business analytickou a konzultační podporu** napříč třemi oblastmi:
- **Integrace** — analýza požadavků, návrh architektury a business zadání pro integrační toky
- **Datové modely** — návrh a review datových struktur pro jednotlivé systémy
- **Konzultace** — podpora týmu při návrhu dílčích funkčností (workflow, UI logika, business pravidla)

Samotná implementace není ve scope — výstupem jsou specifikace, zadání a doporučení.

Zapojené systémy: PP, RP, FOB (dodavatel RADIUM), HEN (dodavatel D2B).
Zadavatel: Marius Pedersen, a.s. (MP SK).

## Cíle
### Integrace
1. Zmapovat a detailně specifikovat integrační toky pro Etapu 1
2. Navrhnout cílovou integrační architekturu (včetně přechodného stavu s MDB)
3. Připravit business zadání pro implementaci každé integrační služby

### Datové modely
4. Navrhnout/reviewovat datové modely pro dotčené systémy
5. Definovat datové kontrakty a mapování mezi systémy

### Konzultace a průřezová podpora
6. Poskytovat analytickou podporu při návrhu dílčích funkčností
7. Identifikovat rizika, závislosti a otevřené body napříč oblastmi

## Požadavky
### Funkční požadavky
- [FR-01] HEN → PP: Přenos číselníků (okruhy, rozvrhy, kalendáře, zóny) a vazeb PZ↔okruh/rozvrh/zóna
- [FR-02] PP → RP: Přenos RPO (min. set), nádob, stanovišť, okruhů, rozvrhů, skupin odpadu, času obsluhy na typu nádoby
- [FR-03] RP → HEN: Export hlavičky DV a seznamu zařazených RPO pro sestavení realizačních dokladů
- [FR-04] RP → FOB: Přenos DV s okruhy dne a lokacemi stanovišť
- [FR-05] FOB → RP: Události realizace (idempotentní, offline fronta) — stavy okruhů a potvrzení obsluhy stanovišť
- [FR-06] RP → PP: Dotaz na RPO dle kalendářního dne a provozovny (pro generování OS)

### Nefunkční požadavky
- [NFR-01] Výkon: latence v jednotkách sekund, dimenzování na 150 vozidel, 600 stanovišť/DV
- [NFR-02] Dostupnost: SaaS dle SLA, FOB offline režim
- [NFR-03] Bezpečnost: TLS/mTLS, OAuth2, šifrování, audit logy
- [NFR-04] Observabilita: X-Correlation-Id, metriky, ELK stack
- [NFR-05] Odolnost: idempotence (UUID), retry s backoff, dead-letter queue, eventual consistency
- [NFR-06] Verzování API: nové REST API služby musí podporovat verzování (konkrétní strategie na implementaci)

## Scope
### In scope
#### Integrace
- Analýza a specifikace 4 integračních toků Etapy 1 (HEN→PP, PP→RP, RP→HEN, RP→PP dotaz)
- Návrh integrační architektury (přímé integrace, REST API + MDB přechodně)
- Datové kontrakty (JSON schema) pro každý tok
- Business zadání pro implementaci jednotlivých služeb
- Identifikace závislostí na třetí strany (D2B)

#### Datové modely
- Návrh a review datových modelů pro dotčené systémy (PP, RP, FOB)
- Mapování entit mezi systémy
- Definice datových kontraktů

#### Konzultace
- Analytická podpora při návrhu dílčích funkčností pro tým
- Review business pravidel a workflow
- Ad-hoc konzultace k technickým a business otázkám

### Out of scope
- Implementace (kód, integrace, datové modely)
- Integrace RP ↔ FOB (řeší M. Taraba)
- Služba RP-sync-external-Pasport / RP → PP zápis (až Etapa 2+)
- Integrace strategické optimalizace (Etapa 2)
- Přechod správy okruhů z HEN na PP (Etapa 2)
- Kompletní migrace MDB → API
- Integrační hub / ESB
- Veřejný web

## Omezení
- MDB komunikace zůstává v Etapě 1 souběžně s novými REST API
- Přímé integrace (bez ESB) — alternativy k prověření později
- Závislost na D2B pro HEN API a testovací prostředí
- Rozhodnutí o paralelním plánování DV (HEN vs. RP) musí předcházet implementaci

## Akceptační kritéria
### Integrace
- [AC-01] Každý integrační tok má kompletní business zadání (směr, účel, triggery, datový kontrakt, chybové stavy, SLA)
- [AC-02] Integrační architektura je zdokumentována a schválena stakeholdery
- [AC-03] Otevřené otázky (zejména OQ-05 paralelní plánování DV) jsou rozhodnuty nebo eskalovány
- [AC-04] Datové kontrakty jsou definovány minimálně na úrovni JSON schema draft
- [AC-05] Závislosti na D2B/HEN jsou explicitně popsány s navrhovaným řešením

### Datové modely
- [AC-06] Datové modely pro dotčené systémy jsou navrženy/reviewovány a zdokumentovány
- [AC-07] Mapování entit mezi systémy je definováno

### Konzultace
- [AC-08] Výstupy konzultací jsou zachyceny v docs/ (zápisy, doporučení, rozhodnutí)
