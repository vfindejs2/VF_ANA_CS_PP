h1. Datový model PP — Entitní analýza pro CS (stav dle ER v1)

bq. *Účel:* Konsolidovaný popis tabulek a vazeb pro Cyklické svozy v PP podle aktuálního ER diagramu.
bq. *Zdroj diagramu:* {{docs/datovy-model/er-diagram-pasport_v1.html}}
bq. *Šablona:* {{docs/datovy-model/sablona-entitni-analyza.md}}
bq. *Aktualizováno:* 2026-02-24

----

h2. Obsah

# [RPO|#1-rpo]
# [Okruh|#2-okruh]
# [Rozvrh|#3-rozvrh]
# [Kalendar|#4-kalendar]
# [Zóna|#5-zóna]
# [Adresy|#6-adresy]
# [Skupina odpadu|#7-skupina-odpadu]
# [Druh odpadu|#8-druh-odpadu]
# [Nádoba|#9-nádoba]
# [Stanoviště|#10-stanoviště]
# [Typ nádoby|#11-typ-nádoby]
# [RPO_Okruh_Rozvrh|#12-rpo_okruh_rozvrh]

----

h2. Společné poznámky k modelu

* Dokument popisuje *tabulky z ER diagramu*, tj. konceptuální / analytický návrh pro PP a související integrace.
* Barvy v ER:
** modrá = nová entita pro CS
** zelená = existující entita rozšířená pro CS
* Přerušovaná vazba v ER znamená *volitelnou / fallback* vazbu (aktuálně {{Nádoba → Skupina odpadu}} při absenci vazby na RPO).
* Klíčová změna proti dřívějším variantám návrhu:
** vazby {{RPO_Okruh}} a {{RPO_Rozvrh}} jsou sjednoceny do temporální tabulky {{RPO_Okruh_Rozvrh}}
** {{RPO → Zóna}} má kardinalitu {{N:1}}
** {{Stanoviště}} nemá přímou vazbu na {{RPO}}
** {{RPO}} odkazuje na {{Adresy}} referencí ({{misto_realizace_adresa_id}})

----

h2. 1. RPO

*Stav:* Existující — rozšíření pro CS  
*Role:* Centrální entita CS (revize položky objednávky), základ pro plánování, filtraci a vazby na provozní objekty.

h3. Klíčové atributy

* {{id}} (PK)
* {{kod_polozky}} (z HEN)
* {{misto_realizace_adresa_id}} (FK → {{Adresy}})  
  Pozn.: nahrazuje původní textové {{misto_realizace}}.
* {{rpo_okruh_rozvrh_id}} (FK → {{RPO_Okruh_Rozvrh}})  
  Pozn.: reference na aktivní vazbu okruh/rozvrh.
* {{druh_odpadu_id}} (FK → {{Druh odpadu}})
* {{skupina_odpadu_id}} (FK → {{Skupina odpadu}})
* {{zona_id}} (FK → {{Zóna}})
* {{provozovna_id}} (FK)
* {{platnost_od}}, {{platnost_do}}
* {{stav}}

h3. Vazby

* {{RPO (N) → Zóna (1)}}
* {{RPO (N) → Adresy (1)}}
* {{RPO (N) → Skupina odpadu (1)}}
* {{RPO (1) → Nádoba (N)}}
* {{RPO (1) → RPO_Okruh_Rozvrh (N)}} (historie přiřazení)

h3. Pravidla a poznámky

* {{RPO}} je nositel obchodního kontextu pro obsluhu místa realizace.
* Aktivní kombinace {{Okruh + Rozvrh}} je reprezentována odkazem na {{RPO_Okruh_Rozvrh}}.
* Vazba na {{Stanoviště}} je *nepřímá přes Nádobu* (přímá vazba byla z návrhu odstraněna).

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* {{RPO}} je jednoznačný podklad o způsobu obsluhy místa realizace nebo navázaných nádob.
* V Etapě 1 jsou data RPO a vazeb na okruh/rozvrh/zónu synchronizována z HEN (PP je eviduje a zobrazuje).
* {{RPO}} je klíčový vstup pro generování objednaných služeb v RP (obsahuje mj. adresu/souřadnice MR, druh/skupinu odpadu, okruh, rozvrh, kalendářní dny).
* Vazba {{RPO → Rozvrh}} je nutnou podmínkou pro vygenerování objednané služby; vazba {{RPO → Okruh}} určuje míru automatizace zařazení do okruhu dne.
* V dalších etapách roste význam PP jako místa správy přiřazení RPO do okruhů/zón/rozvrhů (s dopadem na plánování).

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{rpo}}

