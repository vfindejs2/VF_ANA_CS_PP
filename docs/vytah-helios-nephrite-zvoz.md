# Výtah: Helios Nephrite — modul ZVOZ (User Guide)

**Zdroj:** `data/input/helios_user_guides/HeN_ZVOZ+CYKLICKÝ_kontajner_zóna_okruh_trasa.pdf`
**Datum dokumentu:** srpen 2024
**Počet stran:** 57
**Datum výtahu:** 2026-02-23
**Účel:** Zachytit strukturu dat, business logiku a workflow v HEN modulu ZVOZ — jako podklad pro integrační specifikace (HEN→PP, RP→HEN) a entitní analýzu.

---

## 1. Shrnutí relevance pro projekt

Dokument je **uživatelský manuál modulu ZVOZ** systému Helios Nephrite (HEN). Pokrývá kompletní životní cyklus cyklického svozu od založení predmětů zmluvy přes plánování (zóny, rozvrhy, okruhy) až po realizaci (trasa dňa, realizační doklady).

### Proč je cenný
- **HEN je Source of Truth** pro okruhy, rozvrhy, zóny a kalendáře v Etapě 1 (viz KR z CK)
- Dokument detailně popisuje **datový model a vazby entit v HEN**, které se přenášejí do PP (FR-01) a zpět z RP (FR-03)
- Obsahuje **business pravidla** (verzování PZ, identifikace nádob, stavový automat výsypov), která ovlivňují návrh integračních kontraktů
- Ukazuje **reálné screenshoty UI a formulářů**, z nichž lze odvodit atributy a jejich povinnost

### Mapování na integrační toky projektu
| Integrační tok | Relevantní entity z HEN |
|----------------|------------------------|
| **FR-01: HEN → PP** | Rozvrh, Zóna, Okruh trasy, vazby PZ↔Rozvrh/Okruh/Zóna |
| **FR-03: RP → HEN** | Trasa dňa (realizačný doklad), Okruh trasy, Predmet zmluvy |
| Kontext pro FR-02 (PP→RP) | Predmet zmluvy = zdrojová entita pro RPO v PP |

---

## 2. Entity a datový model v HEN

### 2.1 Predmet zmluvy (PZ) — centrální entita

**Predmet zmluvy** je základní datová jednotka modulu ZVOZ. Eviduje vývoz nádob / obslúženie nádob na stanovisku zákazníka.

#### Typy predmetov zmluvy

| Typ predmetu | Prípad | Cyklika | Pôvod RD | Odpadová evidencia | Fakturácia |
|---|---|---|---|---|---|
| **ZVOZ C** | Čipované nádoby s ID na konkrétnych stanoviskách | cyklika | trasa dňa | bez OE | áno/nie |
| **ZVOZ N** | Virtuálne nádoby bez ID na konkrétnych stanoviskách | cyklika | trasa dňa | bez OE | áno/nie |
| **ZVOZ** (necyklický) | Nádoby bez ID, spoločné stanovisko | nie | trasa dňa / ručne | bez OE | áno/nie |
| **Nakladanie s odpadom** | Eviduje sa vždy | — | váženie/ručne | áno | áno/nie |
| **Ostatné činnosti** | Fakturačné položky | nie | ručne | nie | áno |
| **Časové plnenie** | C-fa položky | cyklika | predpis RD | bez OE | áno |

> **Klíčové pro projekt CS:** Typ ZVOZ C a ZVOZ N jsou cyklické svozy. Rozlišení C/N je na úrovni přiřazení Rozvrhu/kalendáře — samotný typ predmetu nemá atribut „cyklický", to se určí navázáním rozvrhu.

