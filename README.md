## Rój dronów

### 1. Ogólny opis projektu.

Projekt polega na symulacji cyklu życia **roju autonomicznych dronów**, działających w oparciu o mechanizmy systemów operacyjnych w środowisku Linux/UNIX. System składa się z trzech głównych typów procesów: **dowódcy**, **operatora** oraz **dronów**, które współpracują ze sobą i komunikują się przy użyciu procesów, wątków, sygnałów oraz mechanizmów IPC.

Każdy dron funkcjonuje jako niezależna jednostka wykonawcza, realizując cykl:
**lot → powrót do bazy → ładowanie → ponowny start**.  
Liczba dronów jest dynamiczna i zależna od decyzji dowódcy oraz dostępnej pojemności bazy. Ograniczenia systemowe obejmują m.in. maksymalną liczbę dronów w bazie, jednokierunkowy ruch w wąskich wejściach do bazy, czas lotu zależny od poziomu naładowania baterii oraz ryzyko zniszczenia drona przy jej całkowitym rozładowaniu.

Dowódca systemu zarządza globalnymi parametrami roju poprzez wysyłanie sygnałów do operatora i pojedynczych dronów (np. rozbudowa platform startowych, redukcja floty, atak samobójczy). Operator odpowiada za uzupełnianie braków w liczbie dronów, o ile pozwala na to aktualna pojemność bazy.

Cała symulacja generuje **raport tekstowy**, zapisywany do plików, dokumentujący przebieg działania systemu, zdarzenia oraz aktualne stany dronów.

Projekt ma na celu praktyczne wykorzystanie i zaprezentowanie **kluczowych funkcji systemowych** związanych z zarządzaniem procesami, wątkami, synchronizacją, komunikacją międzyprocesową, obsługą sygnałów oraz operacjami na plikach.

# 2. Ogólny opis kodu

Projekt został podzielony na **kilka logicznie rozdzielonych plików źródłowych**, z których każdy odpowiada za odrębny element symulowanego systemu. Taki podział ułatwia rozwój projektu, zwiększa czytelność kodu oraz pozwala na łatwiejsze testowanie i debugowanie poszczególnych komponentów.

## Struktura projektu

- **`dowodca.c`**  
  Główny proces sterujący symulacją. Odpowiada za inicjalizację parametrów systemu (liczba dronów, pojemność bazy, czasy), tworzenie procesu operatora oraz obsługę interfejsu decyzyjnego dowódcy. Realizuje wysyłanie sygnałów sterujących do operatora oraz nadzoruje poprawne zakończenie systemu.

- **`operator.c`**  
  Proces pośredniczący pomiędzy dowódcą a dronami. Odpowiada za tworzenie i usuwanie dronów, reagowanie na sygnały dowódcy (rozbudowa, redukcja, atak), okresowe uzupełnianie liczby dronów oraz sprzątanie zakończonych procesów potomnych. Zarządza również zasobami współdzielonymi systemu.

- **`dron.c`**  
  Implementacja pojedynczego drona jako autonomicznego procesu. Dron realizuje własny cykl życia (lot, powrót, ładowanie), zarządza poziomem baterii oraz reaguje na sygnały systemowe (atak samobójczy). Logika drona uwzględnia ograniczenia bazy, dostępność wejść oraz warunki awaryjne.

- **`semafory.c`**  
  Moduł odpowiedzialny za synchronizację dostępu do pamięci współdzielonej przy użyciu semaforów System V. Zapewnia ochronę sekcji krytycznych i spójność danych globalnych.

- **`pamiec.c`**  
  Obsługa segmentów pamięci dzielonej. Plik odpowiada za tworzenie, podłączanie, odłączanie oraz usuwanie pamięci współdzielonej używanej do przechowywania aktualnego stanu systemu.

- **`kolejka.c`**  
  Implementacja kolejek komunikatów System V wykorzystywanych do synchronizacji dostępu do wejść bazy. Kolejka pełni rolę mechanizmu kontroli jednokierunkowego ruchu dronów przez dwa wąskie wejścia.

- **`log.c`**  
  Moduł odpowiedzialny za rejestrowanie przebiegu symulacji. Zapewnia spójny, czasowo oznaczony zapis zdarzeń do pliku logu.

- **`bledy.c`**  
  Prosty moduł pomocniczy do obsługi błędów krytycznych i kończenia programu w przypadku niepowodzeń operacji systemowych.

- **`shared.h`**  
  Wspólny plik nagłówkowy zawierający deklaracje struktur, zmiennych globalnych oraz prototypy funkcji używanych w całym projekcie.

## Zastosowane rozwiązania zwiększające wydajność i stabilność

- **Asynchroniczna komunikacja za pomocą sygnałów**  
  Sygnały systemowe pozwalają na natychmiastową reakcję procesów bez konieczności ciągłego odpytywania stanu (busy waiting).

- **Minimalizacja sekcji krytycznych**  
  Dostęp do pamięci współdzielonej jest chroniony semaforami tylko w niezbędnych fragmentach kodu, co ogranicza czas blokowania i poprawia równoległość działania procesów.

- **Dynamiczne zarządzanie zasobami**  
  Lista aktywnych dronów jest alokowana dynamicznie i rozszerzana w razie potrzeby, co pozwala na efektywne wykorzystanie pamięci.

- **Oddzielenie logiki od mechanizmów IPC**  
  Wyodrębnienie obsługi semaforów, pamięci i kolejek do osobnych plików zwiększa czytelność kodu oraz ułatwia dalszą rozbudowę systemu.

- **Obsługa sygnałów SIGCHLD**  
  Zapewnia bieżące usuwanie zakończonych procesów dronów, zapobiegając powstawaniu procesów zombie.

---

