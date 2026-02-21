# KONCENTRAT DOKUMENTACJI RAPPELZ
*Sekcja: 1 z X (Wersja WIP)*

## SPIS TREŚCI
1. [I. Wstęp i Słownik Pojęć](#i-wstęp-i-słownik-pojęć)
2. [II. Architektura Tabeli SkillResource (94 kolumny)](#ii-architektura-tabeli-skillresource-94-kolumny)
3. [III. Tabele Wartości Kanonicznych (Referencje)](#iii-tabele-wartości-kanonicznych-referencje)
4. [IV. Typy Efektów (effect_type) - Single Source of Truth](TBD)
5. [V. Szablony Implementacji SQL](TBD)
6. [VI. AI Meta-Wytyczne](TBD)

---

## I. Wstęp i Słownik Pojęć

Single source of truth dla procesu tworzenia skilli oraz ich parametryzacji w instancjach serwerowych Rappelz. Redukuje redundantne zasady do jednego wariantu kanonicznego.

**Słownik Podstawowy:**
* **`SkillResource`**: Główna tabela RDB/SQL składająca się zawsze z dokładnie 94 kolumn, definiująca zachowanie skilli.
* **`effect_type`**: Identyfikator silnika (np. 4, 32021) decydujący o wewnętrznej procedurze C++ obsługującej dany skill. Opcje: aktywny hit, DoT, aura, buff pasywny, etc.
* **SLV**: Skill Level (Poziom umiejętności). Istnieje wiele mnożników dodających modyfikatory per SLV (`_per_skl`).
* **Enhance**: System zaklinania kart (Level karty). Modyfikatory skalujące po formacie `_per_enhance`.
* **State (`StateBase`)**: System nakładania stałych buffów i debuffów (stany). Kolumny `state_...` powiązane są z nakładanym stanem o ID z tabeli `StateBase`.
* **Bitmask / Maski Bitowe**: Zmienne silnika łączone poprzez sumowanie (np. P.Atk to 128, M.Atk to 256. Suma 384 oznacza przypisanie do obydwu, w oparciu o przesunięcia bitowe).

---

## II. Architektura Tabeli SkillResource (94 kolumny)

Poniżej zwięzły opis wszystkich 94 kolumn, podzielony na grupy logiczne (zastępuje pliki 01, 06-11). **Podczas tworzenia nowego zapytania SQL INSERT należy bezwzględnie użyć wszystkich 94 kolumn w określonej kolejności.**

### 1. Identyfikatory i Podstawowe Flagi
Kolumny identyfikujące instancję w bazie, jej powiązania językowe i typ (pasywny/aktywny).
* `id` - Unikalne ID skilla. > `91000`.
* `text_id` - ID nazwy (zawsze `50000000 + id`).
* `tooltip_id` - ID opisu rozszerzonego (zawsze `40000000 + id`).
* `desc_id` - Nieużywane (domyślnie `0`).
* `icon_id` & `icon_file_name` - Grafika UI skilla (np. id: `1530`, plik: `'icon_skill_pas_mage_moral_culture'`).
* `is_valid` - Włączenie skilla. Zawsze `1`.
* `is_passive` - Negacja stanu aktywnego: `'0'` = skill pasywny, `'1'` = skill aktywny!
* `is_physical_act` - `'1'` = fizyczny hit (używa Accuracy), `'0'` = cast (magia). Dotyczy tylko wariantu oddania ataku, sam `effect_type` liczy obrażenia po swojemu.
* `is_harmful` - `'1'` = skill jest wrogi / ofensywny, `'0'` = neutralny lub buforujący. Definiuje też czy buff (0) można kliknąć u sojusznika.
* `is_toggle` & `toggle_group` - Skille odpalane na "trzymanie" (Aura). `'1'` znaczy toggle aktywne.
* `skill_enchant_link_id` - Wiązanie z bonusem karty. Domyślnie `0`.
* `skill_lvup_limit` - Limit maksymalnego poziomu rozwoju per level postaci. Domyślnie `0` (brak/silnik def).
* `elemental` - Kanoniczny żywioł przypisany w `GameType.h`. Kod numeryczny (Patrz Sekcja III: Żywioły). Skille pasywne mają `'0'`.

### 2. Typ Castowania, Czasy i Zasięgi
Definicje rzutowania geometrii umiejętności i klatek czasowych. Kolumny czasu zapisywane jako `Float` reprezentują sekundy.
* `casting_type` - Szansa przerwania castu wroga. `'0'` = nieprzerwalny, `'1'` = można przerwać.
* `casting_level` - Zwykle równe `casting_type`.
* `target` - Kto jest docelowym celem wg. definicji silnika. Zbiór z góry narzucony: m.in. `1` (klikniety target), `2` (AoE Wokół targetu), `3` (AoE ground), `4` (czyste AoE bez ground clicku), `41` (Targetowanie pętli buffa pasywnego).
* `is_need_target` - `'1'` jeśli użytkownik MUSI mięć aktywny cel (nawet na siebie), `'0'` = triggeruje sam (np AoE self-centered, self-buff).
* `is_corpse` - `'1'` dotyczy wyłącznia zwłok (np. resurrect).
* `valid_range` - Promień efektu wybuchu wokół celu bazowego (zasięg radialny np. dla Target = `2`, `3`). Mierzone w metrach gry.
* `cast_range` - Max odległość postaci od targetu inicjującego. Krótsze dla melee (np. 20) vs dystans (np. 200).
* `delay_cast` & `delay_cast_per_skl` & `delay_cast_mode_per_enhance` - Czas channelingu / paska Cast. Oparta na float np `1.50` sekund.
* `delay_cooltime` & `delay_cooltime_per_skl` & `delay_cooltime_mode_per_enhance` - Typowy "Cooldown" recastu skilla.
* `cool_time_group_id` - Współdzielony cooldown skilli (global cooldown dla konkretnej rodzimy).
* `delay_common` - Częstotliwość tików (np. 1.50 dla pasywek).
* `is_projectile` & `projectile_speed` & `projectile_acceleration` - Fizyczny lot pocisku, po którym uderza skill (animacja hitu zależy od drogi). `is_projectile=1` włącza logikę.

### 3. Koszty, Wymagania i Frakcje (`uf_` & `tf_`)
Determinuje jakie zasoby konsumuje skill oraz kogo silnik waliduje przed jego przepuszczeniem.
* KOSZTY HP/MP: `cost_hp`, `cost_mp` (baza płaska), `cost_..._per_skl` (plus per slv), `cost_..._per_enhance` (z karty). ORAZ Wartości Procentowe np. `cost_hp_per` (zuzywa % puli zamiast flat).
* KOSZTY INNE: `cost_havoc`, `cost_energy`, `cost_exp`, `cost_jp`, `cost_item`, `cost_item_count`.
* WYMAGANIA MIN: `need_level`, `need_hp`, `need_mp`, `need_havoc`, `need_havoc_burst`. (Użyć, ale nie zżerać ich).
* OGRANICZENIE BRONI (`vf_` flags): np. `vf_one_hand_sword`, `vf_dagger`, `vf_lightbow`, itd.. Zawsze ustawiane `'1'` (wymaga) lub `'0'`. Domyślnie dla skilla bez wymogu jest to `vf_is_not_need_weapon='1'`.
* FRAKCJE (`uf_` flags): Do kogo mozna to strzelić. `uf_self` (ja), `uf_party` (party), `uf_guild`, `uf_neutral`, `uf_purple` (PK), `uf_enemy` (Wrogowie). Czyste `1` i `0`.
* TYPY ENTITY (`tf_` flags): `tf_avatar` (Gracz), `tf_summon` (Pety), `tf_monster` (Mobki/Bossy).

### 4. Modyfikatory Walki i Zmienne (`varX`)
Złożona matematyka wewnątrz funkcji klas aplikacyjnych.
* `effect_type` - ID Procedury z silnika określające logikę, która zużyje resztę kolumn `varX`.
* `hate_basic` & `hate_mod` & `_per_skl` & `_per_enhance` - Agresja wysyłana do otoczenia. Bazowy hate i modyfikator np. `1.20` generujący dodatkowo 20% threat nad uderzeniem standardowym. (Heale często mają to duże, aby moby biły lecznika).
* `hit_bonus` & `hit_bonus_per_enhance` - Płaski bypass uniku.
* `critical_bonus` & `critical_bonus_per_skl` - Płaski wzrost Crit. Rate przy tym uderzeniu.
* ZMIENNE UNIWESALNE (Var1 - 20) : Tablica w której umieszczane są numery decydujące o ostatecznym skalowaniu wzorów matematcznych w kodzie C++. Cała logika skalowania (Multiplier%, Base Flat, itp) wewnątrz zależy CZYSTO od zapisanego `effect_type`. Do użycia tylko zgodnie z Referencją Typów Efektów.

### 5. Stany Aplikacyjne (`StateBase`)
Buff jako obiekt wtórny wyzwalany tym hitem.
* `state_id` - ID efektu z tabeli State (np. debuff trucizny lub buff tarczy).
* `probability_on_hit` & `probability_inc_by_slv` - Szansa rólki rng np 0–100%. `100` znaczy 100%.
* `state_level_base` & `_per_skl` & `_per_enhance` - Moc owego Buffa (Tier). W silniku zazwyczaj Tier_buffa = base + (SLV*per_skl).
* `state_second` & `_per_level` & `_per_enhance` - Czas trwania Buffa na postaci, wpisywane jako float w sekundach np `5.0000`.
* `percentage` - Nieużywany mnożnik ukryty dla starych systemów RDB bazującego enhance (TBD z używalnością ogólną).
* WARUNKI WBUDOWANE: `need_state_id`, `need_state_level`, `need_state_exhaust` - Pozwala zdefiniować konieczność bycia "Po wpływem X" by dało się odpalić skill (np Combo Attack). Exhaust=`1` ściąga dany Stan.

---

## III. Tabele Wartości Kanonicznych (Referencje)

Tabele te mapowane są na kod numeryczny w varach i kolumnach i współdzielonych pomiędzy typy.

### 1. Elementy Gry (Żywioły)
Trafia do `elemental`. Dotyczy odporności u wrogów i modyfikatorów Atk.
| ID | Nazwa Żywiołu (C++) | Kod | Notatka & Użycie Najczęstsze |
|:---|:---|:---|:---|
| 0  | NONE             | '0' | Ataki fizyczne, pasywki, skille bezżywiołowe |
| 1  | EARTH            | '1' | Obezwładnianie, stany zakorzenienia, trucizny |
| 2  | FIRE             | '2' | Burst damage, podpalenia, DoT |
| 3  | WATER            | '3' | Spowalnianie, leczenie, zamarzanie |
| 4  | WIND             | '4' | Szybkie hity, AoE casty wiatrowe |
| 5  | LIGHT             | '5' | Wskrzeszanie, zaawansowane heale, buffy |
| 6  | DARK             | '6' | Klątwy, kradzież HP, masowe osłabienia debuffowe |

### 2. Standardowe Maski Bitowe (`Standard Bitmasks`)
Stosowane głównie dla: `EF_PARAMETER_AMP` (Sloty: `var1`, `var4`, `var13`, `var16`), a także `EF_INC_PARAM_BASED_PARAM` (Typ 32301 dla `var1`, `var2` jako źródło). Pozwalają na sumowanie wartości! 128 (P.Atk) + 256 (M.Atk) = 384 zaaplikuje oba staty.

| Statystyka | Wartość Maski Bitowej | Statystyka | Wartość Maski Bitowej |
|:---|:---|:---|:---|
| STR | 1 | Accuracy | 16384 |
| VIT | 2 | M.Accuracy | 32768 |
| AGI | 4 | Critical Rate | 65536 |
| DEX | 8 | Block Rate | 131072 |
| INT | 16 | Block Defense | 262144 |
| MEN | 32 | Avoid/Dodge | 524288 |
| LUK | 64 | M.Resistance | 1048576 |
| P.Atk | 128 | Max HP | 2097152 |
| M.Atk | 256 | Max MP | 4194304 |
| P.Def | 512 | HP Regen Add | 16777216 |
| M.Def | 1024 | MP Regen Add | 33554432 |
| Atk Speed | 2048 | Max Weight | 1073741824 |
| Cast Speed | 4096 | Final Dmg Red. | 2147483648 |
| Move Speed | 8192 | - | - |

### 3. Rozszerzone Maski Bitowe (`Extended Bitmasks`)
Wszystko co obsluguje drugi mechanizm tarczy lub parametry penetracji i odległości z prefiksem `FLAG_ET_`.
Stosowane tylko dla Specyficznych slotów (w `EF_PARAMETER_AMP` jest to slot `var7` oraz `var10`). Nie mogą być używane we wszystkich `var`.
(Zasada sumowania dotyczy tu oddzielnego intsetu!)

| Statystyka (Buffowana/Edytowana) | Wartość |
|:---|:---|
| None Resist / Attr.less Res | 1 |
| Fire Resist | 2 |
| Water Resist | 4 |
| Wind Resist | 8 |
| Earth Resist | 16 |
| Light Resist | 32 |
| Dark Resist | 64 |
| Attack Range | 256 |
| Perfect Block | 512 |
| P.Def Ignore | 1024 |
| M.Def Ignore | 2048 |
| P. Penetration | 4096 |
| M. Penetration | 8192 |
| Critical Damage | 268435456 |

*(Koniec fragmentu cz. 1 Koncentratu)*