#### Záložka „Zmluva a financie"
- **Typ predmetu** — výběr z číselníku „Typy predmetov zmlúv" (referenční č. 001=ZVOZ C, 001=ZVOZ N, 002=Nakladanie, ...)
- **UCP** (Účtovná cena prodeje) — identifikátor
- **Ostatné verzie** — počet verzí predmetu
- **Aktívne** — příznak
- **Zmluva** — vazba na smlouvu
- **Zákazka** — vazba na fakturační zakázku
- **Riadok, Verzia** — identifikace řádku a verze v rámci smlouvy
- **Zahájenie / Ukončenie** — platnost predmetu
- **Zmluvné strany:** Stredisko, Odberateľ (+ IČO), Držiteľ
- **Detail ceny:**
  - Typ podkladu ceny (realita/zmluva)
  - Spôsob určenia ceny: MJ
  - MJ: vývoz / ks
  - Počet MJ, Jednotková cena, Cena
- **Ekonomika:** TPF, Aktivita, Text na faktúru (částečně z šablóny)

#### Záložka „Odpad a nádoba"
- **Odpad** — statický vztah na katalóg odpadov (kód odpadu, napr. 200301)
- **Cenová špecifikácia** — vazba na cenovou specifikaci
- **Likvidačné miesto** — kam sa odpad odváža
- **Likvidačné miesto – Vývoz** — odkiaľ sa vyváža
- **Poplatník, Expozitúra, Pôvodca odpadu**
- **Poznámka na VL** — pri RD pôvod Váženie se automaticky doplní do vážneho lístku
- **Stanovisko a nádoba:**
  - **Adresa stanoviska** — statický vztah na adresný bod (**povinné vždy!**)
  - **Čas od / Čas do** — časové okno obsluhy
  - **Zóna** — statický vztah na zónu (vo Waste konfigurácii sa nastavuje kontrola)
  - **Typ zástavby** — statický vztah
  - **Opis stanoviska** — poznámka pre dispečera (plní sa do šablóny, podklad pre vodičov)
  - **Typ nádoby** — statický vztah na typy nádob (výběr z číselníka)
  - **Počet nádob** — sumárny počet
  - **ID Nádob** — riadiaci atribút (viz sekce 2.2)
  - **Interval** — statický vztah na číselník (napr. 1x14)
  - **Poznámka pre dispečéra**

#### Záložka UDA
- **Dni zvozu (pre RADIUM)** — statický vztah na číselník dní zvozu (napr. UT-V = Utorok)
  > **Důležité pro integraci:** Toto pole slouží k přenosu informace o dnu zvozu do systému RADIUM (Fleetware/PP/RP). V případě cykliky se systém řídí Rozvrhom/dnem vývozu.
- **Pôvodní PP** — odkaz na původní PasPort

#### Dynamické vzťahy (Vzťahy) — pravý panel formuláře
| Typ predmetu | Vztah na | Obsah |
|---|---|---|
| Nakladanie odpadu → Ostatné činnosti | Činnosť [Povinná] | automaticky sa vygeneruje v RD |
| Nakladanie odpadu → Ostatné činnosti | Činnosť [Voliteľná] | ponúkne sa k výberu |
| Zvoz → Nakladanie odpadu | **Odpad kombinovaného zberu** | kód odpadu |
| Zvoz → Nakladanie odpadu | **Rozvrh** | kalendár zvozu |
| Zvoz → Nakladanie odpadu | **Užívatelia názvy** | meno majiteľa nádoby |
| Zvoz → Nakladanie odpadu | **Užívatelia adresy** | adresa majiteľa (ak iná ako stanovisko) |

> **Klíčové pro integraci HEN→PP (FR-01):** Rozvrh je navázán na PZ jako **dynamický vztah** (ne jako atribut na záložce). To znamená, že vazba PZ↔Rozvrh se přenáší přes vazební tabulku, ne přes FK na PZ.

