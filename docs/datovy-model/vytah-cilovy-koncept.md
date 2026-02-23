# Strukturovaný výtah: Cílový koncept CS MP SK

> **Zdroj:** Příloha č. 3_Cílový koncept.docx (verze 0.10, 3.10.2025)
> **Zaměření výtahu:** Datový model a integrace mezi systémy
> **Vytvořeno:** 2026-02-23

---

## Manažerské shrnutí
> Kapitola CK: Manažerské shrnutí

Cílem projektu je rozšířit stávající produkty **PP** (Pasport), **RP** (RoadPlan) a **FoB** (FleetwareOnBoard) o podporu pro cyklické svozy odpadu malých nádob.

Pět klíčových oblastí:
1. **Pasport** jako komplexní datová základna statických údajů — nádoby, stanoviště, okruhy, rozvrhy, zóny.
2. **RoadPlan** — plánování denních tras vozidel s podporou cyklických svozů a správou velkého množství objednaných služeb.
3. **FleetwareOnBoard** — mobilní podpora řidičů, zobrazení stanovišť nádob, vizualizace stavu obslouženosti v reálném čase.
4. Poskytování dat o realizaci zpět do **Helios Nephrite** (HEN) pro přípravu realizačních dokladů a zákonnou evidenci odpadu.
5. **Strategická optimalizace** okruhů v RoadPlan (budoucí etapa).

Projekt je navržen v etapách s důrazem na přístup **MVP** (Minimum Viable Product). První etapa dodá základní funkčnost pro plánování CS.

---

## Business procesy
> Kapitola CK: Business procesy

### Přehled identifikovaných procesů
> Kapitola CK: Přehled identifikovaných procesů

[Obr.: Schematické znázornění práce v systémech zapojených do cyklického svozu]

Identifikované procesy podporované systémy:
1. Správa smluvních údajů
2. Pasportizace nádob a stanovišť
3. Příprava okruhů a rozvrhů
4. Plánování denních výkonů CS
5. Realizace naplánované trasy CS
6. Monitoring realizace CS
7. Vyhodnocení realizace CS
8. Příprava realizačních dokladů
9. Strategická optimalizace CS
10. Zobrazení informací pro veřejnost
11. Navázaná kontrolní činnost / analýzy / reporting

### Zapojení systémů
> Kapitola CK: Zapojení systémů

[Obr.: Rozdělení procesů mezi systémy]

Klíčové rozhodnutí — příprava okruhů a vazeb RPO na rozvrhy by měla postupně přejít pod ekosystém FLW, protože:
- Nejde o primární smluvní data, ale odvozená data pro potřeby plánování.
- FLW je prostředí, ve kterém pracuje zodpovědná role (dispečer), a proces navazuje na strategickou optimalizaci.
- FLW nabízí komfortnější prostředí pro práci s okruhy (mapa).
- Princip jednoho zdroje pravdy (SoT) — jeden systém má primární zodpovědnost za danou entitu.

**HEN** by příslušná data měl nadále k dispozici pro potřeby filtrací a přípravy realizačních dokladů — zajištěno integrací.

Správa číselníku **rozvrhů** a generování **kalendářních svozů** zůstává v kompetenci HEN. Na straně FLW PP bude řízena jen vazba **RPO** na rozvrh.

---

## Doménový model
> Kapitola CK: Doménový model

PP již eviduje **nádoby** a **stanoviště** na úroveň detailu požadovanou pro plánování CS, včetně většiny entit z domény smluv/objednávek. Po rozšíření o vazby na **okruhy** a **rozvrhy** se PP stane kompletní datovou základnou statických vstupů pro operativní plánování v RP.

RP se zaměří na operativní plánování tras, sledování realizace a její vyhodnocení. Součástí RP bude také podpora pro strategické plánování, jehož výstupem budou datové podklady transformovatelné na standardní okruhy — ty budou řízeně přeneseny do PP a HEN.

### Doména smluv / objednávek
> Kapitola CK: Doména smluv / objednávek

Většina entit z domény objednávek pro CS je již zavedena v PP. Nové entity:

| Entita | Popis | Vazba |
|---|---|---|
| **Zóna** | Geografická oblast | Nepovinná vazba z RPO |
| **Okruh** | Sdružuje položky objednávek obsluhované společně | Nepovinná vazba z RPO |
| **Rozvrh** | Předepisuje konkrétní kalendářní dny obsluhy nádoby | Nepovinná vazba z RPO |
| **Kalendář** | Konkrétní kalendářní den obsluhy | Vazba na daný rozvrh |

Nově vznikne vazba mezi **druhem odpadu** a **skupinou odpadu** — zjednodušení výběru RPO pro denní plánování i strategickou optimalizaci. Nová vazba řídí nastavení stávající přímé vazby nádoby na skupinu odpadu.

[Obr.: Entity z domény smluv / objednávek]

RP disponuje vlastním datovým modelem objednávek (specifický pro nepravidelnou / kontejnerovou dopravu) — ten zůstává beze změny. Pro CS nebude RP rozšířen o kompletní model objednávek; datový model bude postaven tak, aby v nezbytné míře podporoval potřebnou funkčnost.

RP bude primárně pracovat s entitou **objednaná služba** — instance vznikající na základě **RPO**, která se má obsloužit v konkrétní den.

Minimální sada atributů objednané služby v RP:
- číslo RPO
- konečný příjemce
- adresa a souřadnice místa realizace
- počet nádob
- typ nádoby
- druh a skupina odpadu
- okruh
- rozvrh
- kalendářní dny svozu

### Doména objednaných služeb a plánů
> Kapitola CK: Doména objednaných služeb a plánů

Z RPO cyklického svozu jsou na základě vazeb na **rozvrh** a **okruh** generovány **objednané služby** (stávající entita RP).

Nová entita: **okruh dne** — soubor objednaných služeb, které v rámci denního výkonu vystupují společně. Vzniká v procesu generování objednaných služeb. Typ položky denního výkonu bude rozšířen o hodnotu „okruh dne".

[Obr.: Entity z domény plánování]

---

## Návrh etapizace
> Kapitola CK: Návrh etapizace

Etapizace respektuje logické návaznosti dílčích funkčních celků a sleduje cíl dodat v přiměřeném čase životaschopný produkt (MVP).

### Etapa 1
> Kapitola CK: První etapa

Zaměření na elementární funkčnost pro plánování CS.

**Pasport (PP):**
- Datová základna statických údajů
- K nádobám a stanovištím přibývá evidence **okruhů**, **rozvrhů** a příslušných vazeb
- Funkčnost pro náhledy a práci s novými entitami

**RoadPlan (RP):**
- Generování **objednaných služeb** seskupených do **okruhů dne**
- Sestava denních výkonů, celý životní cyklus DV vč. monitoringu a vyhodnocení
- Rozšíření komunikačního protokolu s FOB
- Správa objednaných služeb CS; podpora RPO v rozsahu potřebném pro funkčnost (plnohodnotná práce s RPO zůstává v PP / HEN)
- API rozhraní pro export podkladů pro realizační doklady do HEN

**FleetwareOnBoard (FOB):**
- Prezentace tras cyklického svozu
- Příjem tras a jejich průběžných aktualizací
- Zobrazení stanovišť nad mapou se stavem obslouženosti v reálném čase
- Podpora offline provozu při výpadku konektivity

### Etapa 2
> Kapitola CK: Druhá etapa

**Strategická optimalizace** — celý životní cyklus strategického plánování (sběr dat, analýza, zavedení nových okruhů do praxe).

**Pasport (PP):**
- Kompletní správa okruhů vč. zakládání nových
- Správa přiřazení RPO k okruhům, zónám, rozvrhům
- Změna primárního systému (SoT) pro příslušné entity z HEN na PP
- Pohled na nezařazené RPO
- Rozšíření typů nádob o atributy pro výpočty kapacity (váha, objem)

**RoadPlan (RP):**
- Rozšíření práce s mapou (přesun služby na jiný okruh dne z mapy, deaktivace OS)
- Vylepšení výpočtu vzdáleností a časů při sestavě itineráře

