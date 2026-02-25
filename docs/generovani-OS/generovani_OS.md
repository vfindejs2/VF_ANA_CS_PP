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

## 2. Shrnuté za HELIOS (analogie generování podkladu k plánu realizace)

Z pohledu HELIOS Nephrite (modul ZVOZ) existuje velmi podobná logika přípravy podkladů pro realizaci, která je analogická k našemu generování OS v PP/RP:

- V HEN je zdrojovou entitou **Predmet zmluvy (PZ)**, která je analogická našemu `RPO` jako vstupní jednotce pro plánování.
- Podklad pro plán realizace v HEN vzniká kombinací:
  - **Rozvrhu vývozu** (kalendář / dny vývozu),
  - **Zóny** (geografické seskupení podle adres stanoviska),
  - **Okruhu trasy** (plánovací seskupení PZ),
  - vazeb PZ na tyto entity.
- **Rozvrh** je v HEN na PZ veden jako **dynamický vztah** (nikoli jen atribut na PZ), což je důležitá analogie k našemu návrhu vazebních entit / historizovaných vazeb.
- **Zóna** je v HEN na PZ doplňována podle **adresy stanoviska** (statická vazba na stanovisku + odvození na PZ), což odpovídá potřebě rozlišovat zdroj lokace a odvozené údaje.
- **Okruh trasy** v HEN slouží jako stavební prvek pro plánování **Trasy dňa**, obdobně jako u nás `Okruh dne` a následně `Denní výkon`.
- V HEN lze při plánování Trasy dňa použít režim **„Pridať podľa rozvrhu“**, tj. systémový výběr položek podle Rozvrhu (a volitelně Okruhu). To je funkčně blízké našemu cíli automaticky připravit OS/okruhy dne pro dispečera.
- HEN při plánování hlídá duplicity položek pro stejný den/útvar, což je analogické našemu pravidlu nevytvářet duplicitní OS pro stejný den.
- HEN rozlišuje původ položek v realizaci (`Podľa rozvrhu`, `Mimo rozvrh`, `Presunuté na iný deň`), což je dobrá inspirace pro evidenci původu OS a přegenerování/přeplánování v RP.

Shrnutí analogie pro PP/RP:
- HEN ukazuje, že „generování podkladu k plánu realizace“ není jen vytvoření seznamu položek, ale kombinace **zdrojové entity + kalendáře/rozvrhu + geografického kontextu + seskupení do plánovací jednotky**.
- Pro náš návrh to potvrzuje směr generování OS jako předpřipraveného podkladu pro finální plánování (ne jako finální plán samotný).
- Zároveň to upozorňuje na kritické oblasti k dopracování: samostatnost vazeb, historizace, odvozené atributy okruhů a pravidla pro položky mimo rozvrh / přesuny.

## 3. Navrhované workflow (dle cílového konceptu)

### 3.1 Standardní workflow generování OS

1. Vstupem jsou RPO a související vazby/číselníky potřebné pro plánovací kontext:
   - vazba na rozvrh (kalendářní dny),
   - vazba na okruh (pokud existuje),
   - nádoby a lokace stanovišť,
   - zóna (pokud je k dispozici; typicky jako pomocný/odvozený údaj).
2. Generování se spouští automaticky podle konfigurace (frekvence, čas, období), případně manuálně oprávněným uživatelem.
3. Systém vybere RPO s účinnou vazbou na rozvrh, jehož kalendář obsahuje den v generovaném období.
4. Systém projde relevantní rozvrhy / kalendářní dny a pro vybraná RPO generuje OS a jejich lokace.
5. Podle vazby na okruh systém:
   - zařadí OS do existujícího okruhu dne, nebo
   - založí nový okruh dne.
6. Pokud je okruhu přiřazeno vozidlo, okruh dne se automaticky zařadí do denního výkonu (DV) vozidla.
7. Dispečer dostane připravená data (OS, lokace, okruhy dne, případně DV) jako **podklad pro finální operativní plánování**.

### 3.2 Pravidla a kontroly v průběhu generování

- Nutná podmínka: RPO musí mít vazbu na rozvrh a rozvrh musí obsahovat den v generovaném období.
- Pro daný kalendářní den rozvrhu vzniká OS a její lokace pouze jednou.
- Okruh dne obsahuje pouze OS jedné skupiny odpadu.
- Pokud chybí stanoviště nádoby, lokalizace OS se doplní fallbackem z místa realizace RPO.
- Zóna slouží primárně jako pomocný/filtrační kontext; sama o sobě není nutnou podmínkou vzniku OS.
- Doporučeno evidovat původ/povahu OS (např. dle rozvrhu / mimo rozvrh / přegenerováno / přeplánováno) pro audit a operativní práci dispečera.
- Pokud už pro RPO a den existuje OS, systém provede přegenerování:
  - původní OS deaktivuje a označí jako přegenerovanou,
  - založí novou OS a její lokace,
  - přegenerované OS a vazby se následně nočně čistí.

### 3.3 Manuální workflow

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
- Manuální workflow má sloužit i jako nástroj pro operativní doplnění podkladu k plánu (analogicky k položkám „mimo rozvrh“ v HEN), nikoli pouze k opravě technických chyb.

### 3.4 Výsledné scénáře podle vazeb RPO

- `RPO v rozvrhu = ANO`, `RPO v okruhu = ANO`, `okruh má vozidlo = ANO`:
  - OS jsou zařazeny do okruhu dne a okruh dne do DV.