*Sloupce (návrh):*
||Položka||Popis||
|id bigint identity primary key|—|
|kod_polozky nvarchar(100) not null|—|
|misto_realizace_adresa_id bigint null|—|
|rpo_okruh_rozvrh_id bigint null|—|
|druh_odpadu_id bigint null|—|
|skupina_odpadu_id bigint null|—|
|zona_id bigint null|—|
|provozovna_id bigint not null|—|
|platnost_od datetime2(0) null|—|
|platnost_do datetime2(0) null|—|
|stav nvarchar(50) null|—|
|audit/tenant sloupce dle standardu PP|—|

*FK (návrh):*
||FK sloupec||Reference / poznámka||
|misto_realizace_adresa_id|→ {{adresy(id)}}|
|rpo_okruh_rozvrh_id|→ {{rpo_okruh_rozvrh(id)}}|
|druh_odpadu_id|→ {{garbage_type(id)}} / ekvivalent DS|
|skupina_odpadu_id|→ {{garbage_group(id)}} / ekvivalent DS|
|zona_id|→ {{zone(id)}} / ekvivalent DS|
|provozovna_id|→ {{organization_unit(id)}}|

*Indexy (návrh):*
||Index||Definice / poznámka||
|IX_rpo_kod_polozky|({{kod_polozky}})|
|IX_rpo_provozovna|({{provozovna_id}})|
|IX_rpo_zona|({{zona_id}})|
|IX_rpo_skupina_odpadu|({{skupina_odpadu_id}})|
|IX_rpo_druh_odpadu|({{druh_odpadu_id}})|
|IX_rpo_adresa|({{misto_realizace_adresa_id}})|
|IX_rpo_aktivni_vazba|({{rpo_okruh_rozvrh_id}})|

----

h2. 2. Okruh

*Stav:* Nová entita pro CS  
*Role:* Seskupení RPO obsluhovaných společně; stavební kámen plánování v RP.

h3. Klíčové atributy

* {{id}} (PK)
* {{nazev}}
* {{provozovna_id}} (FK)
* {{vozidlo_id}} (FK)
* {{aktivni}}

h3. Vazby

* {{Okruh (1) → RPO_Okruh_Rozvrh (N)}}

h3. Pravidla a poznámky

* V ER je vazba na RPO realizována přes {{RPO_Okruh_Rozvrh}} (nikoli přímou M:N tabulkou jen pro Okruh).
* Změna přiřazení okruhu na RPO se řeší historizací v {{RPO_Okruh_Rozvrh}}.

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* Okruh je stavební kámen operativního plánování v RP (tvorba okruhů dne / denních výkonů).
* Vazba {{RPO → Okruh}} je v konceptu businessově nepovinná; bez ní lze OS generovat, ale s nižší mírou automatizace zařazení.
* Etapa 1: SoT okruhů a vazeb je HEN (import do PP, read-only vazby).
* Etapa 2+: PP přebírá správu okruhů a vazeb RPO na okruhy, včetně zakládání/editace/deaktivace.
* Okruh může mít přiřazené pravidelné vozidlo, což usnadňuje generování a automatické zařazení OS do denních výkonů.

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{okruh}}

*Sloupce (návrh):*
||Položka||Popis||
|id bigint identity primary key|—|
|nazev nvarchar(255) not null|—|
|provozovna_id bigint not null|—|
|vozidlo_id bigint null|—|
|aktivni bit not null default 1|—|
|audit/tenant sloupce dle standardu PP|—|

*FK (návrh):*
||FK sloupec||Reference / poznámka||
|provozovna_id|→ {{organization_unit(id)}}|
|vozidlo_id|→ {{vehicle(id)}} / ekvivalent DS (pokud se vede v PP)|