**FleetwareOnBoard (FOB):**
- Navigační funkce v modulu CS
- Rozšíření informací ke stanovišti
- Aktivní odesílání informací o odbavení/vynechání činností
- Hlášení incidentů z FOB
- Zadávání objemového využití vozidla (podpora sběru dat pro strategickou optimalizaci)

**Integrace:**
- Přenos okruhů a vazeb z PP do HEN (obrat směru toku)
- Zastavení odpovídajících přenosů z Etapy 1

### Další rozvoj
> Kapitola CK: Další rozvoj

- Běžný rozvoj reflektující business požadavky a zpětnou vazbu
- Zapojení **FLW PublicWeb** — aplikace pro veřejnost: vizualizace nádob na mapě se stavem služby, zobrazení nejbližšího data svozu

---

## Aplikace Pasport
> Kapitola CK: Popis rozšíření jednotlivých aplikací → Aplikace Pasport

### Přehled
> Kapitola CK: Aplikace Pasport → Přehled

PP se rozšiřuje z evidence nádob na komplexní datovou základnu pro operativní i strategické plánování svozů. Klíčové změny:

- Nový fokus na obrazovku **Revize položky objednávky (RPO)** — zobrazí umístění RPO na mapě, zařazení do **Rozvrhů**, **Okruhů** a **Zón**
- Nádoby dědí hodnoty atributů (okruh, rozvrh, zóna) z navázané RPO
- Rozšířené filtry umožní filtrování přes nové atributy
- PP podporuje budování datové základny nezávisle na stavu pasportizace nádob nebo evidenci předmětů smluv v HEN

[Obr.: Dopady na moduly aplikace Pasport]

**Dva klíčové scénáře použití:**
1. **Kontrola položek objednávek** — zobrazení RPO včetně zařazení do Rozvrhu, Okruhu a Zóny; zobrazení míst realizací a stanovišť nádob na mapě
2. **Editace zařazení RPO do okruhů** — důležitost roste v dalších etapách; uživatel s oprávněním spravuje číselník Okruhů a Zón a přiřazení RPO k nim

**Zdroj pravdy (SoT):**
- **Etapa 1**: Kompletně všechna data pro RPO a vazby na entity pochází z **HEN**
- **Postupně** (Etapa 2+): Primárním zdrojem pro správu rozvrhů, okruhů a vazeb na RPO se stává **PP**

### Správa číselníků
> Kapitola CK: Aplikace Pasport → Správa číselníků

#### Rozvrhy
> Kapitola CK: Aplikace Pasport → Rozvrhy

**Rozvrh** řídí konkrétní kalendářní dny, na které má být vytvořena objednaná služba vznikající z RPO. Je stavebním kamenem pro vytvoření Trasy dne v HEN.

| Aspekt | Detail |
|---|---|
| **Zdroj pravdy** | HEN — kompletní správa rozvrhů probíhá v HEN |
| **Scope v PP** | Pouze zobrazení (read-only) ve Správě systému |
| **Vazba na provozovnu** | Každá provozovna může mít vlastní sadu rozvrhů |
| **Klíčový atribut** | Platnost rozvrhu — bude upřesněn při přípravě finálního řešení |
| **UI** | Přehled rozvrhů s vybranými atributy + pravý panel s kalendářními dny |

[Obr.: Obrazovka Rozvrhy]

#### Druhy odpadu
> Kapitola CK: Aplikace Pasport → Druhy odpadu

Číselník slouží pro přehled druhů odpadů a nastavení vazby **Druh odpadu → Skupina odpadu**.

**Entita Skupina odpadu:**
- Slouží pro jednodušší výběr RPO při plánování (operativním i strategickém)
- Evidována per **Provozovna**
- Podporuje **mísitelnost** druhů odpadů — druhy, které se mohou svážet společně (např. 150101 a 200101)
- Procedura PP automaticky vyplní Skupinu odpadu na RPO dle nastavení číselníku pro daný Druh odpadu

**Speciální skupina "MIX" (Kombinovaný svoz):**
- Automaticky plněna na RPO, pokud je na Předmětu smlouvy HEN vyplněn atribut "Odpad kombinovaného zberu"
- Není přepisována procedurou PP pro standardní mapování Druh → Skupina
- Nelze ji nabídnout jako skupinu pro zadání k druhu odpadu

**Sdílení číselníku:**
- Číselník je společný pro PP a RP
- Nastavení vazby Druh ↔ Skupina možné pouze v PP; RP čerpá read-only

**Realizace:** navržena do Etapy 1

#### Typy nádob
> Kapitola CK: Aplikace Pasport → Typy nádob

Existující číselník (zdroj: ERP HEN/WINX, read-only) se rozšiřuje o editovatelný atribut:

| Nový atribut | Popis |
|---|---|
| **Čas obsluhy typu nádoby** | Číselná hodnota ve vteřinách — průměrná doba obsluhy jedné nádoby |

**Využití nového atributu:**
- Operativní plánování v RP: výpočet výchozí doby trvání Okruhu dne
- Strategické plánování: výpočet výchozí doby trvání Okruhu dne + výpočet váhy a objemu odpadu

#### Okruhy
> Kapitola CK: Aplikace Pasport → Okruhy

**Okruh** je stavebním kamenem pro operativní plánování — tvorbu Okruhů dne jako součást Denního výkonu RP.

| Aspekt | Etapa 1 | Etapa 2 |
|---|---|---|
| **Zdroj pravdy** | HEN | **PP** |
| **Scope** | Import z HEN do DB PP + uložení vazby RPO↔Okruh | Zakládání, editace, deaktivace Okruhů |
| **Vazba na provozovnu** | Ano | Ano |
| **UI číselníku** | Zobrazení + zadání pravidelného vozidla pro okruh | Rozšíření o CRUD operace |
| **Editace vazby RPO↔Okruh** | Ne (read-only) | Ano — změna přiřazení Okruhu na RPO |

**Etapa 1 — detail:**
- Uložení Okruhů z HEN do DB PP
- Uložení vazby RPO↔Okruh
- Uživatel může zadat konkrétní **vozidlo** pravidelně obsluhující daný okruh → usnadní generování objednaných služeb (OS lze zařadit do Okruhu dne i Denního výkonu vozidla)

**Etapa 2 — detail:**
- Mazání okruhu nahrazeno **deaktivací** s podporou filtru pro zobrazení jen aktivních okruhů

[Obr.: Obrazovka Okruhy]

#### Zóny
> Kapitola CK: Aplikace Pasport → Zóny

**Zóna** geograficky sdružuje Předměty smluv HEN do celků. Pomáhá při filtraci pro plánování.

| Aspekt | Detail |
|---|---|
| **Zdroj (Etapa 1)** | HEN |
| **Scope Etapa 1** | Import Zón z HEN do DB PP + uložení vazby RPO↔Zóna |
| **Kardinalita** | RPO : Zóna = **1:1** |
| **Vazba na provozovnu** | Ano |
| **Využití v PP** | Evidence, filtrování, zobrazení na mapě |
| **Využití v RP** | Výběr RPO podle zón → generování Objednaných služeb |

### Revize položky objednávky (RPO)
> Kapitola CK: Aplikace Pasport → Revize položky objednávky

**RPO** je jednoznačný podklad o způsobu obsluhy daného místa realizace nebo navázaných nádob.

- **Zdroj**: Předměty smluv HEN — bez manuálního zakládání nebo změny RPO (ani do budoucna)
- **Výjimka (Etapa 2)**: Možnost změny přiřazení Okruhu nebo Zóny

#### Přiřazení skupiny odpadu položce objednávky
> Kapitola CK: Aplikace Pasport → Přiřazení skupiny odpadu položce objednávky

**Mechanismus přiřazení:**
1. Systém při synchronizaci RPO s předměty smluv HEN vyplní Skupinu odpadu na RPO dle mapování Druh odpadu → Skupina odpadu (spravováno v číselníku Druhy odpadu)
2. Při spuštění CS proběhne **migrace** — mapování skupin odpadů na existující RPO (alternativně zajistí první synchronizace)

