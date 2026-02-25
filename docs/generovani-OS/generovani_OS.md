# Generování OS

## 1. Businessové shrnutí

Generování objednaných služeb (OS) je klíčový proces, který převádí data z revizí položek objednávek (RPO), rozvrhů a okruhů do provozně použitelných jednotek pro denní plánování v RoadPlan (RP).

Z business pohledu platí:
- RP pracuje primárně s entitou objednaná služba (OS), která představuje konkrétní realizaci služby v konkrétní den.
- OS vznikají z RPO na základě vazeb na rozvrh a okruh; nově vzniká také entita „okruh dne", která sdružuje OS pro denní výkon.
- Nutnou podmínkou pro automatické vygenerování OS je vazba RPO na rozvrh (kalendářní den v generovaném období).
- Vazba RPO na okruh a případně přiřazené vozidlo určuje míru automatizace: od pouhého vzniku OS až po automatické zařazení do okruhu dne a denního výkonu.
- Proces generování může běžet automaticky (konfigurovaně) i manuálně; manuální spuštění slouží zejména dispečerovi pro operativní zásah.
- Výstupem generování nejsou jen OS, ale i jejich lokace, okruhy dne a případně denní výkony, aby dispečer dostal připravená data pro finální plánování.
- Při změně vstupních dat (RPO/nádoby) systém podporuje přegenerování: původní OS deaktivuje, vytvoří nové a přegenerované záznamy následně čistí.
- Je podporováno i manuální generování OS „na výzvu" bez vazby na rozvrh pro mimořádné svozy, reklamace a podobné situace.

Hlavní business přínos procesu je zrychlení a standardizace přípravy dat pro dispečink, snížení ruční práce při sestavování denních výkonů a jasné oddělení rolí mezi zdrojovými daty (RPO/rozvrhy/okruhy) a operativním plánováním v RP.

## 2. Navrhované workflow (dle cílového konceptu)

### 2.1 Standardní workflow generování OS

1. Vstupem jsou RPO včetně nádob a lokací stanovišť.
2. Generování se spouští automaticky podle konfigurace (frekvence, čas, období), případně manuálně oprávněným uživatelem.
3. Systém vybere RPO s rozvrhem, který obsahuje kalendářní den v generovaném období.
4. Systém projde aktivní rozvrhy a pro relevantní RPO generuje OS a jejich lokace.
5. Podle vazby na okruh systém:
   - zařadí OS do existujícího okruhu dne, nebo
   - založí nový okruh dne.
6. Pokud je okruhu přiřazeno vozidlo, okruh dne se automaticky zařadí do denního výkonu (DV) vozidla.
7. Dispečer dostane připravená data (OS, lokace, okruhy dne, případně DV) pro finální operativní plánování.

### 2.2 Pravidla a kontroly v průběhu generování

- Nutná podmínka: RPO musí mít vazbu na rozvrh a rozvrh musí obsahovat den v generovaném období.
- Pro daný kalendářní den rozvrhu vzniká OS a její lokace pouze jednou.
- Okruh dne obsahuje pouze OS jedné skupiny odpadu.
- Pokud už pro RPO a den existuje OS, systém provede přegenerování:
  - původní OS deaktivuje a označí jako přegenerovanou,
  - založí novou OS a její lokace,
  - přegenerované OS a vazby se následně nočně čistí.

### 2.3 Manuální workflow

- Manuální spuštění generování z okruhů a rozvrhů:
  - spouští oprávněný uživatel synchronně,
  - platí stejné podmínky jako u automatu,
  - dispečer může omezit období a výběr na konkrétní rozvrhy/okruhy,
  - po doběhu systém vrátí souhrn (počty RPO, OS, lokací, okruhů).

- Manuální generování na výzvu z RPO (mimořádné svozy):
  - používá se bez vazby na rozvrh (např. výzva/reklamace),
  - uživatel vybírá konkrétní den,
  - systém nabídne jen RPO bez existující OS pro tento den,
  - pokud je RPO v okruhu, okruh se ignoruje a vzniká OS bez zařazení do okruhu dne.

### 2.4 Výsledné scénáře podle vazeb RPO