*Indexy (návrh):*
||Index||Definice / poznámka||
|IX_okruh_provozovna|({{provozovna_id}})|
|IX_okruh_aktivni|({{aktivni}})|
|UX_okruh_provozovna_nazev|({{provozovna_id}}, {{nazev}}) [unikátní dle business rozhodnutí]|

----

h2. 3. Rozvrh

*Stav:* Nová entita pro CS  
*Role:* Řízení kalendářních dnů, na které se generují objednané služby z RPO.

h3. Klíčové atributy

* {{id}} (PK)
* {{nazev}}
* {{provozovna_id}} (FK)
* {{platnost_od}}, {{platnost_do}}

h3. Vazby

* {{Rozvrh (1) → Kalendar (N)}}
* {{Rozvrh (1) → RPO_Okruh_Rozvrh (N)}}

h3. Pravidla a poznámky

* V ER v1 je Rozvrh v PP označen jako read-only (zdroj pravdy HEN).
* Změna přiřazení rozvrhu k RPO se historizuje v {{RPO_Okruh_Rozvrh}}.

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* Rozvrh řídí konkrétní kalendářní dny, na které má vzniknout objednaná služba z RPO.
* Rozvrhy a kalendářní svozy jsou v Etapě 1 spravovány v HEN; v PP jsou primárně zobrazovány.
* Rozvrh je vázán na provozovnu a pracuje s platností (businessově významný atribut).
* Vazba {{RPO → Rozvrh}} je podmínkou generování OS v RP.
* V dalších etapách má PP řídit vazbu RPO na rozvrh a stát se SoT pro vazby (se synchronizací do HEN).

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{rozvrh}}

*Sloupce (návrh):*
||Položka||Popis||
|id bigint identity primary key|—|
|nazev nvarchar(255) not null|—|
|provozovna_id bigint not null|—|
|platnost_od date null|—|
|platnost_do date null|—|
|audit/tenant sloupce dle standardu PP|—|

*FK (návrh):*
||FK sloupec||Reference / poznámka||
|provozovna_id|→ {{organization_unit(id)}}|

*Indexy (návrh):*
||Index||Definice / poznámka||
|IX_rozvrh_provozovna|({{provozovna_id}})|
|IX_rozvrh_platnost|({{platnost_od}}, {{platnost_do}})|
|UX_rozvrh_provozovna_nazev|({{provozovna_id}}, {{nazev}}) [unikátní dle business rozhodnutí]|

----

h2. 4. Kalendar

*Stav:* Nová entita pro CS  
*Role:* Konkrétní kalendářní den obsluhy přiřazený k Rozvrhu.

h3. Klíčové atributy

* {{id}} (PK)
* {{rozvrh_id}} (FK → {{Rozvrh}})
* {{datum}}
* {{typ_dne}}

h3. Vazby

* {{Kalendar (N) → Rozvrh (1)}}

h3. Pravidla a poznámky

* Entita reprezentuje provozní dny, ve kterých se reálně sváží.

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* Kalendář reprezentuje konkrétní kalendářní den obsluhy navázaný na rozvrh.
* Kalendářní dny vstupují do generování objednaných služeb v RP (OS vznikají pro dny v období generování).
* Generování OS musí zajistit, že pro stejnou kombinaci RPO/kalendářního dne nevznikne duplicitní OS.
* Kalendář je businessově odvozen od Rozvrhu (správa v kontextu rozvrhů, v CK primárně HEN).

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{kalendar}}

*Sloupce (návrh):*
||Položka||Popis||
|id bigint identity primary key|—|
|rozvrh_id bigint not null|—|
|datum date not null|—|
|typ_dne nvarchar(50) null|—|
|audit/tenant sloupce dle standardu PP|—|

*FK (návrh):*
||FK sloupec||Reference / poznámka||
|rozvrh_id|→ {{rozvrh(id)}}|