**Business pravidla pro manuální změnu skupiny odpadu na RPO:**
- Pokud se nepodařilo namapovat skupinu dle Druhu odpadu → uživatel s právem může změnit skupinu nebo nastavit MIX
- Pokud Druh odpadu není vyplněn → uživatel může vybrat jakoukoli skupinu včetně MIX
- Pokud je povolena změna odpadu na předmětu smlouvy → synchronizace zajistí update skupiny na RPO

**Vazba na RP:** Jeden Okruh dne v RP obsahuje Objednané služby pouze **jedné skupiny odpadu**.

#### Vazba RPO na okruhy, rozvrhy a zóny
> Kapitola CK: Aplikace Pasport → Vazba revize položky objednávky na okruhy, rozvrhy a zóny

**Kardinalita vazeb:**

| Vazba | Kardinalita |
|---|---|
| RPO → Okruh | **M:N** (k jedné RPO více okruhů) |
| RPO → Rozvrh | **M:N** (k jedné RPO více rozvrhů) |
| RPO → Zóna | **1:1** |

**Účel uložení vazeb v PP:**
- Komplexní informace o RPO a nádobách
- Kontrola zařazení RPO (nádob) do okruhů a správné vazby na rozvrh
- V dalších etapách: PP jako místo správy okruhů a zařazení RPO; RP pro práci s objednanými službami, trasami a denními výkony
- Podklad pro generování objednaných služeb pro strategickou optimalizaci tras v RP

#### Rozšíření obrazovky RPO
> Kapitola CK: Aplikace Pasport → Rozšíření obrazovky RPO

Obrazovka RPO se rozšiřuje o:
1. **Mapa** — zobrazení RPO nebo stanovišť nádob
2. **Panel "Přiřazení okruhů"** — záložka v pravém panelu s informací o přiřazení vybrané RPO (Etapa 1: pouze zobrazení, bez editace)
3. **Panel objednaných služeb** vybrané RPO
4. **Pokročilý filtr** — nové skupiny filtrů:
   - Okruh
   - Rozvrh
   - Rozhodující kalendářní den (pohled na budoucí/minulý stav)
   - Aktuálně platné (pohled na aktuální stav)

[Obr.: Rozložení obrazovky Položky objednávek]

#### Geokódování míst realizací a zobrazení RPO v mapě
> Kapitola CK: Aplikace Pasport → Geokódování míst realizací a zobrazení RPO v mapě

**Logika zobrazení bodu na mapě:**
1. Pokud RPO má vazbu na nádobu → zobrazí se souřadnice **stanoviště nádoby**
2. Pokud RPO nemá vazbu na nádobu → zobrazí se geokódovaná souřadnice **místa realizace RPO**

**Geokódování místa realizace:**
- Automaticky při synchronizaci RPO, pokud se změní adresa místa realizace
- Body na mapě vizuálně odlišeny (Místo realizace vs. Stanoviště nádoby)

[Obr.: Metodika použité souřadnice]

#### Volba zobrazení pozic RPO na mapě
> Kapitola CK: Aplikace Pasport → Volba zobrazení pozic RPO na mapě

Třístavový filtr pro zobrazení RPO na mapě:
1. **Všechny pozice** — souřadnice dle MR nebo ST
2. **Jen dle stanovišť nádob (ST)**
3. **Jen dle geokódingu místa realizace (MR)**

### Panel Přiřazení okruhů
> Kapitola CK: Aplikace Pasport → Panel Přiřazení okruhů

Záložka informuje o stavu přiřazení RPO do okruhu a rozvrhu.

**Platnost přiřazení:**
- Na straně HEN neexistuje časový údaj definující, od kdy je RPO zařazena do Okruhu → PP zavádí vlastní **časovou platnost** přiřazení
- Obdobně pro Rozvrh
- Platnost v Etapě 1 bude **nepovinná**
- Rozvrhy jsou typicky platné na jeden kalendářní rok; práce s platností bude upřesněna s D2B

**Zóna na RPO:**
- RPO přebírá zónu z Předmětů smluv HEN
- Nádoby a stanoviště dědí informaci o zóně z RPO

[Obr.: Panel přiřazení okruhů]

**Etapa 2 — rozšíření panelu:**
- Deaktivace současného zařazení RPO do okruhu a rozvrhu
- Nové zařazení RPO do okruhu a rozvrhu

### Pohled na nezařazené RPO a jejich zařazení
> Kapitola CK: Aplikace Pasport → Pohled na nezařazené RPO a jejich zařazení

Samostatná obrazovka plánovaná do dalších etap. Umožní:
- Zobrazit RPO existujících okruhů a zón
- Zobrazit nezařazené RPO (do okruhů, rozvrhů nebo zón)
- Založení nového okruhu nebo zóny
- Formulář pro přiřazení RPO

[Obr.: Obrazovka Nezařazené položky objednávek]

### Nádoby
> Kapitola CK: Aplikace Pasport → Nádoby

Nádoby dědí informace z navázané RPO — zařazení do okruhu, rozvrhu a zóny (plánováno pro další etapy).

#### Skupina odpadu nádoby
> Kapitola CK: Aplikace Pasport → Skupina odpadu nádoby

**Pravidla pro skupinu odpadu na nádobě:**
- Hodnotu skupiny odpadu na nádobě **řídí RPO** — nesmí dojít k nekonzistenci
- Skupina odpadu na nádobě lze zadat manuálně **pouze pokud**:
  - Nádoba nemá vazbu na RPO, nebo
  - RPO nemá skupinu odpadu definovánu
- Při navázání nádoby na RPO → po upozornění uživatele proběhne **přepis** skupiny odpadu na nádobě
- Stejné chování implementováno v mobilní aplikaci **FLW Picker**

#### Zobrazení zařazení na nádobě
> Kapitola CK: Aplikace Pasport → Zobrazení zařazení na nádobě

Informace o okruhu, rozvrhu a zóně z RPO se přenáší i na entitu Nádoby — vizualizace obdobná jako u RPO.

### Validace změn
> Kapitola CK: Aplikace Pasport → Validace změn a zákaz zpětných úprav

Validace životního cyklu RPO a řazení do okruhů, rozvrhů a zón nejsou součástí tohoto konceptu. Pro jednotlivé etapy analýza a dohoda se zákazníkem určí konkrétní validační body.

### Dopady na mobilní aplikaci Picker
> Kapitola CK: Aplikace Pasport → Dopady na modul Pasport mobilní aplikace Picker

Rozšíření FLW Picker plánováno do **Etapy 2**:
- **Detail nádoby**: rozšíření o informace z RPO — přiřazení do okruhu, rozvrhu a zóny
- **Skupina odpadu nádoby**: analogické chování jako ve webové aplikaci PP

### Rozšíření PP do dalších etap
> Kapitola CK: Aplikace Pasport → Rozšíření uvažovaná do dalších etap

| Funkčnost | Popis |
|---|---|
| **Kompletní správa okruhů** | Zakládání nových okruhů ve Správě systému |
| **Změny přiřazení nádob** | Změna přiřazení nádob do okruhů a zón; změna kalendáře nádoby; evidence změn a dopad na plánování |
| **Pohled na nezařazené RPO** | Viz sekce Pohled na nezařazené RPO |

---

## Aplikace RoadPlan
> Kapitola CK: Popis rozšíření jednotlivých aplikací → Aplikace RoadPlan

Rozšíření aplikace RoadPlan o cyklický svoz (CS) je realizováno formou **separátních záložek** v rámci stávajících modulů. Stávající funkčnost pro kontejnerovou/nepravidelnou dopravu zůstává beze změny.

Dotčené moduly:
- Správa objednávek
- Správa objednaných služeb
- Plánování denních výkonů a přehled denních výkonů
- Časové využití vozidel
- Monitoring realizace denních výkonů

[Obr.: Dopady na moduly aplikace RoadPlan]

### Práce s entitou Okruh dne
> Kapitola CK: Aplikace RoadPlan → Práce s entitou okruh dne

**Okruh dne** je nová entita sdružující jednotlivé **Objednané služby (OS)** pro účely denního plánování. Uživatel pracuje primárně s touto entitou.

