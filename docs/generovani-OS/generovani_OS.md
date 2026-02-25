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