#### Kombinácia ZVOZ + Nakladanie s odpadom
- **Vlastná doprava, nádoby bez ID:** 1:1 — 1 ZVOZ + 1 NAKLADANIE za kód odpadu / likvidačné miesto (LM)
- **Vlastná doprava, stanoviská s ID:** x:1 — x predmetov ZVOZ za 1 kód odpadu + nádoba + adresný bod + rozvrh + interval na LM, + 1 NAKLADANIE
- **Vlastná doprava + kombinovaný zber:** 1:x — 1 ZVOZ za 1 nádobu, x NAKLADANIE za x odpadov v jednej nádobe + LM + % prepočtu + komb. zber = áno

#### Verzování PZ
- **Nová verzia predmetu zmluvy** — funkcia pre predmety **bez pohybov nádob (bez ID)**
- Vytvorí novú verziu (Riadok zachovaný, Poradie = Verzia + 1)
- Použitie: zmena ceny, intervalu, likvidačného miesta... s zachovaním histórie
- **Operácia je NEVRATNÁ**
- Funkcie vytvářející novou verzi: Pristavenie nádoby, Odvoz nádoby, Zmena veľkosti nádoby, Zmena rozvrhu, Zmena užívateľov, Hromadná zmena údajov

### 2.2 Nádoba (Zberná nádoba)

#### Identifikace nádob (ID Nádob) — 3 stavy

| Stav | Význam | Počet nádob | Pohyb |
|---|---|---|---|
| **Zatiaľ neurčené** | Nový predmet, ešte nezadaný počet | nemá | nemá |
| **Nádoby bez ID** | Fiktívne nádoby — sumárny počet na stanovisku | vyplnený (sumár) | neviaže sa na konkrétnu nádobu |
| **Nádoby s ID** | Nádoby naviazané cez pohyb-funkciu | podľa pohybov | viazané na konkrétnu nádobu |

#### Atributy záznamu Zberná nádoba
- **Typ nádoby** — z číselníku (napr. 120 l komunál, 1100 l plechový komunál)
- **Evidenčné číslo** — generované systémom (FB6385...)
- **RFID** — číslo čipu (napr. MPSK1111)
- **Výrobné číslo**
- **Skupina nádob**
- **Stav nádoby** (+ Známka)
- **Farba**
- **Objem** — v litroch (napr. 120)
- **MJ** — liter
- **Vlastníctvo nádoby** — kdo vlastní
- **Hmotnosť (kg)**
- **Materiál**
- **Virtuálna nádoba** — Áno/Nie (virtuálna = bez čipu, len vygenerované evidenčné číslo)
- **V používaní** — Áno/Nie

> **Pravidlo:** Po doplnění RFID + Evidenčné číslo do frm. Zbernej nádoby sa stav Virtuálnej nádoby mení z Áno na Nie. Nádoba priradená k predmetu má V používaní = Áno.

#### Funkce pro práci s nádobami
| Funkcia | Popis | Vytvára novú verziu PZ? |
|---|---|---|
| **Pristavenie nádoby** (bez ID) | Pridá počet nádob, vytvorí novú verziu PZ | Áno (ak zmena dátumu) |
| **Odvoz nádoby** (bez ID) | Odoberie počet nádob | Analogicky |
| **Pristavenie nádoby s ID** | Priradí konkrétnu nádobu (virtuálnu alebo čipovanú) | Nie (pohyb na záložke Nádoby) |
| **Odvoz nádoby s ID** | Odoberie konkrétnu nádobu | Nie |
| **Zmena veľkosti nádoby** | Zamení typ nádoby — ukončí pôvodný predmet, pristaví nový | Áno |
| **Migrácia na adresu** | Presunie nádobu na iné stanovisko (iný predmet zmluvy) | Áno (nová verzia riadku) |
| **Zmena rozvrhu** | Zmení kalendár na predmete | Voliteľne (s/bez novej verzie) |
| **Zmena užívateľov** | Zmení užívateľa nádoby | Áno |

### 2.3 Zóna

> **ZÓNA je ZÁKLADNÝM STAVEBNÝM PRVKOM pre plánovanie cyklického zvozu nádob — podľa adresných bodov.**