Klíčové vlastnosti:
- Název okruhu dne = kombinace názvu okruhu + datum (konkrétní podoba bude upřesněna v detailní analýze).
- Stav okruhu dne při realizaci je nastavován na základě informací příchozích z FOB.
- Okruh dne obsahuje pouze OS jedné skupiny odpadu (business pravidlo).

**Výpočet plánovaného času realizace okruhu dne v rámci DV:**

| Veličina | Vzorec | Popis |
|---|---|---|
| TN | konfigurace | Čas na obsluhu daného typu nádoby RPO |
| TOS (trvání obsluhy jedné OS) | TN × počet nádob v RPO | Čas obsluhy jedné objednané služby |
| TOD (trvání obsluhy okruhu dne) | ∑ TOS daného okruhu dne | Součet všech OS v okruhu |
| TDV (čas trvání denního výkonu) | ∑ TOD + ∑ TBP + ∑ TS | TOD + bezpečnostní přestávky + servisní výkony |

Pro výpočet času přejezdu je možné využít některý z bodů v rámci okruhu (způsob určení bodu bude řešen v detailní analýze).

### Správa objednávek
> Kapitola CK: Aplikace RoadPlan → Správa objednávek

Správa objednávek CS se odehrává v aplikaci PasPort (modul Položky objednávek). Z RP je realizován proklik do PP. Sdílené UI platformy Fleetware a jednotná správa identit a práv.

### Správa objednaných služeb
> Kapitola CK: Aplikace RoadPlan → Správa objednaných služeb

Separátní modul pro přehled **Objednaných služeb (OS)** cyklických svozů, analogický k OS nepravidelné dopravy.

Atributy v přehledu (konkrétní seznam určí detailní analýza): mj. informace o okruhu a rozvrhu.

Dostupné akce:
- Filtrační kritéria pro vyhledávání (nenaplánované, nerealizované OS apod.)
- Editace: změna data a času realizace (pro dosud nenaplánované OS), poznámka

Detail vybrané OS obsahuje:
- Seznam odpovídajících nádob včetně stanoviště
- Detail související položky objednávky (elementární údaje)
- Proklik do PasPortu pro kompletní údaje o objednávce

### Plánování denních výkonů
> Kapitola CK: Aplikace RoadPlan → Plánování denních výkonů a přehled denních výkonů

Nová záložka pro plánování **Denních výkonů (DV)** cyklického svozu (stávající plánování DV zůstává beze změny).

[Obr.: Rozložení obrazovky Plánování denních výkonů pro CS]

- Uživatel plánuje pro vybranou provozovnu a den.
- Plánování probíhá primárně přes **Okruhy dne**, které sdružují OS a jejich lokace.
- Okruh dne vzniká buď automaticky (při generování OS dle příslušnosti k okruhu), nebo manuálně.
- **Všechny plánované OS musí být zařazeny v nějakém okruhu dne.**
- Okruh dne je základní stavební prvek DV pro CS (analogie: OS u ostatních druhů dopravy).
- Zobrazení DV a grafu DV je přes okruhy dne; okruhy dne lze přesouvat mezi vozidly.
- V detailu okruhu dne lze jednotlivé OS/lokace přesouvat mezi okruhy dne.
- **Pokud okruh dne má při vzniku přiřazené vozidlo, je automaticky zařazen do DV tohoto vozidla.**

### Okruhy dne a objednané služby — datový model
> Kapitola CK: Aplikace RoadPlan → Okruhy dne a objednané služby

- Řádky v okruhu dne = jednotlivé **Objednané služby (OS)** — dále nedělitelné.
- OS může mít **1 až n lokací**.
- Každá **Lokace OS** má minimálně tyto atributy:
  - příslušnost k OS
  - souřadnice
  - typ nádoby
  - četnost
  - skupina odpadu
  - počet nádob
- Uživatel přesouvá OS mezi okruhy dne; **nelze přesouvat jednotlivé lokace** (nepočítá se s dělením OS).
- Pokud nádoba v rámci OS nemá stanoviště, jako lokace se doplní souřadnice **místa realizace (MR)** z položky objednávky (1–n nádob). V rámci jedné OS tak může být kombinace lokací ze stanovišť a MR.

[Obr.: Obrazovka pro úpravu okruhu dne]

### Nastavení likvidačního místa
> Kapitola CK: Aplikace RoadPlan → Nastavení likvidačního místa

- Okruh dne má nastavené **likvidační místo (LM)** dle OS — zařazeno na konec okruhu dne.
- LM je na okruhu dne automaticky dle nastavení na OS.
- Pokud je v jednom okruhu dne více LM, jsou uvedena všechna.
- V rámci editace dispečer označí, která LM budou zahrnuta do DV (do FOB se přenášejí pouze vybraná LM).
- Mezi okruhy dne nebo za ně lze vkládat i přestávky a servisní výkony.

### Úprava denního výkonu vozidla
> Kapitola CK: Aplikace RoadPlan → Úprava denního výkonu vozidla

K dispozici dva pohledy:

**Pohled přes okruhy dne:**
- Rozložení po okruzích dne — lze měnit pořadí, vkládat přestávky a servis.
- Práce s většími celky DV.
- Nelze manipulovat s jednotlivými OS a jejich lokacemi.

**Pohled přes objednané služby a jejich lokace:**
- Tabulkový přehled plánovaných OS a lokací.
- Přesun OS do jiných okruhů dne a DV.
- Deaktivace jednotlivých OS (zrušení realizace).
- Nelze pracovat s většími celky (celými okruhy dne).

Pro přepínání mezi pohledy je nutné uložit nebo zrušit provedené změny.

[Obr.: Plánování DV — pohled přes okruhy dne]
[Obr.: Plánování DV — pohled přes objednané služby a lokace]

### Monitoring realizace denních výkonů
> Kapitola CK: Aplikace RoadPlan → Monitoring realizace denních výkonů

Nová záložka pro CS. V mapě vždy jen DV jednoho vybraného vozidla.

**Souhrnné informace za vozidlo:**
- Začátek a konec realizace (+ porovnání s plánem)
- Počet najetých km (bez porovnání)
- Čas realizace (+ porovnání s plánem)
- Počet obsloužených nádob (+ porovnání s plánem)

### Potvrzení realizace DV
> Kapitola CK: Aplikace RoadPlan → Potvrzení realizace DV

**Business pravidlo — míra úspěšnosti realizace okruhu dne:**
- Pro CS se předpokládá, že běžně nebude obslouženo 100 % OS.
- Barevné označení dle % obsloužených OS:
  - Zelená: splněno (>= 90 %)
  - Oranžová: částečně splněno (60–90 %)
  - Červená: nesplněno (< 60 %)

**Změna stavu obsluhy OS:**
- Uživatel může změnit stav z neobsloužené na obslouženou a naopak.
- Neobsloužené OS lze přeplánovat na jiný DV.
- **Při potvrzení realizace DV s neobslouženými OS** jsou tyto OS považovány za zrušené (systém předem upozorní).

### Časové využití vozidel
> Kapitola CK: Aplikace RoadPlan → Časové využití vozidel

Nová záložka pro CS. V pohledech Vozidlo za období a Časové využití vozidel za den: namísto počtu služeb uveden počet obsloužených nádob.

### Proces generování objednaných služeb
> Kapitola CK: Aplikace RoadPlan → Proces generování objednaných služeb

**Generování OS** a jejich lokací zajišťuje vznik OS, aby byly připraveny na straně RP pro finální plánování okruhů dne a DV vozidel.

Zdroj dat: **revize položek objednávek (RPO)** včetně nádob a lokací stanovišť navázaných na RPO.

Klíčové charakteristiky:
- Proces spouštěn **automaticky** (konfigurovatelná frekvence), dostupný i pro **manuální spuštění**.
- Výsledkem jsou připravené **OS a jejich lokace**, **Okruhy dne** a **Denní výkony**.
- Dispečer dostane připravenu sadu dat na finální plánování s ohledem na aktuální situaci flotily (dostupnost vozidel).
- **RP je zdrojem pravdy pro entitu Denní výkon** (Trasa dne v HEN) po spuštění CS na provozovně.

