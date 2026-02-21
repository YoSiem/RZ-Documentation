# KONFLIKTY I TBD (To Be Done)

Lista niejasności, sprzeczności oraz brakujących elementów wykrytych podczas konsolidacji dokumentacji.

## Rozstrzygnięte Konflikty

1.  **Szablon SQL (INSERT)**
    *   **Konflikt**: Różne pliki (`04`, `12`, `AGENTS.md`) zawierały drobne różnice w formatowaniu przykładowych zapytań SQL (np. komentarze, formatowanie float).
    *   **Rozstrzygnięcie**: Przyjęto wersję z `AGENTS.md` jako kanoniczną (wszystkie 94 kolumny jawnie wymienione), ponieważ jest najbardziej rygorystyczna i bezpieczna dla AI.

2.  **Znaczenie zmiennej `var7` w efektach magicznych**
    *   **Konflikt**: W typie 232 `var7` to "Liczba uderzeń", w typie 235 to "HP Absorb Ratio".
    *   **Rozstrzygnięcie**: Nie jest to błąd, lecz cecha silnika. W sekcji **4.2 Magia Ofensywna** dodano wyraźne ostrzeżenie o różnicach w interpretacji zmiennych zależnie od ID efektu.

3.  **Wartości Bitmaski `Final DMG Red %`**
    *   **Konflikt**: W niektórych starszych notatkach (poza dostarczonymi plikami, ale w wiedzy ogólnej) bit 31 bywa różnie interpretowany (jako znak lub flaga).
    *   **Rozstrzygnięcie**: Przyjęto definicję z pliku `02_Standard_Bitmasks.md` (`2147483648` = 1 << 31) jako obowiązującą.

## TBD (Braki i Do Uzupełnienia)

1.  **Brak dokumentacji dla pozostałych `effect_type`**
    *   **Opis**: Dokumentacja pokrywa szczegółowo tylko typy: `4` (Pasyw) oraz rodzinę `231/232/235/261/262/263` (Magia). Silnik Rappelz posiada setki innych typów (np. `EF_PHYSICAL_DAMAGE`, `EF_HEAL`, `EF_SUMMON`).
    *   **Status**: TBD. Należy sukcesywnie dopisywać nowe typy do sekcji **4. Biblioteka Typów Efektów** zgodnie z procedurą z sekcji 5.3.

2.  **Szczegóły formuły `hate_mod`**
    *   **Opis**: Dokumentacja podaje, że `hate_mod` to mnożnik. Nie jest jednak jasne, czy `hate_mod=0` całkowicie wyłącza aggro, czy tylko mnożnik obrażeń (pozostawiając `hate_basic`).
    *   **Rekomendacja**: Wymaga testów w grze lub głębszej analizy kodu `StructCreature::AddHate`.

3.  **Interakcja `toggle_group` z `cool_time_group_id`**
    *   **Opis**: Nie jest do końca jasne, czy aury z tej samej grupy (`toggle_group`) dzielą również cooldowny, czy jest to mechanizm czysto logiczny (wyłączanie jednej przy włączaniu drugiej).
    *   **Rekomendacja**: TBD.