*Indexy (návrh):*
||Index||Definice / poznámka||
|IX_kalendar_rozvrh|({{rozvrh_id}})|
|UX_kalendar_rozvrh_datum|({{rozvrh_id}}, {{datum}})|
|IX_kalendar_datum|({{datum}})|

----

h2. 5. Zóna

*Stav:* Nová entita pro CS  
*Role:* Geografické seskupení předmětů smluv HEN pro filtraci a plánování.

h3. Klíčové atributy

* {{id}} (PK)
* {{nazev}}
* {{provozovna_id}} (FK)
* {{geometrie}}

h3. Vazby

* {{RPO (N) → Zóna (1)}}

h3. Pravidla a poznámky

* Aktuální kardinalita v ER v1 je *N:1* (více RPO může spadat do jedné zóny).
* Zóna je samostatná entita, nepřebírá se kompletní adresní struktura HEN.

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* Zóna geograficky sdružuje předměty smluv HEN a slouží jako pomocný filtr pro plánování.
* Vazba {{RPO → Zóna}} je businessově *nepovinná* (zóna je pomocná, nikoli podmínka generování OS).
* V Etapě 1 jsou zóny importovány z HEN a PP eviduje vazby RPO na zóny.
* V dalších etapách roste význam PP pro správu zón a přiřazení RPO (spolu s okruhy).
* Zóna se využívá v PP pro evidenci, filtrování a mapové zobrazení; v RP pro výběr RPO při generování OS.

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{zone}} (nebo {{area_zone}})

*Sloupce (návrh):*
||Položka||Popis||
|id bigint identity primary key|—|
|name nvarchar(255) not null|—|
|external_id nvarchar(255) null|—|
|organization_unit_id bigint not null|—|
|geometry nvarchar(max) null|/ {{geometry}} (SQL typ dle platformy)|
|note nvarchar(255) null|—|
|is_active bit not null default 1|—|
|audit/tenant sloupce dle standardu PP|—|

*FK (návrh):*
||FK sloupec||Reference / poznámka||
|organization_unit_id|→ {{organization_unit(id)}}|

*Indexy (návrh):*
||Index||Definice / poznámka||
|IX_zone_organization_unit|({{organization_unit_id}})|
|IX_zone_external_id|({{external_id}})|
|UX_zone_org_name|({{organization_unit_id}}, {{name}}) [doporučeno]|
|prostorový index na {{geometry}} (pokud bude fyzicky geodatový typ)|—|

----

h2. 6. Adresy

*Stav:* Existující — rozšíření pro CS  
*Role:* Referenční evidence adres používaná mj. pro místo realizace na RPO.

h3. Klíčové atributy

* {{id}} (PK)
* {{zeme}}
* {{mesto}}
* {{ulice}}
* {{cp}}
* {{co}}
* {{X}} (nový atribut pro CS)
* {{Y}} (nový atribut pro CS)

h3. Vazby

* {{RPO (N) → Adresy (1)}}

h3. Pravidla a poznámky

* {{RPO}} ukládá referenci {{misto_realizace_adresa_id}} namísto textového pole.
* Entita je v ER v1 označena zeleně jako *rozšiřovaná existující entita*.

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* Adresa místa realizace je součást minimální sady dat potřebné pro plánování OS v RP.
* Na mapě PP se má zobrazovat lokalita RPO:
** primárně přes stanoviště navázané nádoby
** fallback přes geokódovanou adresu místa realizace RPO
* Při změně adresy místa realizace na RPO má být zajištěna aktualizace geokódované lokace.
* Businessově je potřeba rozlišit body „Místo realizace“ vs. „Stanoviště nádoby“ (vizualizace / zdroj lokace).

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{adresy}} (název dle existujícího DS)

*Sloupce (návrh):*
||Položka||Popis||
|id bigint identity primary key|—|
|zeme nvarchar(100) null|—|
|mesto nvarchar(255) null|—|
|ulice nvarchar(255) null|—|
|cp nvarchar(20) null|—|
|co nvarchar(20) null|—|
|x decimal(18,6) null|—|
|y decimal(18,6) null|—|
|audit/tenant sloupce dle standardu PP|—|

*FK (návrh):*
||FK sloupec||Reference / poznámka||
|bez nových FK povinných pro CS (dle aktuálního ER)|—|