#### Automatické generování OS z okruhů a rozvrhů
> Kapitola CK: Aplikace RoadPlan → Automatické generování OS z okruhů a rozvrhů

**Konfigurace generování v RP:**

| Parametr | Popis |
|---|---|
| Frekvence a čas spuštění | Jak často a kdy se automatické generování spouští |
| Období vzniku OS | Časový rozsah, pro který se OS generují |
| Vznik fiktivního okruhu dne (ANO/NE) | Pokud ANO: do fiktivního okruhu se zařadí všechny OS bez okruhu, ale s vazbou na Rozvrh; dispečer pak fiktivní okruh "rozebere" do existujících okruhů dne. Pokud NE: vzniknou pouze OS a jejich lokace. |

**Podmínky pro vygenerování:**
- Nutná podmínka: Rozvrh musí obsahovat kalendářní den spadající do období, pro které se OS generují.
- Generování zajistí, že OS a její lokace vzniknou na daný kalendářní den rozvrhu **pouze jednou**.
- Pokud došlo ke změně na straně RPO a nádob RPO, funkčnost zajistí **deaktivaci OS** a jejich lokací nebo jejich doplnění o další lokace.

#### Metodika generování OS z okruhů a rozvrhů
> Kapitola CK: Aplikace RoadPlan → Metodika generování OS z okruhů a rozvrhů

Postup generování:

1. Systém najde RPO s rozvrhem obsahujícím kalendářní den od DNES() po kalendářní den určený konfigurací období.
2. Systém projde všechny aktivní rozvrhy.
3. Pro vybranou skupinu RPO začne generovat okruhy dne, OS a jejich lokace:
   - Pokud okruh dne z daného okruhu již existuje — nebude opakovaně zakládán, nově vygenerovaná OS se pouze zařadí do existujícího okruhu dne.
   - Jinak se založí nový okruh dne.
   - **Okruh dne obsahuje pouze OS jedné skupiny odpadu.**
   - Pokud je na okruhu nastaveno vozidlo — okruh dne se zařadí do DV vozidla.
4. Pokud pro RPO na daný den již existuje OS:
   - Stávající OS se **deaktivuje** a označí jako "přegenerovaná".
   - Založí se nová OS a její lokace.
   - **Přegenerované OS a jejich vazby systém každou noc trvale maže.**

#### Manuální spuštění generování OS z okruhů a rozvrhů
> Kapitola CK: Aplikace RoadPlan → Manuální spuštění generování OS z okruhů a rozvrhů

- Uživatel s právem může spustit generování manuálně (synchronní proces).
- Podmínky shodné s automatickým generováním; dispečer navíc může omezit období a generování na vybrané rozvrhy nebo okruhy.
- Doporučení: omezený počet uživatelů s tímto právem na provozovnu.

**Výstup po dokončení generování:**
- Počet vybraných RPO
- Počet vygenerovaných OS, lokací OS, okruhů

#### Manuální generování OS na výzvu z RPO
> Kapitola CK: Aplikace RoadPlan → Manuální generování OS na výzvu z RPO

Účel: vygenerovat OS a lokace pro RPO **bez vazby na Rozvrh** — pro RPO s frekvencí svozu "Výzva", reklamace a jiné mimořádné svozy.

- Uživatel vybere konkrétní jeden kalendářní den.
- Systém nabídne RPO dané provozovny, pro které na daný den neexistuje OS.
- **Pokud je vybraná RPO zařazena v okruhu, okruh se ignoruje** — vznikne OS bez zařazení do okruhu dne.

### Rozšíření RP do dalších etap
> Kapitola CK: Aplikace RoadPlan → Rozšíření uvažovaná do dalších etap

**Plánování DV — rozšířená práce s mapou:**
- Deaktivace/aktivace OS přímo v mapě
- Přesun OS na jiný okruh dne přímo z mapy
- Zobrazení většího počtu DV v mapě současně

**Monitoring realizace DV — rozšířená práce s mapou:**
- Změna stavu obsluhy přímo v mapě
- Přeplánování neobslouženého místa na jiný DV
- Zobrazení většího počtu DV v mapě současně

---

## Aplikace FleetwareOnBoard
> Kapitola CK: Aplikace FleetwareOnBoard

### Přehled a datové entity
> Kapitola CK: Přehled

FOB zobrazuje nový modul **Cyklický svoz**, který nahrazuje stávající modul Trasy a Itinerář (konfiguračně vypnut). Ostatní funkcionalita FOB zůstává beze změny.

FOB pracuje s následujícími entitami:
- **Denní výkon (DV)** — hlavní datový objekt stažený z RP, obsahuje okruhy, stanoviště a činnosti
- **Okruh** — max. 5 per DV, každý může mít stovky stanovišť
- **Stanoviště** — součet v rámci DV reálně cca 600, s rezervou do 1 000
- **Likvidační místo (LM)** — součást okruhu, zobrazuje se Adresa a Skupina odpadu; okruh může mít více LM nebo žádné
- **Přestávka** a **Servis** — činnosti v rámci DV

[Obr.: Dopady na moduly aplikace FOB]

### Komunikace a stažení denního výkonu
> Kapitola CK: Komunikace a stažení denního výkonu

**Protokol stažení DV z RP:**
1. Po spuštění FOB automaticky vyvolá dotaz na stažení aktuálního DV
2. Dotaz se opakuje dle konfiguračního parametru (perioda + počet opakování)
3. K dispozici je i manuální vyvolání stažení
4. Během stahování lze dialog skrýt tlačítkem "Dokončit na pozadí" — po dokončení je uživatel informován

**Výsledky stažení:**
- **Úspěch** — dialog nabídne "Prohlédnout denní výkon", automaticky otevře modul Cyklický svoz
- **Neúspěch** — dialog s možností "Zkusit znovu"
- **Žádný DV** — FOB zobrazí hlášku o nedostupnosti DV pro aktuální vozidlo

### Stavový automat okruhů a komunikace s RP
> Kapitola CK: Práce s okruhem

Akce nad okruhy: **Zahájit**, **Ukončit**, **Přerušit**, **Vynechat**, **Zařadit**. V rychlé volbě jsou dostupné jen akce relevantní pro aktuální stav.

**Stavový automat okruhu:**

| Stav | Dostupné akce | Odesílá se do RP? |
|---|---|---|
| **Výchozí** (bez aktivity) | Zahájit, Vynechat | Zahájit → ANO (zahájení/přihlášení na okruh); Vynechat → NE |
| **Zahájen** | Ukončit, Přerušit | Ukončit → ANO (ukončení/odhlášení z okruhu); Přerušit → ANO (informace o přerušení) |
| **Přerušen** | Zahájit, Vynechat | Zahájit → ANO (znovu přihlášení); Vynechat → NE |
| **Ukončen** | Zahájit | Zahájit → ANO (zrušení ukončení, okruh znovu aktivní) |
| **Vynechán** | Zařadit | Zařadit → NE (vrátí se do výchozího stavu) |

**Klíčové pravidlo:** Stav "Vynechat" a "Zařadit" se neodesílají do RP — zpracovávají se pouze lokálně ve FOB.

### Automatické přepínání režimů
> Kapitola CK: Automatické přepínání režimů

- Zahájení okruhu → FOB automaticky aktivuje režim **Jazda (101)**
- Odbavení přestávky → aktivace režimu **Bezpečnostní přestávka (102)**
- Odbavení servisu → aktivace režimu **Oprava (133)**
- Obousměrně: změna režimu na daný režim vyvolá automatické odbavení příslušné činnosti v DV

### Přestávka a Servis
> Kapitola CK: Přestávka a Servis

| Stav | Dostupné akce | Odesílá se do RP? |
|---|---|---|
| **Výchozí** | Odbavit, Vynechat | V Etapě 1 NE (ani odbavení, ani vynechání) |
| **Vynechán** | Zařadit | NE |

### Práce s mapou a odbavování stanovišť
> Kapitola CK: Práce s mapou

V Etapě 1 se používá pouze **Arcgis mapa (2D)** — bez navigace Sygic.

**Třístavový model odbavení stanovišť (barevné značení):**

