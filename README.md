# Dokumentacja Hat'a: Sterownik silnika krokowego TMC5130A z przetwornicą Buck

## Sekcja 1: Dokumentacja Hat'a

### Krótki opis hat'a
Projekt to sterownik silnika krokowego, oparty na dedykowanym układzie scalonym **TMC5130A-TA** sterowanym po SPI. Urządzenie pozwala na sterowanie silnikami o prądzie cewki do 2A (maksymalnie 2.5A w piku). Dodadkowo na PCB jest przetwornica buck oraz złącza i przełączniki do wybierania zasilania, między VIN, 6.2V, 9V, 12V, 24V.

### Zgodność ze standardem ChainBus

* ✅  Używa złącza ChainBus, nie zmienia jego miejsca ani pinoutu.
* ✅  Używa wyłącznie interfejsów I2C, SPI lub UART i nie inicjuje samodzielnie nowych transmisji (Nie jest master'em I2C albo SPI).
* ✅  Spełnia wymagania mechaniczne standardu (wymiary PCB, rozstaw otworów).
* ✅  Pobiera maksymalny prąd zgodny z ilością na jednego hat'a
* ✅  Obsługuje napięcie wejściowe BRD_VIN do wartości 48V.
### Komunikacja i adresowanie

#### Magistrala SPI

| Układ (IC)   | Funkcja                     | Połączenie CS              |
| :----------- | :-------------------------- | :------------------------- |
| **TMC5130A** | Sterownik silnika krokowego | Bezpośrednio do Chainbus'a |


---

### Pinout złączy

#### J6 — Złącze enkodera
Złącze przeznaczone do podłączenia zewnętrznego enkodera zwrotnego.

| Pin   | Sygnał           | Opis |
| :---- | :--------------- | :--- |
| **1** | `ENCN_DCO`       |      |
| **2** | `ENCB_DCEN_CFG4` |      |
| **3** | `ENCA_DCIN_CFG5` |      |

#### J7 — Złącze silnika krokowego
Złącze wyjściowe do podłączenia faz silnika dwufazowego.

| Pin   | Sygnał | Opis               |
| :---- | :----- | :----------------- |
| **1** | `B1`   | Faza B (wyjście 1) |
| **2** | `B2`   | Faza B (wyjście 2) |
| **3** | `A2`   | Faza A (wyjście 2) |
| **4** | `A1`   | Faza A (wyjście 1) |

---

### Konfiguracja układu (Zworki/Przełączniki)

#### J8 — Wybór źródła zasilania stopnia mocy silnika
Zworka J8 służy do określenia, skąd sterownik TMC5130A pobiera napięcie zasilania cewek:

| Pozycja zworki  | Wybrane źródło zasilania | Opis                                                                                                                                                                        |
| :-------------- | :----------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Pozycja 1-2** | Przetwornica Buck        | Zasilanie silnika z wewnętrznej przetwornicy obniżającej napięcie (napięcie regulowane przełącznikiem SW1).                                                                 |
| **Pozycja 2-3** | Złącze XT60 (VIN)        | Bezpośrednie zasilanie ze złącza XT60. Tor ten posiada trzy szeregowe diody (D5, D6, D7) w celu bezpiecznego dopasowania do maksymalnego zakresu napięć standardu ChainBus. |

#### SW1 — Wybór napięcia wyjściowego przetwornicy Buck
Przełącznik pozwala na wybór napięcia zasilania silnika, jeśli zworka J8 jest ustawiona na zasilanie z przetwornicy.

| Aktywny kanał SW1 | Wyjściowe napięcie | Uwagi                                        |
| :---------------- | :----------------- | :------------------------------------------- |
| **Pozycja 1**     | 24 V               | Domyślne napięcie dla standardowych silników |
| **Pozycja 2**     | 12 V               | Dla silników średniego napięcia              |
| **Pozycja 3**     | 9 V                | Napięcie zredukowane                         |
| **Pozycja 4**     | 6.2 V              | Minimalne stabilne napięcie pracy układu     |

* **⚠️ WAŻNE:** Zmianę położenia przełącznika SW1 należy wykonywać wyłącznie przy całkowicie odłączonym zasilaniu modułu. W danym momencie może być włączony tylko jeden przełącznik wyboru napięcia.
* **Modyfikacja poziomów napięć:** Istnieje możliwość zmiany napięc wyjściowych przetwornicy przez zmiane **R14**, **R15** lub **R16**.

#### RV1 — Trymer regulacyjny prądu referencyjnego (AIN_REF)
Potencjometr montażowy RV1 służy do analogowego ustawienia napięcia odniesienia na pinie `AIN_REF` układu TMC5130A, co pozwala na ręczne ograniczenie maksymalnego prądu płynącego przez cewki silnika krokowego.

---

### Szczegółowy opis techniczny
Hat sterownika silnika krokowego używawający TMC5130A-TA. I = 2A

TMC5130A jest sterowana po SPI co umożliwia pełną, programową kontrolę nad wszystkimi parametrami ruchu. Umożliwia to precyzyjną regulację prędkości i pozycji, automatyczne generowanie ramp przyspieszeń i hamowań (co odciąża główny mikrokontroler od obliczeń w czasie rzeczywistym), a także bieżący odczyt diagnostyczny stanu pracy stopnia mocy oraz bezczujnikowe wykrywanie przeciążeń i zablokowania osi.

Opóźniania spowodowane użyciem SPI zamiast standardowego driver'a stepper'a (STEP DIR) poniżej 100us (przesłanie jedner ramki ~= 10us) oraz powoduje mniejsze zajęcie mikrokontrolera (rząda tylko efektu, nie generuje impulsów STEP/DIR)

Przetwornica buck została zaprojektowana tak, aby konwertować napięcie wejściowe na wybrane niższe napięcia robocze silnika. Zaleca się stosowanie zewnętrznego zasilania doprowadzonego do złącza XT60.

### Gotowe arkusze hierarchiczne
W strukturze projektu wykorzystano następujące arkusze hierarchiczne:
* **Buck Regulator** – Układ impulsowej przetwornicy obniżającej napięcie LM2596, wyposażony w przełącznik sprzężenia zwrotnego (SW1) umożliwiający zmianę napięcia wyjściowego (6.2V, 9V, 12V, 24V).
* **Stepper controller** – Schemat  sterownika silnika krokowego TMC5130A-T

---

## Sekcja 2: Specyfikacja standardu ChainBus

### Architektura i łączenie modułów
Standard ChainBus umożliwia modułowe łączenie hatów. Na jednym MMS3 można zamontować pionowo **do 8 hat'ów**. Połączenie realizowane jest poprzez wpięcie złącza męskiego kolejnego hat'a w złącze żeńskie poprzedniego.

### Komunikacja i sterowanie
Magistrala ChainBus jest w pełni cyfrowa. Płyta główna nie steruje bezpośrednio sygnałami ogólnego przeznaczenia (GPIO) na poszczególnych hat'ach. Wszelkie operacje muszą być realizowane przez dedykowane układy scalone komunikujące się przez interfejsy systemowe.

Wybór aktywnego modułu realizowany jest przez układ przełącznika magistrali (bus switch) na płycie głównej. Dzięki temu linie I2C, SPI i UART są niezależne dla każdego hat'a (brak konfliktów adresów I2C oraz kolizji na liniach SPI CS).
* **Identyfikacja:** Każdy moduł powinien posiadać pamięć EEPROM na magistrali I2C w celu identyfikacji płyty przez system - układ M24C64-W skonfigurowany na adres `1010000` przy liniach adresowych A0, A1, A2 zwartych do masy.

### Zasilanie
Złącze ChainBus dostarcza następujące linie zasilania:

| Magistrala zasilania | Napięcie znamionowe | Maksymalny prąd (łączny dla 8 hatów) | Szacowany prąd na jeden hat |
| :------------------- | :-----------------: | :----------------------------------: | :-------------------------: |
| **5V**               |        5.0 V        |                1.0 A                 |           125 mA            |
| **12V stby**         |       12.0 V        |                0.5 A                 |            65 mA            |
| **BRD_VIN**          |   12.0 V – 48.0 V   |                1.5 A                 |           185 mA            |

*   Komponenty podłączone do linii `BRD_VIN` muszą być przystosowane do pracy z napięciem od 12V do **48 V**.
*   W przypadku dużego zapotrzebowania na prąd (np. jednoczesna praca kilku silników o wysokim momencie obrotowym), dopuszczalne jest skorzystanie z dedykowanego wysokoprądowego złącza XT60.

### Wymagania mechaniczne i złącza
* **Wymiary PCB:** Niedozwolona jest zmiana obrysu płytki oraz położenia otworów montażowych, aby zachować kompatybilność mechaniczną.
* **Pozycjonowanie złączy ChainBus:** Położenie złącza standardu 2x16 SMD (raster 2.54 mm) musi być zgodne z szablonem. Złącze żeńskie montowane jest na stronie FRONT, natomiast złącze męskie na stronie BACK.
* **Komponenty:** Wszystkie komponenty powinny znajdować się na stronie FRONT płytki, aby uniknąć kolizji mechanicznych z elementami sąsiadujących modułów w stosie.

---

## Sekcja 3: Licencja, linki i tagi

### Licencjonowanie projektu

*   **PCB:** [CERN-OHL-W](https://ohwr.org/project/cernohl/wikis/Documents/CERN-OHL-version-2) - Umożliwia modyfikacje i sprzedaż, ale wymaga zachowania informacji o autorze, a wszelkie pochodne projekty muszą być udostępnione jako open source.
*   **Software:** [MIT License](https://opensource.org/licenses/MIT) - Umożliwia dowolne modyfikacje i sprzedaż komercyjną pod warunkiem dołączenia informacji o autorze (kod wynikowy nie musi być udostępniany jako open source).

### Tagi projektu
#chainbus #MMS3 #ModuCard #StepperDriver #TMC5130A #BuckConverter