*Indexy (návrh):*
||Index||Definice / poznámka||
|IX_adresy_mesto|({{mesto}})|
|IX_adresy_ulice|({{ulice}})|
|IX_adresy_cp_co|({{cp}}, {{co}})|
|IX_adresy_xy|({{x}}, {{y}}) [pokud bude využito prostorové dohledávání]|

----

h2. 7. Skupina odpadu

*Stav:* Existující — rozšíření pro CS  
*Role:* Seskupení druhů odpadu pro plánování a výběr RPO; důležitý filtr v CS/RP.

h3. Klíčové atributy

* {{id}} (PK)
* {{nazev}}
* {{provozovna_id}} (FK)
* {{je_mix}} (kombinovaný svoz)

h3. Vazby

* {{RPO (N) → Skupina odpadu (1)}}
* {{Druh odpadu (N) → Skupina odpadu (1)}}
* {{Nádoba (N) → Skupina odpadu (1)}} (volitelná fallback vazba, přerušovaně v ER)

h3. Pravidla a poznámky

* V aktuálním ER v1 je {{Skupina odpadu}} evidována *per provozovna* ({{provozovna_id}}).
* {{Nádoba.skupina_odpadu_id}} slouží jako fallback, pokud nádoba není navázána na {{RPO}}.
* {{je_mix}} reprezentuje podporu kombinovaného svozu.

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* Skupina odpadu slouží ke zjednodušení výběru RPO pro operativní i strategické plánování.
* Vazba {{Druh odpadu → Skupina odpadu}} řídí automatické vyplnění skupiny odpadu na RPO.
* Skupina odpadu je v CK evidována per provozovna.
* Skupiny podporují mísitelnost druhů odpadů (sdružení kompatibilních druhů do jedné plánovací skupiny).
* Business pravidlo v RP: jeden okruh dne obsahuje objednané služby pouze jedné skupiny odpadu.
* Speciální skupina {{MIX}}:
** plněna automaticky při splnění podmínky v HEN
** nepřepisuje se standardním mapováním
** není dostupná pro běžné přiřazení v číselníku Druhy odpadu

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{garbage_group}} (rozšíření existující)

*Sloupce (návrh / delta):*
||Položka||Popis||
|stávající sloupce entity {{garbage_group}}|—|
|organization_unit_id bigint null/not null|(dle rozhodnutí migrace na per-provozovna evidenci)|
|is_mix bit not null default 0|/ mapování na stávající {{je_mix}}|

*FK (návrh):*
||FK sloupec||Reference / poznámka||
|organization_unit_id|→ {{organization_unit(id)}} (pokud se fyzicky zavede per-provozovna vazba)|

*Indexy (návrh):*
||Index||Definice / poznámka||
|IX_garbage_group_organization_unit|({{organization_unit_id}})|
|IX_garbage_group_is_mix|({{is_mix}})|
|UX_garbage_group_org_name|({{organization_unit_id}}, {{name}}) [doporučeno při per-provozovna evidenci]|

----

h2. 8. Druh odpadu

*Stav:* Existující — rozšíření pro CS  
*Role:* Číselník druhů odpadu používaný pro klasifikaci a odvození skupin odpadu.

h3. Klíčové atributy

* {{id}} (PK)
* {{kod}}
* {{nazev}}
* {{skupina_odpadu_id}} (FK → {{Skupina odpadu}})

h3. Vazby

* {{Druh odpadu (N) → Skupina odpadu (1)}}
* {{RPO (N) → Druh odpadu (1)}} (přes atribut na RPO)

h3. Pravidla a poznámky

* ER v1 modeluje vazbu na {{Skupina odpadu}} přímo přes FK {{skupina_odpadu_id}}.
* Entita slouží jako číselník pro přehled a klasifikaci odpadů.

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* Číselník Druhy odpadu je místo, kde se v PP spravuje vazba {{Druh odpadu → Skupina odpadu}}.
* Vazba slouží jako vstup pro automatické vyplnění skupiny odpadu na RPO při synchronizaci z HEN.
* RP čerpá výsledek (skupinu odpadu na RPO / v plánování) read-only; správa mapování je v PP.
* U skupiny {{MIX}} platí omezení nabídky a speciální chování dle pravidel kapitoly Skupina odpadu.

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{garbage_type}} (rozšíření existující)

