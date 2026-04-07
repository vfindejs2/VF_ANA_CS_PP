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
# [Nadoba_Stanoviste|#13-nadoba_stanoviste]
# [Skupina_Druh_odpadu|#14-skupina_druh_odpadu]

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

*Stav:* Existující — rozšíření pro CS \\
*Role:* Centrální entita CS (revize položky objednávky), základ pro plánování, filtraci a vazby na provozní objekty.

h3. Klíčové atributy

* {{id}} (PK)
* {{kod_polozky}} (z HEN)
* {{misto_realizace_adresa_id}} (FK → {{Adresy}})  
  Pozn.: nahrazuje původní textové {{misto_realizace}}.
* {{druh_odpadu_id}} (FK → {{Druh odpadu}})
* {{typ_nadoby_id}} (FK → {{Typ nádoby}})
* {{skupina_odpadu_id}} (FK → {{Skupina odpadu}})
* {{zona_id}} (FK → {{Zóna}})
* {{provozovna_id}} (FK)
* {{platnost_od}}, {{platnost_do}}
* {{stav}}

h3. Vazby

* {{RPO (N) → Zóna (1)}}
* {{RPO (N) → Adresy (1)}}
* {{RPO (N) → Skupina odpadu (1)}}
* {{RPO (N) → Typ nádoby (1)}}
* {{RPO (1) → Nádoba (N)}}
* {{RPO (1) → RPO_Okruh_Rozvrh (N)}} (historie přiřazení)

h3. Pravidla a poznámky

* {{RPO}} je nositel obchodního kontextu pro obsluhu místa realizace.
* Historie přiřazení {{Okruh + Rozvrh}} je vedena v {{RPO_Okruh_Rozvrh}}; {{RPO}} v aktuálním ER nenese přímý FK na „aktivní“ záznam této vazby.
* Vazba na {{Stanoviště}} je *nepřímá přes {{Nádobu}} a vazební entitu {{Nadoba_Stanoviste}}*.

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* {{RPO}} je jednoznačný podklad o způsobu obsluhy místa realizace nebo navázaných nádob.
* V Etapě 1 jsou data RPO a vazeb na okruh/rozvrh/zónu synchronizována z HEN (PP je eviduje a zobrazuje).
* {{RPO}} je klíčový vstup pro generování objednaných služeb v RP (obsahuje mj. adresu/souřadnice MR, druh/skupinu odpadu, okruh, rozvrh, kalendářní dny).
* Vazba {{RPO → Rozvrh}} je nutnou podmínkou pro vygenerování objednané služby; vazba {{RPO → Okruh}} určuje míru automatizace zařazení do okruhu dne.
* V dalších etapách roste význam PP jako místa správy přiřazení RPO do okruhů/zón/rozvrhů (s dopadem na plánování).

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{rpo}}

*Mapování atributů na DDL dle {{inventar-entit-PP.md}} (entita: Revize položky objednávky / {{order_item_revision}}):*
||Atribut (ER)||DDL sloupec v inventáři PP||Stav||Poznámka||
|{{id}}|{{id}}|nalezeno|přímá shoda|
|{{kod_polozky}}|{{order_item_id}} + {{revision}}|částečně|v inventáři není samostatný sloupec {{kod_polozky}}; ER atribut je business zkratka|
|{{misto_realizace_adresa_id}}|{{realization_address_id}}|nalezeno|přímá shoda významu|
|{{druh_odpadu_id}}|{{garbage_type_id}}|nalezeno|přímá shoda významu|
|{{typ_nadoby_id}}|{{container_type_id}}|nalezeno|přímá shoda významu|
|{{skupina_odpadu_id}}|—|nenalezeno|CS rozšíření nad rámec inventáře RPO|
|{{zona_id}}|—|nenalezeno|CS rozšíření nad rámec inventáře RPO|
|{{provozovna_id}}|—|nenalezeno|v inventáři DDL {{order_item_revision}} není přímý sloupec provozovny|
|{{platnost_od}}|{{valid_from}}|nalezeno|přímá shoda významu|
|{{platnost_do}}|{{valid_to}}|nalezeno|přímá shoda významu|
|{{stav}}|{{is_active}}|částečně|ER „stav“ je širší koncept než systémové {{is_active}}|

*Sloupce (návrh):*
||Položka||Popis||
|id bigint identity primary key|—|
|kod_polozky nvarchar(100) not null|—|
|misto_realizace_adresa_id bigint null|—|
|druh_odpadu_id bigint null|—|
|typ_nadoby_id bigint null|—|
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
|druh_odpadu_id|→ {{garbage_type(id)}} / ekvivalent DS|
|typ_nadoby_id|→ {{typ_nadoby(id)}} / ekvivalent DS|
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
|IX_rpo_typ_nadoby|({{typ_nadoby_id}})|
|IX_rpo_adresa|({{misto_realizace_adresa_id}})|

