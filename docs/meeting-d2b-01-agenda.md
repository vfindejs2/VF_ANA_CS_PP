# Meeting s D2B — úvodní schůzka k integracím CS

**Projekt:** Cyklické svozy MP SK — integrační práce
**Účel:** Představení rozsahu projektu, zmapování připravenosti na straně D2B, dohodnutí spolupráce
**Účastníci:** RADIUM (integrace), D2B (HEN), MP SK
**Délka:** 60–90 min

---

## Agenda

### 1. Představení projektu a kontextu (cca 15 min)

- Projekt Cyklické svozy — co se mění, proč, časový rámec
- Role D2B v projektu: dodavatel HEN, klíčový integrační partner
- Integrační toky týkající se HEN:
  - **HEN → PP**: přenos číselníků a vazeb (okruhy, rozvrhy, kalendáře, zóny, vazby PZ)
  - **RP → HEN**: export podkladů pro realizační doklady (hlavička DV + zařazené RPO)

### 2. Detailnější rozpad témat (cca 45 min)

- **HEN → PP rozšíření**: nové entity/atributy pro CS (okruhy, rozvrhy, kalendáře, zóny, vazby PZ↔okruh/rozvrh/zóna)
  - Má HEN tyto entity a atributy k dispozici na existujícím API, nebo je třeba je vyvinout?
  - Jaká je představa o datovém kontraktu?
- **RP → HEN nový tok**: podklady pro realizační doklady
  - Jak dnes HEN přijímá data z externích systémů? Existují nějaké využitelné rozhraní?
  - Co HEN potřebuje dostat, aby vytvořil realizační doklad?
  - Bude to rozšíření stávající služby nebo nový endpoint?
- **MDB vs. REST API v Etapě 1**:
  - Otázka přechodu vybraných MDB entit na API již v Etapě 1.

### 3. Kapacity a spolupráce (cca 10 min)

- Potvrzení kapacit D2B k dispozici pro součinnost na projektu CS. Analytické kapacity, vývojové kapacity.
- Kdo bude technický kontakt na straně D2B pro integrační otázky?
- Preferovaný způsob komunikace (pravidelné cally, ad-hoc komunikace)?
- Bude možné zřídit pro RAD přístup na testovací prostředí HEN - existující API?

### 4. Další kroky a dohody (cca 10 min)

- Shrnutí akcí a odpovědností
- Dohoda na dodání podkladů (v závislosti na výstupu z předchozích kroků):
  - [ ] D2B: API dokumentace stávajících HEN endpointů
  - [ ] D2B: informace o testovacím prostředí (termín, přístupy)
  - [ ] RAD: dodání business zadání integračních toků HEN→PP a RP→HEN (po dokončení specifikace)
- Termín dalšího setkání

---

## Poznámky pro přípravu

**Co vzít s sebou / připravit:**
- RAD: Schéma integračních toků (z HLC)
- RAD: Seznam BCED_ entit relevantních pro HEN toky

**Rizika meetingu:**
- D2B nemusí mít kapacity
