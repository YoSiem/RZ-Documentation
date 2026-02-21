# KONCENTRAT - Dokumentacja Techniczna Skill Database

> **Wersja**: 1.0 (Skonsolidowana)
> **Cel**: Single Source of Truth dla mechaniki umiejętności w silniku Rappelz.
> **Status**: Wersja kanoniczna. Zastępuje rozproszone pliki `01`–`14` oraz instrukcje `AGENTS.md`.

---

## Spis Treści

1.  [Wstęp i Słownik Pojęć](#1-wstęp-i-słownik-pojęć)
2.  [Architektura Bazy Danych (Schema Reference)](#2-architektura-bazy-danych-schema-reference)
    *   2.1 [Identyfikacja i Flagi Podstawowe](#21-identyfikacja-i-flagi-podstawowe)
    *   2.2 [Koszty i Wymagania](#22-koszty-i-wymagania)
    *   2.3 [Casting i Timing](#23-casting-i-timing)
    *   2.4 [Logika Celowania i Frakcje](#24-logika-celowania-i-frakcje)
    *   2.5 [Stany i Wzmocnienia](#25-stany-i-wzmocnienia)
    *   2.6 [Modyfikatory Walki i Zmienne](#26-modyfikatory-walki-i-zmienne)
3.  [Referencje i Stałe](#3-referencje-i-stałe)
    *   3.1 [Bitmaski Standardowe (Statystyki)](#31-bitmaski-standardowe-statystyki)
    *   3.2 [Bitmaski Rozszerzone (Elementy/Penetracja)](#32-bitmaski-rozszerzone-elementypenetracja)
    *   3.3 [System Żywiołów](#33-system-żywiołów)
4.  [Biblioteka Typów Efektów (Effect Types)](#4-biblioteka-typów-efektów-effect-types)
    *   4.1 [Mechanika Pasywna (EF_PARAMETER_AMP - 4)](#41-mechanika-pasywna-ef_parameter_amp---4)
    *   4.2 [Mechanika Magii Ofensywnej](#42-mechanika-magii-ofensywnej) (231, 232, 235, 261, 262, 263)
5.  [Przewodnik Dewelopera i AI](#5-przewodnik-dewelopera-i-ai)
    *   5.1 [Zasady Generowania SQL](#51-zasady-generowania-sql)
    *   5.2 [Pełny Szablon INSERT](#52-pełny-szablon-insert)
    *   5.3 [Procedura Analizy Kodu Źródłowego](#53-procedura-analizy-kodu-źródłowego)

---

## 1. Wstęp i Słownik Pojęć

Niniejszy dokument stanowi kompletny przewodnik po strukturze bazy danych umiejętności (`SkillResource`), mechanice efektów oraz zasadach tworzenia nowych wpisów. Został on stworzony w celu wyeliminowania duplikatów i sprzeczności w dokumentacji projektowej.

### Słownik Pojęć (Glossary)

*   **Skill (Umiejętność)**: Pojedynczy rekord w tabeli `SkillResource` identyfikowany unikalnym kluczem `id`.
*   **Effect Type (Typ Efektu)**: Kluczowa kolumna (`effect_type`), która determinuje, jaka funkcja C++ obsłuży logikę umiejętności (np. `4` dla pasywów, `231` dla magii).
*   **State (Stan)**: Buff lub Debuff nakładany na cel, zdefiniowany w osobnej tabeli `StateBase` i referencjonowany przez `state_id`.
*   **Bitmask (Maska Bitowa)**: Liczba całkowita będąca sumą potęg dwójki (flag), pozwalająca na zakodowanie wielu opcji w jednym polu (np. `P.Atk` + `M.Atk` w jednej zmiennej).
*   **GCD (Global Cooldown)**: Opóźnienie globalne (`delay_common`), blokujące użycie innych umiejętności po rzuceniu czaru.
*   **Var (Zmienna)**: Pola od `var1` do `var20`. Ich znaczenie jest płynne i zależy w 100% od wybranego `effect_type`.
*   **SLV (Skill Level)**: Poziom umiejętności. Wiele parametrów skaluje się "per SLV" (np. `cost_mp_per_skl`).
*   **Enhance (Wzmocnienie)**: Poziom ulepszenia karty umiejętności (widoczny w UI jako `+1`, `+5` itd.). Wpływa na parametry `_per_enhance`.
*   **Tick**: Jednostka czasu serwera, w której odświeżane są stany (zazwyczaj co sekundę lub część sekundy w zależności od konfiguracji).

---

## 2. Architektura Bazy Danych (Schema Reference)

Poniższa sekcja stanowi kompletny opis wszystkich 94 kolumn tabeli `SkillResource`. Kolumny zostały pogrupowane tematycznie.

### 2.1 Identyfikacja i Flagi Podstawowe

*   **`id`** (INT): Unikalny identyfikator umiejętności (np. `90006`). Klucz główny.
*   **`text_id`** (INT): ID z tabeli StringTable dla pełnego opisu umiejętności. Zazwyczaj `50000000 + id`.
*   **`desc_id`** (INT): ID dla drugorzędnego opisu (często `0`).
*   **`tooltip_id`** (INT): ID z tabeli StringTable dla nazwy umiejętności (widocznej w dymku). Zazwyczaj `40000000 + id`.
*   **`is_valid`** (0/1): Flaga aktywności. `1` = Umiejętność aktywna w silniku. `0` = Wyłączona/Debug.
*   **`elemental`** (0-6): Przypisuje żywioł do umiejętności (zob. [3.3 System Żywiołów](#33-system-żywiołów)). `0` = Fizyczny/Brak.
*   **`is_passive`** (0/1): Określa tryb działania.
    *   `0` = Pasywna (ciągły efekt, nie wymaga rzucania).
    *   `1` = Aktywna (wymaga użycia/rzucenia).
*   **`is_physical_act`** (0/1): Typ ataku. `1` = Fizyczny (korzysta z P.Atk, animacji broni). `0` = Magiczny (korzysta z M.Atk).
*   **`is_harmful`** (0/1): Charakter efektu. `1` = Szkodliwy (atak, debuff). `0` = Pozytywny (leczenie, buff).
*   **`is_toggle`** (0/1): Czy umiejętność jest przełączalną aurą (traci MP w czasie).
*   **`toggle_group`** (INT): Grupa aur. Włączenie aury z tej samej grupy wyłącza poprzednią.
*   **`effect_type`** (INT): **Kluczowy parametr logiczny**. Określa funkcję C++ obsługującą działanie skilla (np. `4`=Pasyw, `231`=Magia).
*   **`icon_id`** (INT) & **`icon_file_name`** (STRING): Odwołania do zasobów graficznych klienta (ikona w UI).

### 2.2 Koszty i Wymagania

#### Zasoby (Cost)
Wartości odejmowane przy użyciu umiejętności.
*   **`cost_hp`**, **`cost_mp`**: Stała wartość HP/MP.
*   **`cost_hp_per_skl`**, **`cost_mp_per_skl`**: Wzrost kosztu na poziom umiejętności.
*   **`cost_mp_per_enhance`**: Zmiana kosztu MP zależna od poziomu karty (Alt+Y). Często ujemna (zmniejsza koszt).
*   **`cost_hp_per`**, **`cost_mp_per`**: Koszt procentowy Max HP/MP (używane w aurach). Odpowiedniki `_per_skl_per` skalują procent.
*   **`cost_havoc`**, **`cost_energy`**: Specjalne zasoby klasowe (Asura/Gaia).
*   **`cost_exp`**, **`cost_jp`**: Koszt punktów doświadczenia lub Job Points (rzadkie).
*   **`cost_item`** (ID), **`cost_item_count`**: Wymagany przedmiot w ekwipunku i jego ilość.

#### Wymagania (Need)
Warunki konieczne do aktywacji.
*   **`need_level`**: Minimalny poziom postaci.
*   **`need_hp`**, **`need_mp`**: Minimalna ilość posiadanych zasobów.
*   **`need_state_id`**: Wymagany aktywny buff/debuff na celu lub rzucającym.
*   **`need_state_level`**: Minimalny poziom tego stanu.
*   **`need_state_exhaust`** (0/1): Czy użycie umiejętności usuwa (konsumuje) wymagany stan?

#### Flagi Broni (Weapon Flags - `vf_*`)
Ograniczają użycie umiejętności do konkretnego typu ekwipunku. Wartość `1` oznacza wymóg.

| Kolumna | Typ Broni | Kolumna | Typ Broni |
| :--- | :--- | :--- | :--- |
| `vf_dagger` | Sztylet | `vf_double_dagger` | Podwójne sztylety |
| `vf_one_hand_sword` | Miecz 1H | `vf_two_hand_sword` | Miecz 2H |
| `vf_double_sword` | Podwójne miecze | `vf_spear` | Włócznia |
| `vf_axe` | Topór 2H | `vf_one_hand_axe` | Topór 1H |
| `vf_double_axe` | Podwójne topory | `vf_one_hand_mace` | Buława 1H |
| `vf_two_hand_mace` | Buława 2H | `vf_one_hand_staff` | Różdżka 1H |
| `vf_two_hand_staff` | Kostur 2H | `vf_shield_only` | Tarcza |
| `vf_lightbow` | Łuk lekki | `vf_heavybow` | Łuk długi |
| `vf_crossbow` | Kusza | **`vf_is_not_need_weapon`** | **Ignoruje broń** |

> **Ważne**: Dla umiejętności pasywnych lub magicznych niezależnych od broni, zawsze ustawiaj `vf_is_not_need_weapon = '1'`.

### 2.3 Casting i Timing

Wszystkie czasy podawane są w sekundach (float).

*   **`delay_cast`**: Czas inkantacji (pasek postępu).
*   **`delay_cast_per_skl`**: Skrócenie/wydłużenie czasu rzucania na poziom.
*   **`delay_cooltime`**: Czas odnowienia (Cooldown).
*   **`delay_cooltime_per_skl`**: Skrócenie/wydłużenie Cooldownu na poziom.
*   **`delay_common`**: Global Cooldown (GCD). Standardowo `1.0`–`1.5`.
*   **`casting_type`** / **`casting_level`**: Odporność na przerwanie czaru przy otrzymaniu obrażeń (`0`=Nieprzerywalny).
*   **`is_projectile`** (0/1): Czy generuje pocisk. Jeśli `1`, obrażenia wchodzą przy trafieniu (on impact).
*   **`projectile_speed`** / **`projectile_acceleration`**: Prędkość i przyspieszenie pocisku.

### 2.4 Logika Celowania i Frakcje

#### Cel (Target)
*   **`is_need_target`** (0/1): Czy wymagane jest zaznaczenie celu.
*   **`target`** (INT): Typ logiczny celu.
    *   `0` = Dowolny.
    *   `1` = Zaznaczony cel (Single).
    *   `2` = Obszar wokół celu (AoE Region).
    *   `3` = Obszar na ziemi (Ground Target).
    *   `4` = Obszar wokół siebie (PBAoE).
    *   `41` = Specjalny tryb dla pasywów (obejmuje self/summon).
*   **`cast_range`**: Maksymalny zasięg rzucania (inicjacji).
*   **`valid_range`**: Maksymalny zasięg efektu (np. promień AoE dla `target=2/3`).

#### Frakcje (Faction Flags - `uf_*`)
Określają, kogo można trafić ( = dozwolone).
*   **`uf_self`**: Siebie.
*   **`uf_party`**: Członków drużyny.
*   **`uf_guild`**: Członków gildii.
*   **`uf_neutral`**: Neutralnych graczy.
*   **`uf_enemy`**: Wrogów (potwory, wrogowie PvP).
*   **`uf_purple`**: Graczy w trybie PK.

#### Typ Bytu (Entity Flags - `tf_*`)
*   **`tf_avatar`**: Gracze.
*   **`tf_summon`**: Chowańce/Summony.
*   **`tf_monster`**: Potwory NPC.

### 2.5 Stany i Wzmocnienia

Obsługa Buffów i Debuffów nakładanych przez umiejętność.
*   **`state_id`** (INT): ID stanu z tabeli `StateBase`.
*   **`state_level_base`**: Poziom nakładanego stanu.
*   **`state_level_per_skl`**: Wzrost poziomu stanu wraz z poziomem skilla.
*   **`state_second`**: Czas trwania stanu (w sekundach/tickach).
*   **`state_second_per_level`**: Wydłużenie czasu trwania na poziom.
*   **`skill_enchant_link_id`**: ID powiązania z systemem kart (Alt+Y). Jeśli `0`, skill nie ma kart.

### 2.6 Modyfikatory Walki i Zmienne

*   **`probability_on_hit`** (%): Szansa na nałożenie stanu (`state_id`).
*   **`hit_bonus`**: Dodatkowa celność (płaska wartość).
*   **`critical_bonus`**: Dodatkowa szansa na krytyk (płaska wartość).
*   **`hate_mod`**: Modyfikator generowanego aggro (`1.0` = 100%, `1.5` = 150%).
*   **`var1`** do **`var20`** (FLOAT): **Główne zmienne operacyjne**.
    *   Ich znaczenie zależy wyłącznie od pola `effect_type`.
    *   Mogą przechowywać: mnożniki obrażeń, maski bitowe, liczniki uderzeń, promienie wybuchu.
    *   Szczegóły w sekcji [4. Biblioteka Typów Efektów](#4-biblioteka-typów-efektów-effect-types).


## 3. Referencje i Stałe

Wartości liczbowe używane w kolumnach `var` oraz flagach bitowych.

### 3.1 Bitmaski Standardowe (Statystyki)

Używane w `EF_PARAMETER_AMP` (sloty standardowe: var1, var4, var13, var16).
Aby połączyć efekty, **dodaj** wartości do siebie (np. P.Atk + M.Atk = 128 + 256 = 384).

| Nazwa Statystyki | Wartość (DEC) | Bitset (C++) |
| :--- | :--- | :--- |
| **STR** (Strength) | `1` | 1 << 0 |
| **VIT** (Vitality) | `2` | 1 << 1 |
| **AGI** (Agility) | `4` | 1 << 2 |
| **DEX** (Dexterity) | `8` | 1 << 3 |
| **INT** (Intelligence) | `16` | 1 << 4 |
| **MEN** (Mentality) | `32` | 1 << 5 |
| **LUK** (Luck) | `64` | 1 << 6 |
| **P.Atk** (Attack Point) | `128` | 1 << 7 |
| **M.Atk** (Magic Point) | `256` | 1 << 8 |
| **P.Def** (Defence) | `512` | 1 << 9 |
| **M.Def** (Magic Defence) | `1024` | 1 << 10 |
| **Attack Speed** | `2048` | 1 << 11 |
| **Cast Speed** | `4096` | 1 << 12 |
| **Move Speed** | `8192` | 1 << 13 |
| **Accuracy** (Physical) | `16384` | 1 << 14 |
| **Magic Accuracy** | `32768` | 1 << 15 |
| **Critical Rate** | `65536` | 1 << 16 |
| **Block Rate** | `131072` | 1 << 17 |
| **Block Defence** | `262144` | 1 << 18 |
| **Avoid** (Dodge) | `524288` | 1 << 19 |
| **M.Res** (Magic Resist) | `1048576` | 1 << 20 |
| **Max HP** | `2097152` | 1 << 21 |
| **Max MP** | `4194304` | 1 << 22 |
| **HP Regen** (Flat) | `16777216` | 1 << 24 |
| **MP Regen** (Flat) | `33554432` | 1 << 25 |
| **HP Regen %** | `134217728` | 1 << 27 |
| **MP Regen %** | `268435456` | 1 << 28 |
| **Final DMG Inc %** | `536870912` | 1 << 29 |
| **Max Weight** | `1073741824` | 1 << 30 |
| **Final DMG Red %** | `2147483648` | 1 << 31 |

### 3.2 Bitmaski Rozszerzone (Elementy/Penetracja)

Używane **wyłącznie** w slotach rozszerzonych `EF_PARAMETER_AMP` (zazwyczaj **var7** i **var10**).
Kodują odporności na żywioły, penetrację pancerza i specjalne typy obrażeń.

| Nazwa Statystyki | Wartość (DEC) | Uwagi |
| :--- | :--- | :--- |
| **None Resist** | `1` | Odporność na bezżywiołowe |
| **Fire Resist** | `2` | Odporność na Ogień |
| **Water Resist** | `4` | Odporność na Wodę |
| **Wind Resist** | `8` | Odporność na Wiatr |
| **Earth Resist** | `16` | Odporność na Ziemię |
| **Light Resist** | `32` | Odporność na Światło |
| **Dark Resist** | `64` | Odporność na Mrok |
| **Attack Range** | `256` | Zasięg ataku |
| **Perfect Block** | `512` | Blok absolutny |
| **Ignore P.Def** | `1024` | Ignorowanie P.Def (przebicie) |
| **Ignore M.Def** | `2048` | Ignorowanie M.Def (przebicie) |
| **P.Penetration** | `4096` | Penetracja fizyczna |
| **M.Penetration** | `8192` | Penetracja magiczna |
| **Critical Damage** | `268435456` | Siła krytyka (Crit Power) |
| **HP Regen Stop** | `536870912` | Blokada regeneracji HP |

> **Uwaga**: Wartości takie jak `Ignore P.Def` używają mnożnika (var9/12) jako procentu (0.1 = 10%).

### 3.3 System Żywiołów

Wartości wpisywane do kolumny `elemental`.

| ID | Żywioł | Kod C++ | Interakcje (Zależności) |
| :--- | :--- | :--- | :--- |
| `0` | **None** | `TYPE_NONE` | Brak. Obrażenia fizyczne/neutralne. |
| `1` | **Fire** | `TYPE_FIRE` | Silny na Wiatr, Słaby na Wodę. |
| `2` | **Water** | `TYPE_WATER` | Silny na Ogień, Słaby na Ziemię. |
| `3` | **Wind** | `TYPE_WIND` | Silny na Ziemię, Słaby na Ogień. |
| `4` | **Earth** | `TYPE_EARTH` | Silny na Wodę, Słaby na Wiatr. |
| `5` | **Light** | `TYPE_LIGHT` | Przeciwstawny do Dark. |
| `6` | **Dark** | `TYPE_DARK` | Przeciwstawny do Light. |


## 4. Biblioteka Typów Efektów (Effect Types)

Sekcja ta definiuje logikę interpretacji zmiennych `var1`–`var20` dla kluczowych typów umiejętności.

### 4.1 Mechanika Pasywna (EF_PARAMETER_AMP - 4)

Używana do tworzenia pasywnych wzmocnień statystyk. Wymaga `is_passive=0` (w silniku 0 oznacza pasyw).
Zmienne są zorganizowane w **trójki (sloty)**. Każdy slot definiuje:
1.  **Maskę statystyki** (którą cechę modyfikujemy).
2.  **Wartość bazową** (dodawaną na stałe).
3.  **Mnożnik poziomu** (dodawany za każdy poziom skilla).

#### Schemat Slotów

| Slot | Zmienne (Maska / Baza / Per_Level) | Typ Maski |
| :--- | :--- | :--- |
| **1** | `var1`, `var2`, `var3` | [Standardowa](#31-bitmaski-standardowe-statystyki) |
| **2** | `var4`, `var5`, `var6` | [Standardowa](#31-bitmaski-standardowe-statystyki) |
| **3** | `var7`, `var8`, `var9` | **[Rozszerzona](#32-bitmaski-rozszerzone-elementypenetracja)** |
| **4** | `var10`, `var11`, `var12` | **[Rozszerzona](#32-bitmaski-rozszerzone-elementypenetracja)** |
| **5** | `var13`, `var14`, `var15` | [Standardowa](#31-bitmaski-standardowe-statystyki) |
| **6** | `var16`, `var17`, `var18` | [Standardowa](#31-bitmaski-standardowe-statystyki) |

> **Wzór**: `FinalValue = Base + (Per_Level * SkillLevel)`
> Dla mnożników procentowych (np. `0.05`) oznacza to 5%.

### 4.2 Mechanika Magii Ofensywnej

Rodzina efektów zadających obrażenia magiczne (`is_physical_act=0`).
Wspólne ID: **231, 232, 235, 261, 262, 263**.

#### Kanoniczna Formuła Obrażeń (Shared Damage Formula)

Wszystkie poniższe typy używają identycznego zestawu zmiennych `var1`–`var6` do obliczenia obrażeń pojedynczego trafienia:

```
Obrażenia = M.Atk * (var1 + var2*SLV + var3*Enhance)
          + (var4 + var5*SLV + var6*Enhance)
```

*   **Grupa Mnożnika (%)**: `var1` (baza), `var2` (per lvl), `var3` (per karta). `1.0` = 100% M.Atk.
*   **Grupa Płaska (Flat)**: `var4` (baza), `var5` (per lvl), `var6` (per karta). Dodawane po mnożeniu.

#### Warianty i Różnice

Poniższa tabela wskazuje, jak poszczególne typy rozszerzają formułę podstawową (zmienne `var7+`) oraz jak należy ustawić kolumny celowania.

| ID | Nazwa Typu | Opis | Target | Is Need Target | Specjalne Zmienne (var7+) | Uwagi |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **231** | Single Magic | Standardowy pocisk/hit w 1 cel. | `1` | `1` | Brak. | Obsługuje `state_id` (On-Hit). |
| **232** | Multi Magic | Seria uderzeń w 1 cel. | `1` | `1` | `var7`: Ilość trafień (bazowa)<br>`var8`: Ilość trafień (per lvl)<br>`var9`: Opóźnienie (sec) | Każdy hit może nałożyć stan. |
| **235** | Absorb Magic | Kradzież życia/many (Vampiric). | `1` | `1` | `var7`: HP Absorb %<br>`var10`: MP Absorb % | **Nie nakłada Stanów** (ignoruje state_id). |
| **261** | Single Region | AoE wokół wybranego celu. | `2` | `1` | `valid_range`: Promień AoE | Uderza raz wszystkich w zasięgu. |
| **262** | Special Region | AoE wskazane na ziemi (Meteor). | `3` | `0` | `valid_range`: Promień AoE | Nie wymaga celu (Ground Target). |
| **263** | Multi Region | Seria AoE wokół celu (Blizzard). | `2` | `1` | `var7-9`: Jak w typie 232.<br>`valid_range`: Promień AoE | Seria uderzeń w obszarze. |

> **Ostrzeżenie**: Zmienna `var7` ma zupełnie inne znaczenie w typie 232 (ilość hitów) niż w typie 235 (HP Absorb %). Sprawdź ID przed edycją!


## 5. Przewodnik Dewelopera i AI

Zasady tworzenia nowych wpisów w bazie danych oraz dokumentowania zmian.

### 5.1 Zasady Generowania SQL

Podczas tworzenia nowych umiejętności należy przestrzegać następujących reguł, aby uniknąć kolizji z oficjalnymi danymi serwera:

1.  **Zakres ID**: Używaj ID powyżej **91000**. Zwiększaj o 1 dla każdego nowego skilla.
2.  **Identyfikatory Tekstowe**:
    *   `text_id` = `50000000 + ID` (Długi opis).
    *   `tooltip_id` = `40000000 + ID` (Nazwa skilla).
    *   `desc_id` = `0`.
3.  **Ikony**: Używaj istniejących ID (np. `1100`, `1530`) oraz nazw plików (np. `'icon_skill_pas_mage_moral_culture'`).
4.  **Format Pliku**: Wynikowe zapytania SQL umieszczaj w bloku kodu `sql`.

### 5.2 Pełny Szablon INSERT

Każde zapytanie **musi zawierać wszystkie 94 kolumny** w dokładnie tej kolejności. Nie usuwaj kolumn, których wartości są zerowe.

```sql
INSERT INTO [dbo].[SkillResource] (
    [id], [text_id], [desc_id], [tooltip_id], [is_valid], [elemental],
    [is_passive], [is_physical_act], [is_harmful], [is_need_target],
    [is_corpse], [is_toggle], [toggle_group], [casting_type],
    [casting_level], [cast_range], [valid_range], [cost_hp],
    [cost_hp_per_skl], [cost_mp], [cost_mp_per_skl], [cost_mp_per_enhance],
    [cost_hp_per], [cost_hp_per_skl_per], [cost_mp_per], [cost_mp_per_skl_per],
    [cost_havoc], [cost_havoc_per_skl], [cost_energy], [cost_energy_per_skl],
    [cost_exp], [cost_exp_per_enhance], [cost_jp], [cost_jp_per_enhance],
    [cost_item], [cost_item_count], [cost_item_count_per_skl], [need_level],
    [need_hp], [need_mp], [need_havoc], [need_havoc_burst], [need_state_id],
    [need_state_level], [need_state_exhaust], [vf_one_hand_sword],
    [vf_two_hand_sword], [vf_double_sword], [vf_dagger], [vf_double_dagger],
    [vf_spear], [vf_axe], [vf_one_hand_axe], [vf_double_axe],
    [vf_one_hand_mace], [vf_two_hand_mace], [vf_lightbow], [vf_heavybow],
    [vf_crossbow], [vf_one_hand_staff], [vf_two_hand_staff], [vf_shield_only],
    [vf_is_not_need_weapon], [delay_cast], [delay_cast_per_skl],
    [delay_cast_mode_per_enhance], [delay_common], [delay_cooltime],
    [delay_cooltime_per_skl], [delay_cooltime_mode_per_enhance],
    [cool_time_group_id], [uf_self], [uf_party], [uf_guild], [uf_neutral],
    [uf_purple], [uf_enemy], [tf_avatar], [tf_summon], [tf_monster],
    [skill_lvup_limit], [target], [effect_type], [skill_enchant_link_id],
    [state_id], [state_level_base], [state_level_per_skl], [state_level_per_enhance],
    [state_second], [state_second_per_level], [state_second_per_enhance],
    [probability_on_hit], [probability_inc_by_slv], [hit_bonus],
    [hit_bonus_per_enhance], [percentage], [hate_mod], [hate_basic],
    [hate_per_skl], [hate_per_enhance], [critical_bonus], [critical_bonus_per_skl],
    [var1], [var2], [var3], [var4], [var5], [var6], [var7], [var8], [var9], [var10],
    [var11], [var12], [var13], [var14], [var15], [var16], [var17], [var18], [var19],
    [var20], [icon_id], [icon_file_name], [is_projectile], [projectile_speed],
    [projectile_acceleration]
) VALUES ( ... );
```

### 5.3 Procedura Analizy Kodu Źródłowego

Gdy napotkasz nowy, nieudokumentowany `effect_type`:

1.  **Lokalizacja Źródeł**: Szukaj w katalogu:
    `D:\Rappelz_source\program\server\Rappelz-Game-Server`
2.  **Szukanie Handlera**: Użyj `grep` aby znaleźć funkcję obsługującą ID efektu (np. `EF_INSTANT_DAMAGE`).
    `grep -r "EF_INSTANT_DAMAGE" .`
3.  **Analiza Zmiennych**: Sprawdź wywołania `GetVar(0)` do `GetVar(19)`.
    *   Pamiętaj: C++ indeksuje od 0 (`GetVar(0)` to SQL `var1`).
4.  **Weryfikacja**: Sprawdź czy są nałożone limity (clamp, min/max) oraz jak typowane są zmienne (float vs int vs bitmask).
5.  **Dokumentacja**: Dopisz nowy typ do sekcji [4. Biblioteka Typów Efektów](#4-biblioteka-typów-efektów-effect-types). Nie wklejaj kodu C++, opisz logikę.