----

h2. 2. Okruh

*Stav:* Nová entita pro CS \\
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

*Mapování atributů na DDL dle {{inventar-entit-PP.md}}:*
||Atribut (ER)||DDL sloupec v inventáři PP||Stav||Poznámka||
|{{id}}, {{nazev}}, {{provozovna_id}}, {{vozidlo_id}}, {{aktivni}}|—|nenalezeno|entita je v inventáři PP vedena jen jako nová CS entita bez DDL (sekce 23)|

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

*Stav:* Nová entita pro CS \\
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

*Mapování atributů na DDL dle {{inventar-entit-PP.md}}:*
||Atribut (ER)||DDL sloupec v inventáři PP||Stav||Poznámka||
|{{id}}, {{nazev}}, {{provozovna_id}}, {{platnost_od}}, {{platnost_do}}|—|nenalezeno|entita je v inventáři PP vedena jen jako nová CS entita bez DDL (sekce 23)|

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

*Stav:* Nová entita pro CS \\
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

*Mapování atributů na DDL dle {{inventar-entit-PP.md}}:*
||Atribut (ER)||DDL sloupec v inventáři PP||Stav||Poznámka||
|{{id}}, {{rozvrh_id}}, {{datum}}, {{typ_dne}}|—|nenalezeno|entita je v inventáři PP vedena jen jako nová CS entita bez DDL (sekce 23)|

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

*Stav:* Nová entita pro CS \\
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

*Mapování atributů na DDL dle {{inventar-entit-PP.md}}:*
||Atribut (ER)||DDL sloupec v inventáři PP||Stav||Poznámka||
|{{id}}, {{nazev}}, {{provozovna_id}}, {{geometrie}}|—|nenalezeno|entita je v inventáři PP vedena jen jako nová CS entita bez DDL (sekce 23)|

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

*Stav:* Existující — rozšíření pro CS \\
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

*Mapování atributů na DDL dle {{inventar-entit-PP.md}} (entita: Adresa / {{address}}):*
||Atribut (ER)||DDL sloupec v inventáři PP||Stav||Poznámka||
|{{id}}|{{id}}|nalezeno|přímá shoda|
|{{zeme}}|{{country}}|nalezeno|přímá shoda významu|
|{{mesto}}|{{city}}|nalezeno|přímá shoda významu|
|{{ulice}}|{{street}}|nalezeno|přímá shoda významu|
|{{cp}}|{{registry_number}}|nalezeno|číslo popisné|
|{{co}}|{{orientation_number}}|nalezeno|číslo orientační|
|{{X}}|—|nenalezeno|CS rozšíření v ER, inventář {{address}} uvádí bez těchto sloupců|
|{{Y}}|—|nenalezeno|CS rozšíření v ER, inventář {{address}} uvádí bez těchto sloupců|

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

*Stav:* Existující — rozšíření pro CS \\
*Role:* Seskupení druhů odpadu pro plánování a výběr RPO; důležitý filtr v CS/RP.

h3. Klíčové atributy

* {{id}} (PK)
* {{nazev}}
* {{provozovna_id}} (FK)
* {{je_mix}} (kombinovaný svoz)

h3. Vazby

* {{RPO (N) → Skupina odpadu (1)}}
* {{Skupina odpadu (1) → Skupina_Druh_odpadu (N)}} (mapování druhů na skupiny přes vazební entitu)
* {{Nádoba (N) → Skupina odpadu (1)}} (volitelná fallback vazba, přerušovaně v ER)

h3. Pravidla a poznámky

* V aktuálním ER v1 je {{Skupina odpadu}} evidována *per provozovna* ({{provozovna_id}}).
* Vazba na {{Druh odpadu}} není v ER vedena přímo; je modelována přes temporální vazební entitu {{Skupina_Druh_odpadu}}.
* {{Nádoba.skupina_odpadu_id}} slouží jako fallback, pokud nádoba není navázána na {{RPO}}.
* {{je_mix}} reprezentuje podporu kombinovaného svozu.

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* Skupina odpadu slouží ke zjednodušení výběru RPO pro operativní i strategické plánování.
* Mapování {{Druh odpadu ↔ Skupina odpadu}} (v návrhu přes {{Skupina_Druh_odpadu}}) řídí automatické vyplnění skupiny odpadu na RPO.
* Skupina odpadu je v CK evidována per provozovna.
* Skupiny podporují mísitelnost druhů odpadů (sdružení kompatibilních druhů do jedné plánovací skupiny).
* Business pravidlo v RP: jeden okruh dne obsahuje objednané služby pouze jedné skupiny odpadu.
* Speciální skupina {{MIX}}:
** plněna automaticky při splnění podmínky v HEN
** nepřepisuje se standardním mapováním
** není dostupná pro běžné přiřazení v číselníku Druhy odpadu

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{garbage_group}} (rozšíření existující)