*Sloupce (návrh / delta):*
||Položka||Popis||
|stávající sloupce entity {{garbage_type}}|—|
|garbage_group_id bigint null|(pokud bude vazba vedena přímo dle ER v1)|

*FK (návrh):*
||FK sloupec||Reference / poznámka||
|garbage_group_id|→ {{garbage_group(id)}}|

*Indexy (návrh):*
||Index||Definice / poznámka||
|IX_garbage_type_garbage_group|({{garbage_group_id}})|
|UX_garbage_type_code|({{code}}) [pokud již neexistuje]|

----

h2. 9. Nádoba

*Stav:* Existující — rozšíření pro CS  
*Role:* Malá odpadová nádoba pro cyklické svozy; provozní objekt navázaný na RPO / stanoviště / typ.

h3. Klíčové atributy

* {{id}} (PK)
* {{rfid}}
* {{typ_nadoby_id}} (FK → {{Typ nádoby}})
* {{stanoviste_id}} (FK → {{Stanoviště}})
* {{rpo_id}} (FK → {{RPO}})
* {{skupina_odpadu_id}} (FK → {{Skupina odpadu}}, vlastní fallback bez RPO)
* {{objem_litry}}

h3. Vazby

* {{Nádoba (N) → RPO (1)}}
* {{Nádoba (N) → Stanoviště (1)}}
* {{Nádoba (N) → Typ nádoby (1)}}
* {{Nádoba (N) → Skupina odpadu (1)}} (volitelná / fallback, přerušovaná vazba v ER)

h3. Pravidla a poznámky

* Primární business kontext nádoba typicky dědí z RPO.
* Současně je explicitně umožněno uložit vlastní {{skupina_odpadu_id}} pro případy bez vazby na RPO.

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* PP již eviduje nádoby v detailu potřebném pro plánování CS; rozšíření se týká vazeb a atributů pro plánování.
* Nádoba dědí z navázané RPO informace o okruhu, rozvrhu a zóně (zobrazení a plánovací kontext).
* Nádoba vstupuje do generování OS a lokalizace tras; pokud není k dispozici stanoviště, používá se fallback přes místo realizace RPO.
* Při navázání nádoby na RPO musí být ošetřen dopad na skupinu odpadu nádoby (prevence nekonzistence).

h3. Aplikační pravidla priority — {{Nádoba.skupina_odpadu_id}} vs {{RPO.skupina_odpadu_id}}

# Pokud je {{Nádoba}} navázána na {{RPO}} a {{RPO.skupina_odpadu_id}} je vyplněna, *řídicí hodnota je z RPO*.
# Přímé vyplnění {{Nádoba.skupina_odpadu_id}} je povoleno pouze pokud:
** nádoba není navázána na RPO, nebo
** navázané RPO nemá skupinu odpadu vyplněnu.
# Při navázání nádoby na RPO má aplikace po upozornění uživatele přepsat skupinu odpadu nádoby hodnotou z RPO (pokud je na RPO vyplněna).
# Pokud není skupina odpadu dostupná ani na navázaném RPO ani na nádobě, výsledek je {{NULL}} a záznam je kandidát na datovou kontrolu.
# Kontrolní pravidlo (doporučeno):
** pokud existuje vazba na RPO a současně je na nádobě odlišná skupina odpadu, systém eviduje nesoulad (warning / audit) a aplikačně preferuje hodnotu z RPO.

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{nadoba}} (rozšíření existující)

*Sloupce (návrh / delta):*
||Položka||Popis||
|stávající sloupce entity {{nadoba}}|—|
|rpo_id bigint null|—|
|stanoviste_id bigint null|—|
|typ_nadoby_id bigint null|—|
|skupina_odpadu_id bigint null|(nová fallback reference)|
|rfid nvarchar(100) null|—|
|objem_litry decimal(10,2) null|—|