- Je pomocným nástrojom pre vytvorenie **OKRUHU TRASY** a následne **TRASY DŇA**
- = množina združených **ulíc, mestských častí, obcí, okresov**

#### Atributy
- **Reference subjekt** — automatické číslo
- **Názov subjektu** — ľubovoľný názov (napr. "KO - Beluša - Pondelok")
- **Stav** — aktívny
- **Poznámka**
- **Vzťahy / Útvar** — dynamický vztah na stredisko (napr. 4010 MGW Považská Bystrica)
- **Región** — statický vzťah

#### Položky zóny (adresná štruktúra)
Definujú sa na rôznej úrovni granularity:
- **Okres** (najhrubší)
- **Obec**
- **Časť obce**
- **Mestská časť**
- **Ulica** (+ voliteľne číslo domu)

> Nemusíte ísť až na úroveň ulice/číslo domu. Stačí vybrať napr. Časť obce a systém dohľadá adresné body na predmetoch zmlúv.

#### Funkcie
- **Adresy zóny** — náhľad adresných bodov patriacich k vybraným položkám
- **Pridať zónu na PZ** — zobrazí predmety zmluvy s adresami patriacimi do danej zóny, po vyznačení priradí Zónu k PZ
  - Ak nie sú presne definované ulice na položkách → zobrazí všetky ulice k danej časti obce
  - Ak definuje konkrétne ulice + číslo → ponúkne len predmety so stanoviskom na tej ulici + čísle

> **Dôležité pre HEN→PP integráciu:** Zóna sa na PZ ZVOZ dopĺňa automaticky na základe **adresy stanoviska** predmetu zmluvy. Podmienky: adresa je v aktívnej Zóne, zhoda v Útvare, a ak je nastavená Kontrola zóny vo Waste konfigurácii, musí zodpovedať obojstranne.

### 2.4 Rozvrh vývozu (Rozvrh vývozov)

> **ROZVRH je ZÁKLADNÝM STAVEBNÝM PRVKOM pre plánovanie cyklického zvozu nádob — podľa kalendára.**

- Je pomocným nástrojom pre vytvorenie **OKRUHU TRASY** a následne **TRASY DŇA**

#### Atributy hlavičky
- **Referencia** — automaticky podľa prednastavenej masky
- **Názov** — automaticky po uložení podľa zadania (rok platnosti / kód frekvencie / nastavenie frekvencie / počet vývozov ročne)
  - Príklad: `24|TÝŽ.P,N/PO.PIA./105`
- **Zmluva** — voliteľný statický vzťah (ak sa rozvrh používa pre viac zmlúv, nevypĺňa sa)
- **Frekvencia:**
  - **Denne** — Prvý deň vývozu (dátum) + Opakovať každý (číslo dní)
  - **Týždenne** — Párny/Nepárny týždeň + konkrétne dni (Po, Ut, St, Št, Pi, So, Ne)
  - **Vlastné** — ručné zadanie
- **Rok platnosti** — číslo (napr. 2024)
- **Počet vývozov vo výskyte** — defaultne 1 (viac = viacnásobná obsluha za deň)
- **Počet opakovaní za rok** — vypočíta systém (pri Vlastné sa zadáva ručne)
- **Distribúcia po mesiacoch:** Jan – Dec (pri frekvenčnom nastavení automaticky, pri Vlastné ručne)
- **Poznámka** — textové pole
- **Rozdiel v dňoch vývozu** — východzí stav 0 (rozdiel medzi systémom vygenerovanými a užívateľom zmenenými dňami)

#### Položky rozvrhu = Dni vývozu
- Generujú sa funkciou **Vytvoriť dni rozvozu**
- Každý deň má: Vývoz (Áno/Nie), Deň v týždni, Deň, Týždeň v mesiaci, Štvrtok, Týždeň, Typ týždňa, Víkend, Sviatok, Posledný v mesiaci, Poznámka
- **Dni vývozu nie je možné užívateľom editovať** — stav sa mení funkciou **Prepnúť vývoz**
- **Prepnúť vývoz** invertuje stav vybraných položiek (Áno↔Nie)