*Mapování atributů na DDL dle {{inventar-entit-PP.md}} (entita: Skupina odpadu / {{garbage_group}}):*
||Atribut (ER)||DDL sloupec v inventáři PP||Stav||Poznámka||
|{{id}}|{{id}}|nalezeno|přímá shoda|
|{{nazev}}|{{name}}|nalezeno|přímá shoda významu|
|{{provozovna_id}}|—|nenalezeno|ER CS rozšíření; inventář {{garbage_group}} neuvádí přímou vazbu na provozovnu|
|{{je_mix}}|—|nenalezeno|ER CS rozšíření; inventář neuvádí explicitní příznak mixu|

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

*Stav:* Existující — rozšíření pro CS \\
*Role:* Číselník druhů odpadu používaný pro klasifikaci a odvození skupin odpadu.

h3. Klíčové atributy

* {{id}} (PK)
* {{kod}}
* {{nazev}}

h3. Vazby

* {{Druh odpadu (1) → Skupina_Druh_odpadu (N)}} (mapování na skupiny přes vazební entitu)
* {{RPO (N) → Druh odpadu (1)}} (přes atribut na RPO)

h3. Pravidla a poznámky

* ER v1 modeluje vazbu na {{Skupina odpadu}} nepřímo přes vazební entitu {{Skupina_Druh_odpadu}} (temporální mapování).
* Entita slouží jako číselník pro přehled a klasifikaci odpadů.

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* Číselník Druhy odpadu je součást správy mapování {{Druh odpadu ↔ Skupina odpadu}} v PP.
* Vazba slouží jako vstup pro automatické vyplnění skupiny odpadu na RPO při synchronizaci z HEN.
* RP čerpá výsledek (skupinu odpadu na RPO / v plánování) read-only; správa mapování je v PP.
* U skupiny {{MIX}} platí omezení nabídky a speciální chování dle pravidel kapitoly Skupina odpadu.

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{garbage_type}} (rozšíření existující)

*Mapování atributů na DDL dle {{inventar-entit-PP.md}} (entita: Druh odpadu / {{garbage_type}}):*
||Atribut (ER)||DDL sloupec v inventáři PP||Stav||Poznámka||
|{{id}}|{{id}}|nalezeno|přímá shoda|
|{{kod}}|{{code}}|nalezeno|přímá shoda významu|
|{{nazev}}|{{description}}|částečně|inventář používá spíše „Popis“ než samostatný {{name}}|
|vazba přes {{Skupina_Druh_odpadu}}|—|nenalezeno|inventář {{garbage_type}} neobsahuje mapování na skupinu odpadu|

*Sloupce (návrh / delta):*
||Položka||Popis||
|stávající sloupce entity {{garbage_type}}|—|
|bez přímého FK na {{garbage_group}} (mapování je vedeno přes {{skupina_druh_odpadu}})|—|

*FK (návrh):*
||FK sloupec||Reference / poznámka||
|bez nové přímé FK na {{garbage_group}}|—|

*Indexy (návrh):*
||Index||Definice / poznámka||
|UX_garbage_type_code|({{code}}) [pokud již neexistuje]|

----

h2. 9. Nádoba

*Stav:* Existující — rozšíření pro CS \\
*Role:* Malá odpadová nádoba pro cyklické svozy; provozní objekt navázaný na RPO, lokalizovaný přes vazbu na stanoviště.

h3. Klíčové atributy

* {{id}} (PK)
* {{rfid}}
* {{rpo_id}} (FK → {{RPO}})
* {{skupina_odpadu_id}} (FK → {{Skupina odpadu}}, vlastní fallback bez RPO)
* {{objem_litry}}

h3. Vazby

* {{Nádoba (N) → RPO (1)}}
* {{Nádoba (1) → Nadoba_Stanoviste (N)}} (časově platné přiřazení ke stanovištím)
* {{Nádoba (N) → Skupina odpadu (1)}} (volitelná / fallback, přerušovaná vazba v ER)

h3. Pravidla a poznámky

* Primární business kontext nádoba typicky dědí z RPO.
* Typ nádoby je v aktuálním ER navázán na {{RPO}} (nikoli přímo na {{Nádobu}}).
* Vazba na {{Stanoviště}} je v aktuálním ER vedena přes existující vazební entitu {{Nadoba_Stanoviste}}.
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

