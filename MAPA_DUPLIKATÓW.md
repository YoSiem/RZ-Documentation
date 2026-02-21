# MAPA DUPLIKATÓW
*Mapowanie redukcji nadmiarowych instrukcji w procesie tworzenia KONCENTRAT.md.*

| ŹRÓDŁO (plik / sekcja) | NOWE MIEJSCE W KONCENTRAT.md | RODZAJ OPERACJI | UZASADNIENIE |
|:---|:---|:---|:---|
| `00_Table_of_Contents.md` | Odrzucone / Zastąpione | Usunięto | Spis treści tworzony jest od nowa, by odzwierciedlał zwięzłą hierarchię. |
| `01_Skill_Database_Architecture.md` | I. Wstęp i Słownik, II. Architektura Tabeli | Scalone | Główny szkielet dla opisu 94 kolumn połączony ze szczegółami plików 06-11. |
| `02_Standard_Bitmasks.md` | III. Tabele Wartości Kanonicznych - Pkt 2 | Przeniesione w całości | Przekształcone w jedną kanoniczną tabelę, do której odwołują się skille. Usunięto redundantny wstęp. |
| `03_Extended_Bitmasks.md` | III. Tabele WartościKanonicznych - Pkt 3 | Przeniesione w całości | Tabela mask z rozszerzonymi wariantami (penetracje). Opisy przeniesione pod jedną flagę. |
| `05_Elementals_Guide.md` | III. Tabele Wartości Kanonicznych - Pkt 1 | Przeniesione i Skrócone | Tabelka z numerami kodów połączona z krótkimi wzmiankami "Kiedy jakiego żywiołu użyć", by nie wodolejować. |
| `06_Skill_Identifiers_and_Core_Flags.md` | II. Architektura Tabeli - Pkt 1 | Skrót + Odsyłacz | Zredukowano do bullet pointsów, unikając wielostronicowych "Ten kolumn robi...". |
| `07_Skill_Casting_and_Timing.md` | II. Architektura Tabeli - Pkt 2 | Skrót + Odsyłacz | Zredukowano do bullet pointsów. |
| `08_Skill_Costs_and_Requirements.md` | II. Architektura Tabeli - Pkt 3 | Skrót + Odsyłacz | Skondensowano opisy zasobów i wymogów na postać / ekwipunek do minimalnej pigułki o limitach. |
| `09_Skill_Target_and_Usage.md` | II. Architektura Tabeli - Pkt 2/3 | Scalone i skrócone | Skonkretyzowano listę dopuszczalnych ID dla frakcji docelowej (`uf`/`tf`), wyrzucono opis poezji i przeniesiono czyste enumy. |
| `10_Skill_States_and_Enhancements.md` | II. Architektura Tabeli - Pkt 5 | Skrót + Odsyłacz | Logika Buffów (`StateBase`) upchnięta logicznie do jednej małej sekcji bez opowieści. |
| `11_Skill_Combat_Modifiers_and_Variables.md` | II. Architektura Tabeli - Pkt 4 | Skrót + Złączenie | Pozbyto się opisów "Dlaczego omija def targetu", zostawiono matematykę bazową + var1-20 odstawiono jako przypis "Tylko z Effect Type". |
| `AGENTS.md` (Sekcje Bitmask / Architecture) | - | Usunięte jako duplikat | AGENTS.md masowo kopiowało Tabele z 02, 03. Teraza będzie to wyłącznie referencja do "III. Tabele Wartości...". |