#### Navázání na PZ
- Cez **dynamické Vzťahy** na formulári PZ → Rozvrh (kalendár)
- Alebo hromadne cez funkciu **Doplň rozvrh** (Filter zvozových predmetov → výběr PZ → priradenie rozvrhu)
- Rozvrh je možné staticky naviazať ku konkrétnej Zmluve

> **Pozor!** Ak zmeníte Rozvrh, treba upraviť aj Interval a Deň zvozu v záložke UDA — nie sú previazané.

### 2.5 Okruh trasy

> **OKRUH TRASY je STAVEBNÝM PRVKOM pre plánovanie TRASY DŇA pomocou ZÓNY a ROZVRHU VÝVOZU.**

#### Atributy
- **Reference subjekt** — automatické číslo
- **Názov subjektu** — ľubovoľný (napr. "KO - Nimnica, Visolaje", "KO - Domaniža")
- **Stav** — aktívny (Áno/Nie)
- **Poznámka**
- **Dni zvozu** — odvodené z predmetov (napr. Pondelok, Streda, Piatok)
- **Typ týždňa** — Párny / Nepárny / Oba (odvodené od Rozvrhu na predmetoch)
- **Celkový počet nádob** — sumár za okruh
- **Celkový objem nádob** — sumár za okruh

> V dynamickom vzťahu sa automaticky naviaže **Útvar** (stredisko).

#### Vytvorenie okruhu — formulár „Pridať predmety"
Systém vyhľadáva predmety zmluvy ZVOZ podľa troch kritérií:
1. **Rozvrh vývozov** — podľa navázaného rozvrhu na PZ
2. **Zóny** — podľa skupiny stanovísk/ulíc priradenej zóny
3. **Okruhy** — podľa existujúcich okruhov

Filtrovanie:
- **Deň zvozu** — zaškrtnutím dňa a typu týždňa systém identifikuje a filtruje predmety pre konkrétny deň
- **Odpad zvozu** — kód odpadu
- **Predmety v zónach** — zónové rozdelenie
- **Predmety v okruhoch** — existujúce okruhy

Tlačidlo **Filtrovať** zobrazí výber PZ, ktoré spĺňajú parametre. Informácia: počet predmetov, počet nádob, sumárny objem.

Tlačidlo **Potvrdiť výber** — vyznačíme konkrétne predmety a potvrdíme zaradenie.

> **Jeden predmet môže byť vo viacerých Okruhoch** (vizuálne zduplikované riadky v šablóne, ale stále to isté stanovisko a nádoba).

#### Funkce nad okruhy
- **Pridať do okruhu** — hromadné priradenie vybraných PZ do okruhu
- **Odstrániť z okruhu** — hromadné odobranie
- **Doplniť zónu automaticky** — doplní zónu na PZ ZVOZ na základe adresy stanoviska
- **Aktualizácia predmetov** — prepočíta počet nádob podľa aktuálneho dátumu a pohybov, vymaže ukončené predmety z okruhu

### 2.6 Trasa dňa (realizačný doklad typu ZVOZ)

> **REALIZAČNÝ DOKLAD (jazda)** je základným evidenčným podkladom, z ktorého sa následne ťahajú dáta k fakturácii a k odpadovej legislatíve (Envita).

#### 4 typy realizačných dokladov (podľa pôvodu)
1. **TRASA DŇA** — typ predmetu ZVOZ (cyklický zvoz)
2. **VÁŽENIE** — typ predmetu NAKLADANIE S ODPADOM
3. **RUČNE** — typ predmetu NAKLADANIE S ODPADOM, ZVOZ, OSTATNÉ
4. **PREDPIS RD** — typ predmetu ČASOVÉ PLNENIE