*Mapování atributů na DDL dle {{inventar-entit-PP.md}} (entita: Nádoba / {{container}}):*
||Atribut (ER)||DDL sloupec v inventáři PP||Stav||Poznámka||
|{{id}}|{{id}}|nalezeno|přímá shoda|
|{{rfid}}|—|nenalezeno|v inventáři DDL {{container}} není uveden samostatný sloupec RFID|
|{{rpo_id}}|—|nenalezeno|ER CS rozšíření; inventář vazbu řeší přes jinou vazební entitu|
|{{skupina_odpadu_id}}|{{garbage_group_id}}|nalezeno|přímá shoda významu|
|{{objem_litry}}|—|nenalezeno|v inventáři DDL {{container}} není přímý sloupec objemu nádoby|

*Sloupce (návrh / delta):*
||Položka||Popis||
|stávající sloupce entity {{nadoba}}|—|
|rpo_id bigint null|—|
|skupina_odpadu_id bigint null|(nová fallback reference)|
|rfid nvarchar(100) null|—|
|objem_litry decimal(10,2) null|—|

*FK (návrh):*
||FK sloupec||Reference / poznámka||
|rpo_id|→ {{rpo(id)}}|
|skupina_odpadu_id|→ {{garbage_group(id)}}|

*Indexy (návrh):*
||Index||Definice / poznámka||
|IX_nadoba_rpo|({{rpo_id}})|
|IX_nadoba_skupina_odpadu|({{skupina_odpadu_id}})|
|UX_nadoba_rfid|({{rfid}}) [pokud je RFID unikátní]|

----

h2. 10. Stanoviště

*Stav:* Existující entita (beze změn v rámci tohoto návrhu CS) \\
*Role:* Fyzické umístění nádob a lokalizace objednaných služeb.

h3. Klíčové atributy

* {{id}} (PK)
* {{adresa}}
* {{gps_lat}}
* {{gps_lon}}

h3. Vazby

* {{Stanoviště (1) → Nadoba_Stanoviste (N)}} (časově platné přiřazení nádob)

h3. Pravidla a poznámky

* V ER v1 *není přímá vazba Stanoviště → RPO*.
* Vazba {{Stanoviště ↔ Nádoba}} je vedena přes existující vazební entitu {{Nadoba_Stanoviste}}.
* Vazba na RPO je nepřímá přes {{Nádobu}}.

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* Stanoviště je klíčová lokalizační entita pro pasportizaci nádob a plánování/realizaci CS.
* PP a FOB používají stanoviště pro mapové zobrazení a sledování obslouženosti.
* Při zobrazení RPO na mapě se preferuje lokalita stanovišť navázaných nádob; místo realizace RPO je fallback.
* Stanoviště vstupují do generování lokací OS a do plánování tras v RP/FOB.

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{stanoviste}} (existující; bez nové FK na RPO v tomto návrhu)

*Mapování atributů na DDL dle {{inventar-entit-PP.md}} (entita: Stanoviště / {{site}}):*
||Atribut (ER)||DDL sloupec v inventáři PP||Stav||Poznámka||
|{{id}}|{{id}}|nalezeno|přímá shoda|
|{{adresa}}|{{address_id}}|částečně|ER atribut je zjednodušený pohled; v inventáři je reference na entitu Adresa|
|{{gps_lat}}|{{lat}}|nalezeno|přímá shoda významu|
|{{gps_lon}}|{{lng}}|nalezeno|přímá shoda významu|

*Sloupce (návrh / delta):*
||Položka||Popis||
|stávající sloupce entity {{stanoviste}}|beze změn pro CS v této kapitole|
|bez {{rpo_id}} (přímá vazba na RPO se v ER nevyskytuje)|—|

*FK (návrh):*
||FK sloupec||Reference / poznámka||
|bez přímé FK na {{rpo}}|—|

*Indexy (návrh):*
||Index||Definice / poznámka||
|IX_stanoviste_gps|({{gps_lat}}, {{gps_lon}})|
|IX_stanoviste_adresa|({{adresa}}) [fulltext / prefix dle potřeby]|

----

h2. 11. Typ nádoby

*Stav:* Existující — rozšíření pro CS \\
*Role:* Číselník typů nádob; zdroj parametrů pro plánování a výpočet obsluhy.

h3. Klíčové atributy

* {{id}} (PK)
* {{nazev}}
* {{objem_litry}}
* {{cas_obsluhy_sec}} (nový / editovatelný pro CS)

h3. Vazby

* {{RPO (N) → Typ nádoby (1)}}

h3. Pravidla a poznámky

