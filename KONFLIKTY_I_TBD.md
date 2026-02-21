# KONFLIKTY, NIEJASNOŚCI I TBD (To-Be-Determined)
*Raport sprzeczności i braków w oryginalnej dokumentacji z propozycjami rozstrzygnięć.*

### 1. Przynależność `percentage` na poły pomięta jako "Enhanced", ale bez detali matematycznych
* **Miejsce**: W pliku `10_Skill_States_and_Enhancements.md` wzmiankowana wokół procentowych skal z poziomu zaklinania karty, a w `11_Skill_Combat_Modifiers...` wrzucone w dół ignorując detale. W pliku 22 jako `0`.
* **Rodzaj konfliktu**: Niejasność i brak precyzji działania silnika dla starszych RDB co do owej kolumny `percentage`.
* **Rozstrzygnięcie / Sugestia**: Uznaje się z domyślną bezpieczną wartość `0` przy 99% generowania Skilli i wpisuje tę kolumnę do Sekcji Stanów i Zmieniaczy, jako przestarzałą pozostałość z informacją by jej unikać (safe 0).

### 2. Status parametru `var19` oraz `var20`
* **Miejsce**: Całkowity brak opisu w `11_Skill_Combat_Modifiers_and_Variables.md` (urywa się na var18 dla bitmask slotu 6, i mówi "var19-20 są puste).
* **Rodzaj konfliktu**: Brak wyjaśnienia, dlaczego 94 kolumny muszą mieć `var19` i `var20`, jeśli silnik nawet w C++ tego nie parsuje według `04_Creating_...`.
* **Rozstrzygnięcie**: Traktujemy jako martwy padding strukturalny klienta dla struktury sieciowej/RDB, oznaczam w architekturze TBD oraz ostrzegam by podczas SQL wrzucać stale `0.0000`.

### 3. Nakładająca się rola `probability_on_hit` oraz `probability_inc_by_slv` 
* **Miejsce**: Fragmentarycznie rozbite między pliki 10 i 11.
* **Rodzaj konfliktu**: Skąd mamy wiedzieć kiedy 100 on_hit jest uznawane za gwarancję pętli silnika bez testu na "Miss" z Accuracy/M.Accuracy?
* **Rozstrzygnięcie**: Scaliłem i wrzuciłem logikę Prawdopodobieństw do Sekcji o "State Applications / Buffach". Ustanowiłem, że te zmienne służą WYŁĄCZNIE nakładaniu kolumny `state_id`, a nie testowi na sam Hit skilla względem dmg. Test na miss leci po Accuracy Target vs Blok z defaultu pomijąjąc obydwie wspomniane wartości.

*(To doc listowane będzie na bieżąco iteracyjnie przy dodawaniu Formuł Math dla Effect Types)*