| Barva | Stav | Popis | Akce |
|---|---|---|---|
| Červená | **Neodbaveno** (výchozí) | Stanoviště čeká na odbavení | Odbavit |
| Žlutá | **Odbaveno FOB** | Odbaveno lokálně ve FOB, čeká na potvrzení z RP | Zobrazeno "Odbaveno FOB" |
| Zelená | **Potvrzeno RP** | RP potvrdil odbavení | Zobrazeno "Odbaveno, Potvrzeno RP" |

**Způsoby odbavení stanoviště:**
1. **Ruční odbavení uživatelem** — perimetr jako konfigurační parametr FOB (na mapě znázorněn kružnicí)
2. **Automatické odbavení po aktivaci PTO** — perimetr jako konfigurační parametr; hromadná změna stavu všech stanovišť v nastaveném perimetru

**Komunikace s RP při odbavení:**
- Po každém odbavení ve FOB se **odešle zpráva o odbavení konkrétních stanovišť do RP**
- FOB obdrží zprávu z RP o potvrzení odbavení (zelená)
- **Počet obsloužených stanovišť v celé aplikaci FOB prezentuje až počet potvrzených odbavení z RP**

**Omezení v Etapě 1:**
- Nelze vzít zpět odbavení stanoviště z FOB — zpětnou změnu stavu může provést pouze RP
- Potvrzené stanoviště z RP nelze přes FOB vrátit do výchozího stavu

### Podpora pro offline provoz
> Kapitola CK: Podpora pro offline provoz

- FOB si musí pamatovat neodeslaná data při nedostupnosti serveru (přihlášení na okruh, ukončení okruhu, odbavení stanovišť)
- Data musí přežít i restart zařízení
- Odesílání se opakuje, dokud FOB nedostane **potvrzení z RP**
- Platnost: **pouze v rámci aktuálního dne** — následující den FOB neodeslaná data z minulého dne ignoruje

### Rozšíření FOB do dalších etap
> Kapitola CK: Rozšíření uvažovaná do dalších etap

| Rozšíření | Podstata |
|---|---|
| **Navigace Sygic** | Tlačítko navigace u LM, přepnutí na Sygic mapu |
| **Rozšíření informací na stanovišti** | Detail — rozpad nádob podle Typů nádob a Skupin odpadu; možnost vrátit odbavení |
| **Odesílat odbavení Přestávka/Servis** | Do RP se odešle zpráva o odbavení konkrétní činnosti |
| **Odesílat Vynechání okruhu** | Do RP se odešle zpráva o vynechaném okruhu |
| **Odbavení LM zvlášť** | Samostatné odeslání odbavení LM do RP |
| **Incident z FOB** | Odeslání incidentu nad stanovištěm — výběr Typu incidentu + poznámka |
| **Aktivace SWIN** | Ruční odbavení stanoviště aktivuje vstup ve vozidlové jednotce |
| **Zadání váhy s objemem** | Rozšíření o zadání objemu přes posuvník |

---

## Komunikace mezi systémy
> Kapitola CK: Komunikace mezi systémy

### Principiální schéma komunikace
> Kapitola CK: Principiální schéma komunikace

[Obr.: Principiální schéma komunikace mezi systémy HEN, RP, PP]

Nová/upravená komunikační API rozhraní:

| Rozhraní | Typ | Popis |
|---|---|---|
| **PP-sync-external-Helios** | stávající, rozšiřovaná | Přenos dat z MDB do PP; rozšíření o aktivní REST API komunikaci PP <-> HEN |
| **RP-sync-external-Helios** | stávající, rozšiřovaná | Přenos dat z MDB do RP a opačně; rozšíření o aktivní REST API komunikaci RP <-> HEN |
| **PP-sync-external-RoadPlan** | nová | Komunikace PP -> RP |
| **RP-sync-external-Pasport** | nová | Komunikace RP -> PP |

Komunikace prostřednictvím **MDB** je určena k postupnému zániku — bude nahrazena REST službami. V Etapě 1 je MDB zachována; nová integrační rozhraní budou API.

Alternativní řešení (integrační hub / ESB) budou prověřena v detailní technické analýze. Pro potřeby nabídky se uvažuje o přímých integracích.

### Integrační mapa (Etapa 1)
> Kapitola CK: Integrační mapa

[Obr.: Schéma entit vyměňovaných mezi systémy HEN, RP, PP, FLWW2]

**HEN -> PP:**
- Číselníky: **okruhy**, **rozvrhy** a **kalendáře**, **zóny**
- Vazby mezi **předmětem smlouvy** a entitami okruh, rozvrh, zóna

**PP -> RP:**
- Elementární údaje o **RPO** v rozsahu potřebném pro RP (generování OS, zobrazení základních informací)
- Základní údaje o **nádobách** a **stanovištích** (lokalizace OS, údaje o nádobě)
- **Okruhy**, **rozvrhy** a vazby na RPO
- **Skupiny odpadu**, vazba mezi skupinou odpadu a druhem odpadu
- Čas obsluhy na **typu nádoby**

**RP -> HEN:**
- Podklady pro realizační doklady: hlavička **denního výkonu** (trasa dne v kontextu HEN) a seznam zařazených RPO (předmětů smluv)

### Rozšíření plánovaná do dalších etap
> Kapitola CK: Rozšíření plánovaná do dalších etap

**Správa okruhů v PP:** Přenos okruhů a vazeb RPO-okruh/rozvrh z PP do HEN (obrat směru). Zastavení souvisejících přenosů z Etapy 1.

**Přechod z MDB na API:** Po vybudování a ověření API rozhraní budou entity z MDB přesměrovány na nové kanály.

**Strategická optimalizace:** Doplnění integrace RP -> PP (transformace výstupů optimalizace na standardní okruhy, rozvrhy a vazby na předměty smluv). Integrace zvoleného optimalizačního nástroje.

---

## Strategická optimalizace
> Kapitola CK: Strategická optimalizace

### Úvod a definice
> Kapitola CK: Úvod a Definice

**Strategické plánování (SP)** je proces v aplikacích RP a PP pro nalezení optimálního rozložení položek objednávek a jejich nádob do nové sady okruhů. Na rozdíl od operativního plánování nejde o rutinní proces — uživatelé vkládají vlastní invenci v každém kroku životního cyklu SP.

**Výsledky SP:**
- Nová sada okruhů s optimálním rozložením RPO
- Změna rozvrhů RPO (kompletní přepracování nebo dodatečná optimalizace)

**Klíčové entity strategické optimalizace:**
- **Strategický plán (SP)** — celkový výstup procesu strategické optimalizace
- **Strategická objednaná služba (SOS)** — generovaná z RPO dle frekvence svozu, vstupní jednotka pro optimalizaci
- **Strategický okruh dne (SOD)** — výstup optimalizačního algoritmu, seskupení SOS do denních okruhů
- **Verze optimalizace** — výsledek jednoho spuštění optimalizačního výpočtu; škálováno po provozovnách
- **Distanční matice** — matice vzdáleností/časů mezi uzly dopravní sítě

### Životní cyklus strategického plánování
> Kapitola CK: Životní cyklus SP

[Obr.: Životní cyklus strategického plánování svozu odpadu]

1. **Analýza dat svozu odpadu** — ověření kvality vstupních dat
2. **Příprava datové základny** — zadání parametrů do systému (vozidla, kapacity, lokality)
3. **Výpočet sady okruhů dne** — spuštění optimalizačního algoritmu → verze optimalizace
4. **Kontrola a manuální editace** — dispečer zpřesňuje navržené SOD
5. **Evidence nových okruhů** — přenos SOD do standardních okruhů PP
6. **Přenos do HEN** — synchronizace nových okruhů a vazeb s HEN

### Analýza dat svozu odpadu
> Kapitola CK: Analýza dat svozu odpadu

Ověřované oblasti: datová základna RPO a nádob, lokality, adresy a souřadnice, zóny, skupiny odpadu, typy nádob, počty nádob, frekvence vývozu.

**Analýza produkce odpadu:** Dva klíčové kapacitní faktory — hmotnostní limit (efektivní objem po lisování) a objemový limit (u lehkých odpadů se naplní prostor dříve než hmotnostní limit).