* ER v1 uvádí zdroj ERP HEN/WINX (read-only) s editovatelným {{cas_obsluhy_sec}}.
* V aktuálním ER je vazba na {{Typ nádoby}} vedena z {{RPO}} (nikoli přímo z {{Nádoby}}).

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* Typy nádob jsou existující číselník (HEN/WINX) rozšířený o businessově významný atribut času obsluhy.
* {{cas_obsluhy_sec}} slouží pro výpočet plánované doby realizace okruhu dne v RP.
* V dalších etapách se očekává rozšíření typů nádob o parametry pro kapacitní výpočty (váha/objem) pro strategickou optimalizaci.

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{typ_nadoby}} (rozšíření existující)

*Mapování atributů na DDL dle {{inventar-entit-PP.md}} (entita: Typ nádoby / {{container_type}}):*
||Atribut (ER)||DDL sloupec v inventáři PP||Stav||Poznámka||
|{{id}}|{{id}}|nalezeno|přímá shoda|
|{{nazev}}|{{name}}|nalezeno|přímá shoda významu|
|{{objem_litry}}|—|nenalezeno|v inventáři DDL {{container_type}} není přímý sloupec objemu|
|{{cas_obsluhy_sec}}|—|nenalezeno|CS rozšíření v ER; inventář {{container_type}} ho neuvádí|

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

*Stav:* Nová entita pro CS \\
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

*Mapování atributů na DDL dle {{inventar-entit-PP.md}}:*
||Atribut (ER)||DDL sloupec v inventáři PP||Stav||Poznámka||
|{{id}}, {{rpo_id}}, {{okruh_id}}, {{rozvrh_id}}, {{platnost_od}}, {{platnost_do}}|—|nenalezeno|nová CS entita; v inventáři PP není explicitně popsána|

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

h2. 13. Nadoba_Stanoviste

*Stav:* Existující vazební entita (využitá v návrhu CS) \\
*Role:* Časově platné přiřazení {{Nádoba ↔ Stanoviště}}; zdroj lokalizace nádoby a nepřímě i lokalizace RPO/OS.

h3. Klíčové atributy

* {{id}} (PK)
* {{nadoba_id}} (FK → {{Nádoba}})
* {{stanoviste_id}} (FK → {{Stanoviště}})
* {{platnost_od}}
* {{platnost_do}} ({{NULL = aktivní}})

h3. Vazby

* {{Nádoba (1) → Nadoba_Stanoviste (N)}}
* {{Stanoviště (1) → Nadoba_Stanoviste (N)}}

h3. Pravidla a poznámky

* V aktuálním ER v1 nahrazuje tato vazební entita přímý atribut {{Nádoba.stanoviste_id}}.
* Vazba je modelována jako časově platná (historie přistavení / stažení nádoby).
* Vazba {{RPO → Stanoviště}} zůstává nepřímá: {{RPO -> Nádoba -> Nadoba_Stanoviste -> Stanoviště}}.
* Je vhodné potvrdit business pravidlo, zda smí mít jedna nádoba více souběžně aktivních přiřazení ke stanovišti (typicky se očekává max. 1 aktivní přiřazení).

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* Stanoviště nádob jsou primárním zdrojem lokalizace pro plánování a realizaci CS.
* Při generování lokací OS se využívají stanoviště navázaných nádob; pokud chybí, používá se fallback přes místo realizace RPO.
* Časová platnost vazby nádoba–stanoviště je důležitá pro správné vyhodnocení aktuální lokace v plánovacím období.

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{site_container_assignment}} (existující DDL; v ER alias {{nadoba_stanoviste}})

*Mapování atributů na DDL dle {{inventar-entit-PP.md}} (entita: Přiřazení nádoby ke stanovišti / {{site_container_assignment}}):*
||Atribut (ER)||DDL sloupec v inventáři PP||Stav||Poznámka||
|{{id}}|{{id}}|nalezeno|přímá shoda|
|{{nadoba_id}}|{{container_id}}|nalezeno|přímá shoda významu|
|{{stanoviste_id}}|{{site_id}}|nalezeno|přímá shoda významu|
|{{platnost_od}}|{{valid_from}}|nalezeno|přímá shoda významu|
|{{platnost_do}}|{{valid_to}}|nalezeno|přímá shoda významu|

*Sloupce (návrh / delta):*
||Položka||Popis||
|stávající sloupce entity {{site_container_assignment}}|beze změn pro CS v tomto návrhu|
|container_id ({{nadoba_id}})|existující FK na {{container}}|
|site_id ({{stanoviste_id}})|existující FK na {{site}}|
|valid_from ({{platnost_od}})|začátek platnosti přiřazení|
|valid_to ({{platnost_do}})|konec platnosti přiřazení|