- `RPO v rozvrhu = ANO`, `RPO v okruhu = ANO`, `okruh má vozidlo = NE`:
  - OS jsou zařazeny do okruhu dne, který čeká na zaplánování do DV.
- `RPO v rozvrhu = ANO`, `RPO v okruhu = NE`:
  - vzniknou OS a lokace, dispečer je musí zařadit do okruhů dne.
- `RPO v rozvrhu = NE`:
  - OS a lokace se standardním generováním nevzniknou.

### 3.5 Dopad na roli dispečera

- Dispečer neřeší ruční zakládání všech OS, ale pracuje s předpřipravenou sadou dat.
- Jeho role se posouvá na kontrolu, finální sestavení okruhů dne a DV a řešení výjimek.
- Součástí role dispečera je i kontrola kvality podkladu (vazby na rozvrh/okruh, původ OS, chybějící lokace a fallbacky).
- Míra ruční práce závisí na kvalitě vazeb RPO -> Rozvrh -> Okruh a na přiřazení vozidel k okruhům.

## 4. Otevřené body / rozhodnutí k dopracování

### 4.1 Platnost rozvrhů a přiřazení (zásadní pro generování)

- Upřesnit datový model a pravidla pro `platnost rozvrhu` (v CK je uvedeno jako bod k dopracování).
- Upřesnit práci s `platností přiřazení RPO -> Okruh` a `RPO -> Rozvrh`:
  - v HEN chybí časový údaj „od kdy“,
  - v PP se zavádí vlastní platnost,
  - v Etapě 1 má být platnost nepovinná.
- Rozhodnout, jak se platnost promítne do generování OS při změnách v průběhu období (od kdy přegenerovat, co ponechat).
- Potvrdit, zda má být vazba na rozvrh v PP/RP chápána striktně jako samostatná (dynamická) vazba i v integračním kontraktu, nebo zda bude přenášena ve zploštěné podobě.

### 4.2 Pravidla pro změny v HEN a dopad na přegenerování OS

- HEN umožňuje editaci RPO i do minulosti; je potřeba potvrdit:
  - zda generování/přegenerování sahá i do minulých dnů,
  - jaké období je chráněné před automatickými zásahy,
  - jak se budou evidovat a auditovat přegenerované OS.
- Přidání/odebrání RPO z okruhu v HEN nevytváří novou verzi RPO:
  - je nutné definovat, podle čeho RP bezpečně pozná změnu a spustí odpovídající přegenerování.
- Naopak některé materiální změny v HEN verziují PZ/RPO (např. změna rozvrhu / typu nádoby); je nutné popsat, jak se tato historizace propíše do pravidel přegenerování na straně PP/RP.

### 4.3 Validace business pravidel při generování

- HEN nevaliduje logiku přiřazení RPO do okruhu; potvrdit, která validační pravidla musí být v RP/FLW povinná.
- HEN nevaliduje skupiny/druhy odpadu v okruhu; dopracovat konkrétní validaci na straně FLW/RP:
  - kdy generování zablokovat,
  - kdy pouze varovat dispečera.
- Dodefinovat validaci a původ hodnoty `zóna` na RPO (odvození z adresy stanoviska vs. ruční zásah), včetně reakce na nesoulad mezi adresou stanoviska a přiřazenou zónou.
- Potvrdit finální pravidla pro vznik „fiktivního okruhu dne“ a kdy má být použité jako default.

### 4.4 Governance manuálního generování

- Potvrdit oprávnění a role pro manuální spuštění generování (CK uvádí doporučení omezit počet uživatelů).
- Dodefinovat provozní pravidla:
  - kdo může spouštět generování za provozovnu,
  - zda bude potřeba schvalování / logování spuštění,
  - limity rozsahu (období, počet rozvrhů) kvůli výkonu a riziku kolizí.
- Potvrdit, jak se v UI a auditu rozliší manuálně vytvořené OS „mimo rozvrh“ od OS vzniklých standardním generováním.

### 4.5 Paralelní plánování HEN vs. RP

- Otevřený bod z CK: nutnost paralelního plánování v HEN i RP.
- Rozhodnutí je kritické pro workflow generování OS, protože ovlivňuje:
  - kdo je finální autorita pro denní plán,
  - jak řešit kolize změn po vygenerování OS,
  - jak nastavit synchronizaci a přepisování dat mezi RP a HEN.
- Upřesnit mapování mezi „Trasa dňa“ v HEN a „Denní výkon“ v RP/PP při scénářích přeplánování a položek mimo rozvrh.

### 4.6 Technické a provozní parametry generování k potvrzení

- Výchozí konfigurace automatu:
  - frekvence a čas spuštění,
  - generované období dopředu,
  - zapnutí/vypnutí fiktivního okruhu dne.
- Rozsah přenášených dat z HEN pro rozvrh:
  - pouze seznam dní vývozu (flattened),
  - nebo i detailní parametrizace rozvrhu (pro audit/rekonstrukci logiky).
- Chování při opakovaném spuštění ve stejném dni:
  - jak agresivně přegenerovávat,
  - co dělat s již ručně upravenými okruhy dne / DV.
- Definovat technický klíč/strategii prevence duplicit (min. kombinace RPO + den + typ vzniku/původ OS).
- Definovat očekávané výstupy a monitoring běhu:
  - minimální sada metrik (počet RPO, OS, lokací, okruhů, chyb, varování),
  - způsob upozornění při neúspěšném nebo částečně úspěšném běhu.