### Sběr dat — proces shromažďování
> Kapitola CK: Proces shromažďování relevantních dat

**Cílové parametry pro skupiny odpadu:**
- **Měrná hmotnost (hustota) odpadu** na 1 litr nádoby → definuje hmotnost odpadu typu nádoby za skupinu odpadu
- **Průměrné objemové zaplnění** typu nádoby pro skupiny odpadu

**Zdroje dat:**

| Systém | Vstup | Detail |
|---|---|---|
| **FOB** | Váha na LM | Řidič zadává hmotnost naváženou na LM k okruhu dne |
| **FOB** | Váha při ukončení okruhu | Pro ukončení okruhu musí řidič zadat váhu odpadu |
| **RP** | Uložení a editace | Hodnoty z FOB se ukládají k Okruhu dne RP; uživatel může editovat |
| **BI** | Analytický nástroj | Power BI nad datovým skladem — váhy výsypů, rozpočtené váhy z LM, objemová zaplněnost po lisování |

### Příprava datové základny
> Kapitola CK: Příprava datové základny

**Zdroje pro plánování (vozidla):**
- Vozidla vstupující do optimalizace, váhové a objemové kapacity
- Pracovní doba, umístění depa a likvidačních míst

**Položky vstupující do plánování:** Položky objednávek, Nádoby, Stanoviště

### Hlavní optimalizační kritérium
> Kapitola CK: Hlavní optimalizační kritérium

- Hlavní kritérium: **čas** — algoritmus hledá sadu okruhů tak, aby celková doba strávená vozidly při obsluze nádob byla minimální
- Využívá heuristické metody s postupným vylepšováním řešení
- Pracuje s dopravní sítí a **distanční maticí** pro přesný výpočet času přejezdu mezi stanovišti

### Dopravní síť a distanční matice
> Kapitola CK: Dopravní síť a distanční matice

**Dopravní síť** je modelována jako graf:
- **Uzly** — místa realizace, stanoviště, likvidační místa, depa
- **Hrany** — vzdálenosti mezi uzly

**Distanční matice** počítá vzdálenost nebo čas na přemístění mezi uzly.

**Správa:** Dopravní síť a distanční matice → na straně dodavatele. Správa míst → na straně zákazníka.

### Kontrola a manuální editace navržených SOD
> Kapitola CK: Kontrola a manuální editace navržených okruhů dne

Dispečer provádí:
- Slučování a dělení SOD
- Přesouvání SOS mezi SOD
- Řazení nezahrnutých SOS do SOD
- Konkretizace míst výjezdu a likvidačních míst

**Rozhodovací kritéria:** pracovní čas, zaplněnost vozidla (uživatel **není limitován** překročením zaplněnosti).

### Evidence nových okruhů a přenos do HEN
> Kapitola CK: Evidence nových okruhů a řazení RPO do nových okruhů; Přenos nastavení okruhů DO HEN

Kroky:
1. Založení nových okruhů v PP
2. Mapování SOD na nové okruhy
3. Zařazení položek objednávek do nových okruhů a odpovídajících rozvrhů

PP je **zdrojem pravdy** pro okruhy a vazby RPO na okruhy a rozvrhy. Proto:
- Po založení nových okruhů → **automatická synchronizace** okruhů s HEN
- Po doplnění nových vazeb RPO → **automatická synchronizace** vazeb s HEN

### Datové toky strategické optimalizace — souhrn

| Tok | Směr | Obsah |
|---|---|---|
| FOB → RP | FOB → RP | Váhy odpadu a objemové využití z okruhů dne |
| RP → BI | RP → BI (datový sklad) | Váhy a objemy per okruh dne, data objednávek |
| BI → uživatel | BI → analytik | Předzpracované reporty pro analýzu produkce odpadu |
| RP (optimalizace) | Interní RP | SOS → optimalizační algoritmus → SOD (verze optimalizace) |
| RP → PP | RP → PP | Nové okruhy, vazby položek objednávek, rozvrhy |
| PP → HEN | PP → HEN | Automatická synchronizace nových okruhů a vazeb |

---

## Dopady na architekturu řešení
> Kapitola CK: Dopady na architekturu řešení

V současnosti jsou RP a PP vzájemně nezávislé, bez přímé komunikace. Obě využívají stejné externí zdroje:
- **Fleetware** — autentizace/autorizace, poziční data, údaje o vozidlech a zaměstnancích
- **ERP** (HEN, postupně opouštěný WinyX) — smlouvy, navázané entity, číselníky

Po zavedení CS dochází ke značnému přiblížení PP a RP — aplikace budou u určitých UC pracovat se stejnými daty.

Vyžadované oblasti řešení:
1. Zavedení komunikačního rozhraní mezi RP a PP
2. Odpovídající rozšíření datových modelů obou aplikací
3. Příprava aplikací (zejména RP) na mnohem větší datové objemy a výkonové nároky

### Uplatňované architektonické principy
> Kapitola CK: Uplatňované architektonické principy

| Princip | Naplnění |
|---|---|
| **Jasně dané vlastnictví dat (SoT)** | PP je primární zdroj detailních dat ze smluv. RP vytváří trasy / denní plány a tato data vlastní. |
| **Preference jednosměrných synchronizací** | Jeden zdroj pravdy je zdrojem dat pro ostatní systémy. Výjimka pouze s dobrým důvodem. |
| **Centralizace logiky — neduplikovat** | Např. generování OS řeší jedna aplikace. Pokud s výstupy pracují obě, data si předají. |
| **Synchronizovat pouze nutná data** | RP pracuje pouze s podmnožinou dat relevantní pro plánování. |
| **Volná vazba — prostřednictvím API** | Nepřijatelné je přímé čtení z DB jiné aplikace. Možný je deep linking. |
| **Přístup MVP** | Nejprve základní funkcionalita, iterativní vylepšování na základě zpětné vazby. |

---

## Nefunkcionální požadavky
> Kapitola CK: Nefunkcionální požadavky

**Výkon a škálovatelnost:**
- Propustnost: cca **150 vozidel**, cca **600 objednaných služeb** na jeden denní výkon
- Obvyklé operace v jednotkách sekund při běžném zatížení
- Horizontální i vertikální škálovatelnost

**Spolehlivost:** SaaS režim dle SLA, definované RPO a RTO.

**Bezpečnost:** TLS/SSL, šifrování dat v klidu, ochrana proti OWASP top 10.

---

## Identifikovaná rizika
> Kapitola CK: Identifikovaná rizika

| Riziko | Mitigace |
|---|---|
| **Zpomalení systémů** — nárůst objemu dat zejména v RP | Optimalizace výkonu od počátku, PoC, archivace dat |
| **Překročení rozsahu (Overscoping)** | Přístup MVP |
| **Součinnost s třetími stranami** (d2B, dodavatel optimalizace) | Prověření partnera, jasné kompetence, pravidelná komunikace |
| **Strategická optimalizace** — komplexní problém | Předimplementační analýza a PoC, spolehlivý dodavatel |

---

## Přílohy

### Slovník pojmů a zkratek
> Kapitola CK: Slovník pojmů a zkratek

| Pojem | Význam |
|---|---|
| **Fleetware (FLW)** | Ekosystém aplikací — GPS monitoring, plánování tras, evidence nádob |
| **FLW Pasport (PP)** | Webová aplikace pro evidenci malých odpadových nádob |
| **FLW RoadPlan (RP)** | Webová aplikace pro plánování tras |
| **FleetwareWeb2 (FLWW2)** | Webová aplikace pro GPS monitoring |
| **FLW Picker** | Mobilní aplikace pro terénní práci s nádobami a kontejnery |
| **FleetwareOnBoard (FOB)** | Mobilní aplikace pro realizaci naplánovaných tras, vozidlový terminál |
| **FLW PublicWeb** | Webová aplikace pro prezentaci dat veřejnosti / koncovým zákazníkům |
| **Helios Nephrite (HEN)** | ERP systém Helios — implementace v MP SK |
| **Smlouva** | HEN: smluvní ujednání s zákazníkem. V PP = **objednávka** |
| **Předmět smlouvy** | HEN: specifikace nasmlouvané služby. V PP = **položka objednávky** / **RPO** |
| **Typ předmětu smlouvy** | HEN: rozlišení druhů řádků (např. KOM). V PP rozlišeno prefixem čísla RPO |
| **Okruh** | HEN: sdružuje řádky smluv obsluhované společně. V FLW nová entita |
| **Rozvrh** | HEN: reálné termíny obsluhy dle smluvní frekvence. V FLW nová entita |
| **Zóna** | HEN: oblast / soubor adres. V FLW nová entita |
| **Cyklický svoz (CS)** | Pravidelný svoz malých nádob. Typ předmětu „ZVOZ C" |
| **Trasa dne** | HEN: soubor řádků smluv pro datum/vozidlo/osádku. V RP = **denní výkon** |
| **Operativní plánování** | Postupy k naplánování denních tras / výkonů |
| **Strategické plánování** | Optimalizace skladby okruhů na základě parametrů |