#### Atributy Trasy dňa (hlavička)
- **Informácie o doklade:**
  - **Stav** — Nespracované / RD Vytvorený
  - **Referencia** — automatické číslo
  - **Organizácia** — napr. TEST MEGAWASTE SLOVAKIA
  - **Útvar** — stredisko (napr. 4010 MGW Považská Bystrica)
- **Vodič a vozidlo:**
  - **Dátum** — dátum trasy
  - **EČV** — evidenčné číslo vozidla
  - **Jazda** — navázanie na **Fleetware jazdu** (ID jazdy)
  - **Vodič** — meno vodiča (doplní sa z jazdy)
  - **Vozidlo** — typ vozidla

#### Krok 1: Plánovanie (výber predmetov do trasy)
Dva spôsoby:
- **Pridať predmety** — ručný výber zo všetkých platných PZ ZVOZ pre daný útvar
  - Systém sleduje duplicity (nepovolí pridať PZ, ktorý je už v trase pre rovnaký útvar v rovnaký deň)
  - Možnosť filtrovať podľa Okruhu alebo Riadkového filtra
  - Informácia: „Dostupné predmety: 892 PZ zvoz pre útvar MGW"
- **Pridať podľa rozvrhu** — systémový výber podľa Okruh trasy + Rozvrh vývozu
  - Áno → systém ponúkne Okruhy trasy podľa Rozvrhu vývozu
  - Nie → systém ponúkne Predmety typu Zvoz podľa Rozvrhu vývozu
  - Informácia: „Dnes ešte nenaplánované predmety: 10"

> **Klíčové pre integraci RP→HEN (FR-03):** Trasa dňa v HEN = ekvivalent Denního výkonu (DV) v RP. Export z RP do HEN by měl naplnit právě tuto entitu — hlavičku (dátum, vozidlo, vodič) + seznam PZ/stanovísk.

#### Krok 2: Spracovanie výsypov (porovnání plánu s realitou)

Položky trasy mají stavy:
- **Nespracované** — identifikáciou obslúženia sa mení na Obslúžené/Neobslúžené
- **Výsyp** — Nie/Áno
- **Pôvod** — Mimo rozvrh / Podľa rozvrhu / Presunuté na iný deň

Systém automaticky spočíta všetky vložené položky podľa stavu:
- **Nespracované:** X
- **Neobslúžené:** Y (identifikované — neuskutočnený výsyp)
- **Obslúžené:** Z (identifikované — uskutočnený výsyp, prenášajú sa do RD k fakturácii)
- **Nespárované výsypy:** W (z FLW evidovaný výsyp ale nenašiel plán v HEN)

#### Funkční tlačidla spracovania
| Funkcia | Popis |
|---|---|
| **Obslúžené všetky** | Hromadne zmení stav z Nespracované na Obslúžené (najrýchlejší) |
| **Spárovať s výsypmi** | Automaticky spáruje výsypy z Fleetware. Nespárované = ručne |
| **Spracovať položky trasy** | Detailní formulár — zákazky → stanoviská → stavy. Tlačidlo Spracovať mení stav vybraných položiek |
| **Obslúžiť bez ID** | Zvýšiť/znížiť počet plánovaných výsypov (len pre nádoby bez ID) |
| **Zmena LM** | Zmení likvidačné miesto na vybraných položkách |
| **Presun na iný deň** | Presunie nespracovanú položku na inú (existujúcu alebo novú) Trasu dňa |

#### Vytvorenie realizačného dokladu (RD)
- Funkcia **Vytvoriť realizačný doklad** z Trasy dňa
- Do RD sa preklopia **len položky v stave Obslúžené**
- **Všetky položky musia byť SPRACOVANÉ** (žiadna nesmie byť Nespracovaná)
- RD sa naviaže na Fleetware jazdu
- RD lze **Zrušiť** — ale len v stave fakturácie Nespracovaný