- `RPO v rozvrhu = ANO`, `RPO v okruhu = ANO`, `okruh má vozidlo = ANO`:
  - OS jsou zařazeny do okruhu dne a okruh dne do DV.
- `RPO v rozvrhu = ANO`, `RPO v okruhu = ANO`, `okruh má vozidlo = NE`:
  - OS jsou zařazeny do okruhu dne, který čeká na zaplánování do DV.
- `RPO v rozvrhu = ANO`, `RPO v okruhu = NE`:
  - vzniknou OS a lokace, dispečer je musí zařadit do okruhů dne.
- `RPO v rozvrhu = NE`:
  - OS a lokace se standardním generováním nevzniknou.

### 2.5 Dopad na roli dispečera

- Dispečer neřeší ruční zakládání všech OS, ale pracuje s předpřipravenou sadou dat.
- Jeho role se posouvá na kontrolu, finální sestavení okruhů dne a DV a řešení výjimek.
- Míra ruční práce závisí na kvalitě vazeb RPO -> Rozvrh -> Okruh a na přiřazení vozidel k okruhům.

## 3. Otevřené body / rozhodnutí k dopracování

### 3.1 Platnost rozvrhů a přiřazení (zásadní pro generování)

- Upřesnit datový model a pravidla pro `platnost rozvrhu` (v CK je uvedeno jako bod k dopracování).
- Upřesnit práci s `platností přiřazení RPO -> Okruh` a `RPO -> Rozvrh`:
  - v HEN chybí časový údaj „od kdy“,
  - v PP se zavádí vlastní platnost,
  - v Etapě 1 má být platnost nepovinná.
- Rozhodnout, jak se platnost promítne do generování OS při změnách v průběhu období (od kdy přegenerovat, co ponechat).

### 3.2 Pravidla pro změny v HEN a dopad na přegenerování OS

- HEN umožňuje editaci RPO i do minulosti; je potřeba potvrdit:
  - zda generování/přegenerování sahá i do minulých dnů,
  - jaké období je chráněné před automatickými zásahy,
  - jak se budou evidovat a auditovat přegenerované OS.
- Přidání/odebrání RPO z okruhu v HEN nevytváří novou verzi RPO:
  - je nutné definovat, podle čeho RP bezpečně pozná změnu a spustí odpovídající přegenerování.

### 3.3 Validace business pravidel při generování

- HEN nevaliduje logiku přiřazení RPO do okruhu; potvrdit, která validační pravidla musí být v RP/FLW povinná.
- HEN nevaliduje skupiny/druhy odpadu v okruhu; dopracovat konkrétní validaci na straně FLW/RP:
  - kdy generování zablokovat,
  - kdy pouze varovat dispečera.
- Potvrdit finální pravidla pro vznik „fiktivního okruhu dne“ a kdy má být použité jako default.

### 3.4 Governance manuálního generování

- Potvrdit oprávnění a role pro manuální spuštění generování (CK uvádí doporučení omezit počet uživatelů).
- Dodefinovat provozní pravidla:
  - kdo může spouštět generování za provozovnu,
  - zda bude potřeba schvalování / logování spuštění,
  - limity rozsahu (období, počet rozvrhů) kvůli výkonu a riziku kolizí.

### 3.5 Paralelní plánování HEN vs. RP

- Otevřený bod z CK: nutnost paralelního plánování v HEN i RP.
- Rozhodnutí je kritické pro workflow generování OS, protože ovlivňuje:
  - kdo je finální autorita pro denní plán,
  - jak řešit kolize změn po vygenerování OS,
  - jak nastavit synchronizaci a přepisování dat mezi RP a HEN.

### 3.6 Technické a provozní parametry generování k potvrzení

- Výchozí konfigurace automatu:
  - frekvence a čas spuštění,
  - generované období dopředu,
  - zapnutí/vypnutí fiktivního okruhu dne.
- Chování při opakovaném spuštění ve stejném dni:
  - jak agresivně přegenerovávat,
  - co dělat s již ručně upravenými okruhy dne / DV.
- Definovat očekávané výstupy a monitoring běhu:
  - minimální sada metrik (počet RPO, OS, lokací, okruhů, chyb, varování),
  - způsob upozornění při neúspěšném nebo částečně úspěšném běhu.