*FK (návrh):*
||FK sloupec||Reference / poznámka||
|rpo_id|→ {{rpo(id)}}|
|stanoviste_id|→ {{stanoviste(id)}}|
|typ_nadoby_id|→ {{typ_nadoby(id)}}|
|skupina_odpadu_id|→ {{garbage_group(id)}}|

*Indexy (návrh):*
||Index||Definice / poznámka||
|IX_nadoba_rpo|({{rpo_id}})|
|IX_nadoba_stanoviste|({{stanoviste_id}})|
|IX_nadoba_typ|({{typ_nadoby_id}})|
|IX_nadoba_skupina_odpadu|({{skupina_odpadu_id}})|
|UX_nadoba_rfid|({{rfid}}) [pokud je RFID unikátní]|

----

h2. 10. Stanoviště

*Stav:* Existující — rozšíření pro CS  
*Role:* Fyzické umístění nádob a lokalizace objednaných služeb.

h3. Klíčové atributy

* {{id}} (PK)
* {{adresa}}
* {{gps_lat}}
* {{gps_lon}}

h3. Vazby

* {{Nádoba (N) → Stanoviště (1)}} (z pohledu nádoby)

h3. Pravidla a poznámky

* V ER v1 *není přímá vazba Stanoviště → RPO*.
* Atribut {{rpo_id}} byl z návrhu odstraněn.
* Vazba na RPO je nepřímá přes {{Nádoba}}.

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* Stanoviště je klíčová lokalizační entita pro pasportizaci nádob a plánování/realizaci CS.
* PP a FOB používají stanoviště pro mapové zobrazení a sledování obslouženosti.
* Při zobrazení RPO na mapě se preferuje lokalita stanovišť navázaných nádob; místo realizace RPO je fallback.
* Stanoviště vstupují do generování lokací OS a do plánování tras v RP/FOB.

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{stanoviste}} (rozšíření existující)

*Sloupce (návrh / delta):*
||Položka||Popis||
|stávající sloupce entity {{stanoviste}}|—|
|adresa nvarchar(500) null|—|
|gps_lat decimal(10,7) null|—|
|gps_lon decimal(10,7) null|—|
|bez {{rpo_id}} (odebráno z návrhu)|—|

*FK (návrh):*
||FK sloupec||Reference / poznámka||
|bez přímé FK na {{rpo}}|—|

*Indexy (návrh):*
||Index||Definice / poznámka||
|IX_stanoviste_gps|({{gps_lat}}, {{gps_lon}})|
|IX_stanoviste_adresa|({{adresa}}) [fulltext / prefix dle potřeby]|

----

h2. 11. Typ nádoby

*Stav:* Existující — rozšíření pro CS  
*Role:* Číselník typů nádob; zdroj parametrů pro plánování a výpočet obsluhy.

h3. Klíčové atributy

* {{id}} (PK)
* {{nazev}}
* {{objem_litry}}
* {{cas_obsluhy_sec}} (nový / editovatelný pro CS)

h3. Vazby

* {{Nádoba (N) → Typ nádoby (1)}}

h3. Pravidla a poznámky

* ER v1 uvádí zdroj ERP HEN/WINX (read-only) s editovatelným {{cas_obsluhy_sec}}.

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* Typy nádob jsou existující číselník (HEN/WINX) rozšířený o businessově významný atribut času obsluhy.
* {{cas_obsluhy_sec}} slouží pro výpočet plánované doby realizace okruhu dne v RP.
* V dalších etapách se očekává rozšíření typů nádob o parametry pro kapacitní výpočty (váha/objem) pro strategickou optimalizaci.

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{typ_nadoby}} (rozšíření existující)

*Sloupce (návrh / delta):*
||Položka||Popis||
|stávající sloupce entity {{typ_nadoby}}|—|
|objem_litry decimal(10,2) null|—|
|cas_obsluhy_sec int null|(nový / editovatelný)|

*FK (návrh):*
||FK sloupec||Reference / poznámka||
|bez nové FK v rámci CS|—|