*FK (návrh):*
||FK sloupec||Reference / poznámka||
|nadoba_id / {{container_id}}|→ {{nadoba(id)}} / {{container(id)}}|
|stanoviste_id / {{site_id}}|→ {{stanoviste(id)}} / {{site(id)}}|

*Indexy (návrh):*
||Index||Definice / poznámka||
|IX_ns_nadoba|({{nadoba_id}}) / ({{container_id}})|
|IX_ns_stanoviste|({{stanoviste_id}}) / ({{site_id}})|
|IX_ns_platnost|({{platnost_od}}, {{platnost_do}}) / ({{valid_from}}, {{valid_to}})|
|IX_ns_nadoba_historie|({{nadoba_id}}, {{platnost_od}} desc)|
|UX_ns_nadoba_aktivni|filtrovaný index na {{nadoba_id}} kde {{platnost_do is null}} [jen pokud business potvrdí max 1 aktivní vazbu]|

----

h2. 14. Skupina_Druh_odpadu

*Stav:* Nová entita pro CS \\
*Role:* Temporální mapování {{Druh odpadu ↔ Skupina odpadu}} pro odvození plánovací skupiny odpadu v čase.

h3. Klíčové atributy

* {{id}} (PK)
* {{skupina_odpadu_id}} (FK → {{Skupina odpadu}})
* {{druh_odpadu_id}} (FK → {{Druh odpadu}})
* {{platnost_od}}
* {{platnost_do}} ({{NULL = aktivní}})

h3. Vazby

* {{Skupina odpadu (1) → Skupina_Druh_odpadu (N)}}
* {{Druh odpadu (1) → Skupina_Druh_odpadu (N)}}

h3. Pravidla a poznámky

* V aktuálním ER v1 je mapování {{Druh odpadu -> Skupina odpadu}} vedeno nepřímo přes tuto vazební entitu (nikoli přímým FK v {{Druh odpadu}}).
* Entita zavádí časovou platnost mapování, aby bylo možné dohledat historické změny klasifikace pro plánování a audit.
* Výsledek mapování slouží jako vstup pro vyplnění {{RPO.skupina_odpadu_id}}.
* Je nutné potvrdit business pravidlo, zda v jednom čase smí mít {{Druh odpadu}} více aktivních mapování na různé skupiny (zejména vzhledem ke skupině {{MIX}} a automatickým pravidlům).

h3. Businessové požadavky (z {{vytah-cilovy-koncept.md}})

* PP je místem správy mapování druhů odpadů na skupiny odpadů pro potřeby plánování CS.
* Mapování slouží ke zjednodušení výběru RPO a k automatickému doplňování skupiny odpadu při synchronizaci / přípravě dat.
* Pravidla pro skupinu {{MIX}} vyžadují kontrolovanou správu mapování a omezení běžné editace.
* Správnost mapování přímo ovlivňuje seskupování OS do okruhů dne v RP (1 okruh dne = 1 skupina odpadu).

h3. Návrh fyzického DDL (SQL)

*Tabulka:* {{skupina_druh_odpadu}}

*Mapování atributů na DDL dle {{inventar-entit-PP.md}}:*
||Atribut (ER)||DDL sloupec v inventáři PP||Stav||Poznámka||
|{{id}}, {{skupina_odpadu_id}}, {{druh_odpadu_id}}, {{platnost_od}}, {{platnost_do}}|—|nenalezeno|nová CS vazební entita; v inventáři PP není popsána|

*Sloupce (návrh):*
||Položka||Popis||
|id bigint identity primary key|—|
|skupina_odpadu_id bigint not null|—|
|druh_odpadu_id bigint not null|—|
|platnost_od datetime2(0) not null|—|
|platnost_do datetime2(0) null|—|
|audit/tenant sloupce dle standardu PP|—|

*FK (návrh):*
||FK sloupec||Reference / poznámka||
|skupina_odpadu_id|→ {{garbage_group(id)}}|
|druh_odpadu_id|→ {{garbage_type(id)}}|

*Indexy (návrh):*
||Index||Definice / poznámka||
|IX_sdo_skupina|({{skupina_odpadu_id}})|
|IX_sdo_druh|({{druh_odpadu_id}})|
|IX_sdo_platnost|({{platnost_od}}, {{platnost_do}})|
|IX_sdo_druh_historie|({{druh_odpadu_id}}, {{platnost_od}} desc)|
|UX_sdo_druh_aktivni|filtrovaný index na {{druh_odpadu_id}} kde {{platnost_do is null}} [pokud business potvrdí max 1 aktivní mapování na druh]|
|UX_sdo_kombinace_obdobi|({{druh_odpadu_id}}, {{skupina_odpadu_id}}, {{platnost_od}}) [min. ochrana proti duplicitnímu vložení]|