---

## 3. Business pravidla relevantní pro integraci

### 3.1 Vazby mezi entitami (hierarchie)

```
Zmluva
  └── Fakturačné podklady (Zákazka)
        └── Predmet zmluvy (PZ)
              ├── Stanovisko + Nádoba (záložka Odpad a nádoba)
              ├── Vzťah → Rozvrh (dynamický)
              ├── Vzťah → Zóna (statický, na stanovisku)
              └── Záložka Okruh trasy (odvozená)

Zóna
  └── Položky (adresná štruktúra: okres/obec/časť/ulica)

Rozvrh vývozu
  └── Položky = Dni vývozu (generované, stav Áno/Nie)

Okruh trasy
  ├── Predmety zmluvy (M:N — jeden PZ vo viacerých okruhoch)
  └── Odvozené: Dni zvozu, Typ týždňa, Počet nádob, Objem

Trasa dňa
  ├── Hlavička (dátum, vozidlo, vodič, jazda FLW)
  ├── Položky = PZ s výsypovým stavom
  └── → Realizačný doklad (len Obslúžené)
```

### 3.2 Pravidla verzování PZ
- Každá materiální změna (rozvrh, nádoba, cena, LM, užívateľ) vytvára **novú verziu** predmetu
- Nová verzia = rovnaký Riadok, nové Poradie (= Verzia)
- Starý predmet dostane dátum Ukončenia
- **Operace je NEVRATNÁ**
- Výnimka: Pristavenie/Odvoz nádob s ID = pohyb na záložke, nevytvára novú verziu

### 3.3 Pravidla identifikace nádob
- **ZVOZ C** = čipované nádoby s presnou definíciou stanovísk → **Nádoby s ID**
  - Každá nádoba má RFID + Evidenčné číslo
  - Počet nádob sa plní systémovo podľa pohybov
- **ZVOZ N** = virtuálne nádoby, spoločné stanovisko → **Nádoby bez ID**
  - Počet = sumárny na stanovisku (fiktívne nádoby)
  - Rozvrh nie je povinný (na rozdiel od ZVOZ C)

### 3.4 Pravidlo přiřazení Zóny
- Zóna sa dopĺňa automaticky na PZ ZVOZ na základe **adresy stanoviska**
- Podmienky: adresa v aktívnej Zóne + zhoda Útvaru
- Vo Waste konfigurácii sa nastavuje, či má kontrolovať aktívne a vyhovujúce zóny

### 3.5 Spojení s Fleetware
- Trasa dňa se naváže na **Fleetware jazdu** (EČV + dátum → ID jazdy)
- **Spárovanie s výsypmi** = automatické párovanie výsypov z FLW s plánovanými položkami v HEN
- Nespárované výsypy (FLW eviduje, HEN nevidí plán) = ručné spárovanie

> **Kontext pro integraci:** V budoucím modelu bude RP generovat DV (= Trasu dňa) a FOB (= mobilní klient) nahradí výsypy z FLW. Stavový automat výsypov v HEN je důležitý referenční bod.

---

## 4. Mapování HEN entit na projektové entity

| HEN entita | PP entita (cílová, FR-01) | RP entita (zdrojová, FR-03) | Poznámka |
|---|---|---|---|
| Predmet zmluvy (PZ) | RPO (položka objednávky) | RPO (objednaná služba) | Centrální entita, přenáší se z HEN→PP→RP |
| Rozvrh vývozu | Rozvrh / Kalendář (nová entita) | Rozvrh (nová entita) | Přenos z HEN do PP, pak z PP do RP |
| Zóna | Zóna (nová entita) | — (nepoužívá se v RP přímo) | Přenos z HEN do PP |
| Okruh trasy | Okruh (nová entita) | Okruh (nová entita) | Přenos z HEN do PP, pak z PP do RP |
| Trasa dňa | — | Denní výkon (DV) | Opačný směr: RP→HEN |
| Realizačný doklad | — | — | Generuje se v HEN z Trasy dňa |
| Zberná nádoba | Nádoba (container) | — | Přenos z HEN→PP |
| Stanovisko (adresa) | Stanoviště (address) | — | Součást PZ |