*Indexy (návrh):*
||Index||Definice / poznámka||
|IX_typ_nadoby_objem|({{objem_litry}})|
|IX_typ_nadoby_cas_obsluhy|({{cas_obsluhy_sec}})|
|UX_typ_nadoby_nazev|({{nazev}}) [pokud již neexistuje]|

----

h2. 12. RPO_Okruh_Rozvrh

*Stav:* Nová entita pro CS  
*Role:* Temporální vazební tabulka pro přiřazení {{RPO ↔ Okruh ↔ Rozvrh}} s historií změn.

h3. Klíčové atributy

* {{id}} (PK)
* {{rpo_id}} (FK → {{RPO}})
* {{okruh_id}} (FK → {{Okruh}})
* {{rozvrh_id}} (FK → {{Rozvrh}})
* {{platnost_od}}
* {{platnost_do}} ({{NULL = aktivní}})

h3. Vazby

* {{RPO (1) → RPO_Okruh_Rozvrh (N)}}
* {{Okruh (1) → RPO_Okruh_Rozvrh (N)}}
* {{Rozvrh (1) → RPO_Okruh_Rozvrh (N)}}

h3. Pravidla a poznámky

* Změna {{Okruhu}} nebo {{Rozvrhu}} na RPO:
** ukončí aktuální vazbu ({{platnost_do}})
** založí nový záznam v {{RPO_Okruh_Rozvrh}}
* V jednom čase má mít {{RPO}} maximálně jednu aktivní vazbu.
* {{RPO.rpo_okruh_rozvrh_id}} může sloužit jako rychlá reference na aktuálně aktivní záznam.

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* Vazební entita reprezentuje business koncept přiřazení RPO do okruhu a rozvrhu s časovou platností.
* V Etapě 1 je vazba synchronizována z HEN; v dalších etapách má být správa přiřazení přesunuta do PP.
* Změny přiřazení RPO k okruhu/rozvrhu musí být dohledatelné v čase (dopad na generování OS a plánování).
* Při změně vazeb na RPO musí systém zohlednit dopad na generované OS/lokace (deaktivace / doplnění dle období generování).
* Vazba {{RPO ↔ Okruh ↔ Rozvrh}} přímo ovlivňuje:
** vznik OS v RP (nutná vazba na rozvrh)
** automatické seskupení OS do okruhů dne (vazba na okruh)

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{rpo_okruh_rozvrh}}

*Sloupce (návrh):*
||Položka||Popis||
|id bigint identity primary key|—|
|rpo_id bigint not null|—|
|okruh_id bigint not null|—|
|rozvrh_id bigint not null|—|
|platnost_od datetime2(0) not null|—|
|platnost_do datetime2(0) null|—|
|audit/tenant sloupce dle standardu PP|—|

*FK (návrh):*
||FK sloupec||Reference / poznámka||
|rpo_id|→ {{rpo(id)}}|
|okruh_id|→ {{okruh(id)}}|
|rozvrh_id|→ {{rozvrh(id)}}|

*Indexy (návrh):*
||Index||Definice / poznámka||
|IX_ror_rpo|({{rpo_id}})|
|IX_ror_okruh|({{okruh_id}})|
|IX_ror_rozvrh|({{rozvrh_id}})|
|IX_ror_platnost|({{platnost_od}}, {{platnost_do}})|
|UX_ror_rpo_aktivni|filtrovaný index na {{rpo_id}} kde {{platnost_do is null}} (zajištění max 1 aktivní vazby)|
|IX_ror_rpo_historie|({{rpo_id}}, {{platnost_od}} desc)|

----

h2. Doporučení pro další rozpracování DS/DDL

* U každé kapitoly doplnit fyzický DDL návrh (typy sloupců, indexy, FK, {{NULL/NOT NULL}}).
* U temporální tabulky {{RPO_Okruh_Rozvrh}} doplnit constraint / pravidlo pro „max 1 aktivní vazba na RPO“.
* U {{Nádoba.skupina_odpadu_id}} popsat prioritu vyhodnocení vůči {{RPO.skupina_odpadu_id}} v aplikační logice.
* U {{Adresy}} doplnit význam atributů {{X}}, {{Y}} (souřadnice / lokální souřadný systém / jiný význam).