----

h2. Doporučení pro další rozpracování DS/DDL

* U každé kapitoly doplnit fyzický DDL návrh (typy sloupců, indexy, FK, {{NULL/NOT NULL}}).
* U temporální tabulky {{RPO_Okruh_Rozvrh}} doplnit constraint / pravidlo pro „max 1 aktivní vazba na RPO“.
* U {{Nádoba.skupina_odpadu_id}} popsat prioritu vyhodnocení vůči {{RPO.skupina_odpadu_id}} v aplikační logice.
* U {{Adresy}} doplnit význam atributů {{X}}, {{Y}} (souřadnice / lokální souřadný systém / jiný význam).

h3. Poznámky k návrhu řešení

*Kolize / nesoulady vůči {{inventar-entit-PP}}:*

* Vazba {{RPO -> Nádoba (1:N)}} a atribut {{Nádoba.rpo_id}} jsou v kolizi s inventářem PP. V inventáři je vazba vedena přes existující vazební entitu *Přiřazení nádoby k položce objednávky* ({{container_order_item_assignment}}), nikoliv přímým FK v {{Nádoba}}.
* Entita {{Skupina odpadu}} je v návrhu rozšířena o {{provozovna_id}} a popsána jako evidovaná per provozovna. Inventář PP pro {{garbage_group}} tuto vazbu neuvádí (entita je popsána jako standardní číselník se systémovými atributy a vazbou na barvu skupiny odpadu).
* Umístění nových entit {{Okruh}}, {{Rozvrh}}, {{Kalendář}}, {{Zóna}} přímo do modelu PP je v napětí s inventářem PP. Inventář je uvádí jako „nové pro CS“, ale zároveň poznamenává, že pravděpodobně budou vznikat v RoadPlanu, nikoliv v PasPortu.

*Další problémy / nejasnosti v návrhu:*

* Popis {{RPO_Okruh_Rozvrh}} stále obsahuje pravidlo „max 1 aktivní vazba na RPO“, což je potřeba explicitně potvrdit nebo upravit podle finální business logiky (zejména pokud se připustí více souběžně aktivních vazeb).
* {{Stanoviště}} je nyní správně značeno jako existující entita, ale textový popis je formulován jako „Rozšířená existující entita“; to je vhodné sjednotit.
* {{Skupina_Druh_odpadu}} je validní CS rozšíření návrhu, ale není zatím promítnuta do {{inventar-entit-PP}} (inventář u {{Druh odpadu}} aktuálně uvádí bez referenčních asociací). Je potřeba rozhodnout, zda půjde o nové PP rozšíření, nebo jen aplikační mapování mimo DS/DDL PP.
* {{RPO_Okruh_Rozvrh}} není zatím explicitně zapsána v inventáři PP mezi novými entitami pro CS; doporučeno doplnit inventář nebo uvést, že jde pouze o konceptuální / integrační vrstvu mimo PP DS.

h3. Poznámky k prověření k datovému modelu (viz Findings — kolize / problémy)

* Prověřit a rozhodnout kolizi {{RPO -> Nádoba (1:N)}} + {{Nádoba.rpo_id}} vs. existující PP vazba přes {{container_order_item_assignment}} (inventář PP vede vazbu přes vazební entitu, ne přímým FK v {{Nádoba}}).
* Prověřit, zda {{Skupina odpadu.provozovna_id}} je skutečně cílové CS rozšíření PP, nebo zda má zůstat {{garbage_group}} globální/tenantový číselník bez přímé vazby na provozovnu.
* Prověřit architektonické umístění nových entit {{Okruh}}, {{Rozvrh}}, {{Kalendář}}, {{Zóna}} (PP vs. RP) a sjednotit to napříč ER diagramem, {{datovy-model-PP.md}} a inventářem PP.
* Prověřit a explicitně potvrdit pravidlo pro maximální počet aktivních vazeb v {{RPO_Okruh_Rozvrh}} (aktuálně model předpokládá max. 1 aktivní vazbu na {{RPO}}).
* Dopsat / sjednotit dokumentaci existujících entit a vazebních entit:
** {{Stanoviště}} (textový popis vs. status existující entity),
** {{Skupina_Druh_odpadu}} (doplnění do inventáře PP nebo označení jako mimo DS/DDL PP),
** {{RPO_Okruh_Rozvrh}} (doplnění do inventáře PP nebo označení jako konceptuální vrstva).

h3. Poznámky k HELIOS (viz Připomínky — kolize / rizika z {{vytah-helios-nephrite-zvoz.md}} vůči {{datovy-model-PP.md}})