### Aktéři a jejich participace v procesech
> Kapitola CK: Aktéři a jejich participace v procesech

| Aktér | Proces | Primární systém |
|---|---|---|
| Obchodní zástupce | Správa smluvních údajů | HEN |
| Dispečer | Příprava okruhů a rozvrhů | HEN -> RP / PP |
| Dispečer | Plánování denních výkonů CS | RP |
| Dispečer | Monitoring realizace CS | RP |
| Dispečer | Vyhodnocení realizace CS | RP |
| THP | Příprava realizačních dokladů | HEN |
| THP | Pasportizace nádob a stanovišť | PP |
| Řidič | Realizace naplánované trasy CS | FOB |
| Smluvní zákazník | Zobrazení informací pro veřejnost | FLW PublicWeb |
| Specialista strategické optimalizace | Strategická optimalizace CS | RP |
| Vedoucí pracovník | Kontrolní činnost / analýzy / reporting | PP / RP / Reporting |

### Výsledky generování OS dle vazeb RPO

| # | RPO v Rozvrhu | RPO v Okruhu | Okruhu přiřazeno vozidlo | Výsledek |
|---|---|---|---|---|
| 1 | ANO | ANO | ANO | OS zařazeny do okruhu dne -> okruh dne zařazen do DV |
| 2 | ANO | ANO | NE | OS zařazeny do okruhu dne -> připraven k zaplánování do DV |
| 3 | ANO | NE | NE | Založeny OS a lokace -> dispečer musí zařadit do okruhů dne |
| 4 | NE | NE | NE | OS a lokace nebudou vygenerovány |

Klíčový poznatek: Vazba RPO na **Rozvrh** je nutnou podmínkou pro vygenerování objednaných služeb. Vazba na **Okruh** určuje míru automatizace zařazení do denního výkonu.

### Vstupní parametry strategické optimalizace

**Data z PP:**

| Entita | Atribut | Stav |
|---|---|---|
| RPO | Skupina odpadu | budoucí |
| RPO | Frekvence vývozu | aktuální |
| RPO | Typ nádoby | aktuální |
| RPO | Počet nádob | aktuální |
| RPO | Platnost | aktuální |
| RPO | Adresa MR (alt.: stanoviště nádoby) | aktuální |
| RPO | Souřadnice MR (alt.: souřadnice stanoviště) | budoucí |
| Stanoviště nádoby | Souřadnice | aktuální |
| Typ nádoby | Čas obsluhy | budoucí |
| Typ nádoby | Váhové naplnění per skupina odpadu | budoucí |
| Typ nádoby | Objemové zaplnění per skupina odpadu | budoucí |

**Data z RP / FLWW2:**

| Entita | Atribut | Stav |
|---|---|---|
| Likvidační místo | Skupina odpadu | budoucí |
| Likvidační místo | Souřadnice LM | aktuální |
| Vozidlo | Použít pro SP | budoucí |
| Vozidlo | Objem nástavby | budoucí |
| Vozidlo | Nosnost | budoucí |
| Vozidlo | Pracovní doba (jedna směna) | budoucí |

**Konfigurace výpočtu optimalizace (RP):**

| Parametr | Stav |
|---|---|
| % využití objemu | budoucí |
| % využití nosnosti | budoucí |
| % využití pracovní doby | budoucí |

Pozn.: „aktuální" = atribut již existuje; „budoucí" = bude potřeba doplnit.

### Kritéria dělení okruhů

| Kritérium | Ukončení řazení OS do okruhu dne | Rozhodnutí |
|---|---|---|
| **Pracovní doba** | Čas přejezdů + čas obsluhy > pracovní doba vozidla | Dle toho, co nastane dříve |
| **Váha odpadu** | Celková váha > nosnost vozidla | Dle toho, co nastane dříve |
| **Objem odpadu** | Celkový objem > objem nástavby | Dle toho, co nastane dříve |

### Frekvence vývozu

| Frekvence | Počet svozů týdně | Příklad |
|---|---|---|
| 1x týdně | 1 | středa |
| 2x týdně | 2 | pondělí, čtvrtek |
| 3x týdně | 3 | pondělí, středa, pátek |
| 4x týdně | 4 | pondělí, úterý, čtvrtek, pátek |
| 5x týdně | 5 | pondělí až pátek |
| 1x za 14 dní | 0.5 | středa (každý druhý týden) |
| 1x měsíčně | ~0.25 | první středa v měsíci |

### Srovnání RP a PP — původní vs. nový stav

| | RoadPlan | Pasport |
|---|---|---|
| **Původní stav** | Nepravidelná doprava — velkoobjemové kontejnery. Objednávky vznikají v RP. | Evidence malých nádob na komunální a tříděný odpad pro CS. Z ERP importovány smlouvy a předměty smluv. |
| **Nový stav** | Pravidelná doprava — CS malých nádob. Řádově větší množství OS. Pro plánování potřebné detailní údaje z PP. | Poskytuje funkčnost práce s okruhy (vizualizace nad mapou, sestava okruhů). |

### Harmonogram

| Krok | Termín | Stav |
|---|---|---|
| Schůzky k funkčním požadavkům | 06/2025 | Dokončeno |
| Schválení funkčních požadavků | 9.7.2025 | Dokončeno |
| Příprava konceptu | 29.8.2025 | Dokončeno |
| Schválení konceptu | 11.9.2025 | — |
| Podání nabídky | 6.10.2025 | — |
| Návrh řešení I. fáze | 12/2025 | — |

### Otevřené body z CK
> Kapitola CK: Otevřené body

Celkem 16 otevřených bodů projednaných s MP SK. Klíčové závěry:

**Okruhy a rozvrhy (body #01-04):**
- Přidání/odebrání RPO z **okruhu** v HEN nevytváří novou verzi RPO.
- Funkce „Změnit rozvrh" vytváří novou verzi RPO od zadaného data (pokud je datum zadáno).
- **Okruh** je na uvážení uživatele — HEN nevaliduje logiku přiřazení.
- **Zóna** je nepovinná, pouze pomocný filtr.
- Editace RPO v HEN je možná i do minulosti.
- Trasu dne lze v HEN sestavit i bez použití okruhu a rozvrhu (čistě výběrem RPO).

**Skupiny odpadu (#05):**
- HEN nemá validaci skupin/druhů odpadu v rámci okruhu — záleží na uživateli.
- Validace skupin odpadu v trase dává smysl a bude řešena na straně FLW.

**Realizační doklady (#06):**
- Účelem RP je předat do HEN zkontrolovanou trasu dne. THP poté v HEN vytvářejí realizační doklady.

**Provozní parametry (#09-13):**
- V Etapě 1 se neřeší dvousměnný provoz vozidel CS.
- Vozidlo obsluhuje daný okruh/trasu zpravidla pravidelně (stabilní přiřazení).
- Průměrné počty nádob za směnu: město KO ~600, obce KO ~500, město separace = srovnatelné s KO.
- Plánování jen na vozidla, posádka se neřeší.

**Otevřené body (#07, #14):**
- Přístupy RAD do testovacího HEN — otevřeno.
- Nutnost paralelního plánování v HEN i RP — otevřeno (komplikuje řešení obousměrnou synchronizací).