---

## 5. Nové poznatky oproti CK a dosavadní analýze

### 5.1 Detaily, které CK nespecifikuje
1. **Verzování PZ** — CK zmiňuje verze, ale HEN guide ukazuje přesnou mechaniku: které operace vytvářejí novou verzi, jaká je struktura (Riadok/Poradie), nevratnost
2. **ID Nádob — 3 stavy** — CK rozlišuje jen čipované/virtuální, HEN přidává stav "Zatiaľ neurčené"
3. **Dynamické vs. statické vzťahy** — Rozvrh je dynamický vztah, Zóna je statický (na stanovisku). To má vliv na integrační kontrakt
4. **UDA záložka s „Dni zvozu (pre RADIUM)"** — explicitní atribut pro přenos do našeho systému
5. **Kombinace ZVOZ + Nakladanie s odpadom** — pravidla poměrů 1:1, x:1, 1:x podle typu dopravy a identifikace nádob
6. **Rozvrh — detailní parametrizace** — frekvence (denně/týdně/vlastní), párny/nepárny týždeň, distribúcia po mesiacoch, počet vývozov vo výskyte
7. **Okruh trasy — odvozené atributy** — celkový počet nádob, celkový objem, dni zvozu, typ týždňa (automaticky z PZ)
8. **Trasa dňa — stavový automat položiek** — Nespracované → Obslúžené/Neobslúžené + pôvod (Podľa rozvrhu / Mimo rozvrh / Presunuté)
9. **Spárovanie s výsypmi FLW** — existující mechanismus v HEN pro porovnání plánu a reality

### 5.2 Upřesnění pro integrační toky
- **FR-01 (HEN→PP):** Potřebujeme přenést: Rozvrh (kompletní s položkami = dni vývozu), Zónu (s adresní strukturou), Okruh (s vazbami na PZ), a vazby PZ↔Rozvrh (dynamický vztah), PZ↔Zóna (staticky na stanovisku)
- **FR-03 (RP→HEN):** Denní výkon z RP se mapuje na Trasu dňa v HEN. Potřebujeme naplnit: hlavičku (dátum, EČV, vodič) + seznam PZ/stanovísk + navrhnout jak řešit spárovanie výsypov (FOB vs. FLW)

### 5.3 Otevřené otázky z Helios guide
- **OQ-HEN-01:** Jak přesně vypadá API pro dynamické vztahy (Rozvrh na PZ)? Je to REST endpoint nebo se přenáší jako součást PZ payloadu?
- **OQ-HEN-02:** Atribut „Dni zvozu (pre RADIUM)" na UDA záložce — bude se i nadále používat, nebo ho nahradí přenos celého Rozvrhu?
- **OQ-HEN-03:** Kombinovaný zber (1:x poměr ZVOZ:Nakladanie) — jak to ovlivní integrační kontrakt? Přenáší se i Nakladanie predmety, nebo jen ZVOZ?
- **OQ-HEN-04:** Spárovanie výsypov — v novém modelu (FOB místo FLW) bude párování v RP nebo HEN? Ovlivňuje to tok RP→HEN.

---

## 6. Doporučení pro další práci

1. **Aktualizovat input-analysis.md** — přidat Helios guide jako dokument #9
2. **Využít při entitní analýze PP** — entity Rozvrh, Zóna, Okruh mají v HEN guide detailnější specifikaci než v CK
3. **Využít při specifikaci FR-01 a FR-03** — atributové detaily z HEN formulářů poskytují přesný obraz zdrojových/cílových dat
4. **Eskalovat OQ-HEN-01 až OQ-HEN-04** — na příštím meetingu s D2B/MP