* Prověřit zásadní rozpor HEN vs. PP modelu u okruhů:
** HEN připouští, že jeden PZ může být ve více okruzích,
** PP model {{RPO_Okruh_Rozvrh}} aktuálně předpokládá max. 1 aktivní vazbu na {{RPO}}.
* Prověřit, zda kombinovaná entita {{RPO_Okruh_Rozvrh}} odpovídá HEN integrační realitě:
** HEN komunikuje vazby PZ↔Rozvrh, PZ↔Okruh, PZ↔Zóna separátně,
** může být potřeba rozdělit model nebo přesně popsat transformační logiku v integraci.
* Doplnit do kapitol {{RPO}}, {{Rozvrh}}, {{RPO_Okruh_Rozvrh}} explicitní integrační pravidlo z HEN:
** Rozvrh je na PZ *dynamický vztah*, ne prostý atribut na PZ.
* Doplnit do kapitol {{Zóna}} a {{Stanoviště}} logiku odvození zóny:
** zóna se v HEN na PZ doplňuje podle *adresy stanoviska*,
** v modelu PP je potřeba zachytit původ/derivaci {{RPO.zona_id}} a validační podmínky (aktivní zóna, shoda útvaru).
* Rozšířit kapitoly {{Rozvrh}} a {{Kalendar}} o detailnější model HEN rozvrhu (nebo explicitně popsat, že PP přebírá zjednodušený/flattened přenos):
** HEN rozvrh má detailní parametrizaci a položky rozvrhu (dny vývozu).
* Rozšířit kapitolu {{Okruh}} o HEN odvozené atributy (nebo explicitně označit jako odvozené/nepersistované):
** celkový počet nádob,
** celkový objem nádob,
** dny zvozu,
** typ týdne.
* Doplnit do kapitoly {{Typ nádoby}} poznámku k rozlišení:
** typ nádoby jako kontraktační údaj na PZ/RPO,
** vs. fyzická nádoba a její evidence/pohyby.
* Prověřit dopad dvojí historizace:
** HEN verzuje PZ při materiálních změnách,
** PP navrhuje temporální vazby ({{RPO_Okruh_Rozvrh}}, {{Nadoba_Stanoviste}}, {{Skupina_Druh_odpadu}}),
** je nutné přesně vymezit odpovědnost jednotlivých vrstev historizace.

h4. Dotazy pro zákazníka

* Má být v cílovém řešení zachována možnost, aby *jeden PZ / RPO byl současně ve více okruzích a rozvrzích*? A pokud ano, tak dochází ke změně okruhu a rozvrhu najednou?
* Má PP v Etapě 1 přebírat z HEN pouze výsledný stav vazeb pro {{PZ↔Rozvrh}}, {{PZ↔Okruh}}, {{PZ↔Zóna}}? Co bude spuštěčem přenosu do PP?
* Má být {{RPO.zona_id}} v PP považováno za:
** odvozenou/synchronizovanou hodnotu z adresy stanoviska,
** nebo za samostatně editovatelný údaj?
* Existují nějaké validační podmínky pro doplnění zóny z HEN jsou pro zákazníka závazné i v PP (aktivní zóna, shoda útvaru, kontrola zóny)?
* Pro plánování v RP počítáme s tím, že RP bude pracovat pouze se seznamem dní vývozu. Okruh, Rozvrh a Zóna budou pouze infromativní u OS. Je to v pořádku?
* Jak má být řešena historizace řazení RPO do Okruhu, Rozvrhu a Zóny:
** stačí historizace v HEN (verze PZ),
** nebo je požadována i samostatná historizace vazeb v PP?


h4. Dotazy pro dodavatele informačního systému (HELIOS Nephrite)

* Jak přesně je v HEN reprezentován *dynamický vztah PZ↔Rozvrh*
* Jaké identifikátory a atributy jsou dostupné pro vazbu {{PZ↔Rozvrh↔Kalendář}}
* Jak je v HEN reprezentována vazba {{PZ↔Okruh}}:
* Jaký je přesný datový model vazby {{PZ↔Zóna}}:
* Potvrďte, zda zóna na PZ vzniká vždy automaticky z *adresy stanoviska*, a jaké jsou přesné podmínky / validační pravidla (aktivní zóna, útvar, kontrola zóny).
* Jak v HEN vypadá datový model *Rozvrhu vývozu* a jeho položek (dní vývozu):
** seznam tabulek/entit,
** klíčové atributy,
** vazba na PZ,
** způsob rozlišení C/N cyklického svozu.
* Jak HEN verzuje PZ při materiálních změnách (okruh, rozvrh, typ nádoby apod.):
** vzniká vždy nová verze,
** jak se značí návaznost verzí,
** jak je dostupná historie přes API/export?

