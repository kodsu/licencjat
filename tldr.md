# POC licencjatu 

Webowa aplikacja dla najmłodszych uczących się zasad ortografii oraz dorosłych, chcących przestestować się na wysokim poziomie trudności. 

Użytkownik testując swoje umiejętności pisania, nie może widzieć tekstu, musi on być mu przeczytany. Należy więc użyć jakiegoś modelu text to speech. Tekst musi być sensownie dzielony na kawałki które moga być przeczytane naraz. Najprościej byłoby zdanie po zdaniu, ale może nie być to najlepsze. 

Ciekawe rzeczy do poszukania (mały research jako część pracy): 
+ jak ocenić trudność takiego 'dyktanda' -- potrzebne jest jakiś aparat statystyczny / własny model językowy 
+ zliczanie błędów -- nie jest to prosta odległość levenshteina
+ wyjaśnianie błędów -- niektóre z zasad są do wyjaśnienia i można by je wdrożyć (np `po p używamy rz, w pióro jest ó bo wymienia się na e w pierze`) 
+ adaptacyjność -- program mógłby zbierać informacje o użytkowniku i sugerować dalsze kroki w nauce, zapamiętywać problematyczne wyrazy itp  


Przydatne funckcjonalności: 
+ Konta nauczyciel–uczeń, klasy, zadawanie dyktand z terminem, agregat błędów per reguła dla klasy (heatmapa) i profil ucznia, eksport CSV, konfigurowalne progi ocen.
+ Edytor własnych dyktand z automatycznym oznaczaniem pułapek i wariantów. Generowanie “na zamówienie”: “10 wyrazów z ó wymiennym, poziom X”.
+ Tryb ćwiczeniowy (informacja zwrotna po każdym kawałku) kontra sprawdzian (na końcu, limit powtórzeń odsłuchu), regulacja tempa.
+ Ekran po dyktandzie: diff z podświetleniem i wyjaśnieniem reguły.
+ Ekranowa klawiatura z polskimi znakami dla młodszych użytkowników.

Parę poprawek: istnieją pewne rozwiązania, lepsze i gorsze: 
+ speecher.org -- super interface, szerokie narzędize, tylko 16 tekstów, bardzo multi-jezkowe, nie konkretnie dla polskiego 
+ dyktanda.net -- dosyć podstawowe z nienajlepszym interface'm 


## mvp +  elementy do zbadania  


| Komponent           | Odpowiedzialność                                          | Priorytet     |
| ------------------- | --------------------------------------------------------- | ------------- |
| Konta i dane | profil, statstyki, dane, indywidualny uzytkownik| MVP           |
| Biblioteka dyktand  | Tekst, metadane, reguły, poziom, autor, status publikacji | MVP           |
| Odtwarzacz audio    | TTS, segmenty, tempo, powtórzenia, limit odsłuchów        | MVP           |
| Silnik porównania   | Normalizacja, wyrównanie, diff, kandydaci błędów          | MVP  |
| Klasyfikator błędów | Kategorie ortograficzne, pewność, wyjaśnienie             | MVP + dalsze iteracje       |
| Podstawowe dodawanie dyktand   | z przygowoanego tekstu| MVP    |
| Inteligentne generowanie i rekomendacja dyktand   | na podstawie błędów w profilu| Iteracja 2    |
| Ocena trudności  | na podstawie tekstu dyktanda: liczby 'pułapek' i statystyczności słowa    | Iteracja 2           |
| Profil ucznia       | Historia, słabe reguły, powtórki, rekomendacje            | Iteracja 2    |
| Wyjaśnianie błędów   | Po zidentyfikowaniu, próba automatycznego wyjaśnienia jaką z reguł została złamana       | dalszy kierunek    |
| Autoryzacja i role  | Uczeń, nauczyciel, rodzic/opiekun; klasy i przypisania    | dalszy kierunek           |
| Panel nauczyciela   | Zadania, terminy, heatmapa klasy, eksport CSV             | dalszy kierunek    |





### Decyzje projektowe 
+ co jest jednym błędem 
+ interpunkcja do rozstrzygnięcia 
+ format danychh

### silnik błedów 
```
tekst wzorcowy
        │
        ▼
normalizacja + tokenizacja
        │
        ▼
wyrównanie tokenów / fraz
        │
        ▼
kandydaci edycji
        │
        ▼
klasyfikacja edycji
        │
        ▼
scalenie w błędy językowe
        │
        ▼
wyjaśnienie + zapis do profilu
```

Warto rozważyć pewną hierarchie błedów: 
| Kategoria     | Przykłady                                      |
| ------------- | ---------------------------------------------- |
| ó_u           | gura → góra, piuro → pióro                     |
| rz_z          | żeka → rzeka, morze → morze                    |
| ch_h          | hleb → chleb, chemia → hemia                   |
| nie           | napewno → na pewno, nie boję się → nieboję się |
| wielka_litera | wrocław → Wrocław                              |
| ą_om_on       | robiom → robią                                 |
| ę_em_en       | widzem → widzę                                 |
| interpunkcja  | brak przecinka, błędny znak końcowy            |
| pominięcie    | brakujące słowo                                | 
| dodanie       | zbędne słowo                                   |
| inny / pojedyncza literówka / zamiana liter          | nie sklasyfikowane automatycznie               |


### trudność dyktanda: 

koncepcja oceny truności: 

Proste podejście (MVP): na podstawie listy 'pułapek' ortograficznych i częstości słów 

Docelowe (iteracja 2): uwzględnienie statyki cząstek wyrazowych / tokenów  

popatrzeć w tekście z błędami jak często w danych tokenach popełniane są błędy. 
np. podział wg sylab / n-gramów 

```
pż bardzo rzadkie: sypże 

prze szłość  

gó ra
gu ma 

rów ni na 
fru wa 

zół tko 
rzu cać

pie gża  
pod grzać 

ohy da 
och ła py 
zes chnię ty  
```


### adaptacyjność 
**statystyki**: 
+ reguła 
+ słowo 
+ typ błędu 

**profil słabości/siły** 
+ najczestsze typy błedów 
+ słow ktore są powtarzalnym probelmem 
 itp 

Prawdziwwa daptacyjność: 
**(prościej) algorytm wyboru dyktanda z dostępnych**

+ (trudniej) generacja dopasowanego indywidualnie dyktanda 


### typy ról (dalszy kierunek) 

| Rola           | Uprawnienia                                                   |
| -------------- | ------------------------------------------------------------- |
| Uczeń / indywidualny użytkownik         | Rozwiązuje dyktanda, widzi własny profil i historię           |
| Nauczyciel     | Tworzy klasy, zadaje dyktanda, widzi agregaty, eksportuje CSV |
| Rodzic/opiekun | Opcjonalnie: podgląd postępów dziecka                         |
| Admin          | Zarządza regułami, słownikiem wyjątków, treścią               |



## Kolejność prac

| Etap | Zawartość | Kryterium ukończenia |
|---|---|---|
| 0. Specyfikacja | definicja błędu, taksonomia v1, format danych (tekst + adnotacje + kawałki), 20–30 dyktand z ręcznymi adnotacjami, złoty zestaw par (wzorzec, wpis, oczekiwane błędy) | dokument + dane testowe |
| 1. Silnik | biblioteka + CLI: normalizacja, wyrównanie, klasyfikacja regułowa, literówki | przechodzi złoty zestaw |
| 2. Szkielet end-to-end | pipeline audio, odtwarzacz z kawałkami, pole wpisu, ekran wyniku z diffem | działa jedno dyktando od początku do końca |
| 3. MVP | konta indywidualne, historia i statystyki per reguła i słowo, tryb ćwiczeniowy vs sprawdzian, ekranowa klawiatura | pierwsi testerzy |
| 4. Iteracja 2 | grupy, role, przypisania z terminem, panel nauczyciela, heatmapa, CSV, ocena trudności z danych | pilotaż w jednej klasie |
| 5. dalszy kierunek | edytor dyktand, adaptacyjny dobór, wyjaśnianie przez wymianę morfologiczną, generator | |

Szkielet z etapu 2 warto zrobić wcześnie, nawet na prymitywnym silniku, żeby od razu widzieć, co naprawdę boli w interfejsie. Silnik z etapu 1 jest jednak najbardziej ryzykowną i najciekawszą częścią, więc zaczynałbym od niego i od złotego zestawu.

Jeśli chcesz, mogę teraz rozpisać etap 0: konkretną definicję błędu, taksonomię v1 i schemat JSON dyktanda z adnotacjami.


# technologia 

Krótko: **Python do całej logiki analitycznej, Postgres jako jedyna baza, audio generowane z góry i serwowane jako pliki, a na początek jeden mały VPS w Europie.** Dalej uzasadnienie.

## Języki programowania

| Komponent | Język | Dlaczego |
|---|---|---|
| Silnik porównań, klasyfikacja i liczenie błędów | **Python** | Wyrównanie to prosty DP na kilkuset tokenach, więc wydajność nie jest problemem. Za to ekosystem NLP (Morfeusz 2, spaCy, HuggingFace) jest tu zdecydowanie najlepszy. |
| Ocena trudności, IRT, analizy | **Python** (batch offline) | Pandas, statsmodels i notebooki. Wynik trafia do bazy jako metadane dyktanda z wersją modelu, poza ścieżką żądania. |
| API i logika biznesowa | **Python (FastAPI)** | Silnik jest importowany jako biblioteka w tym samym procesie, bez mikroserwisów. |
| Frontend | **TypeScript + React** | Odtwarzacz, pole wpisu, klawiatura ekranowa, diff. |

Przepisanie silnika na Rust/WASM (natychmiastowa informacja w trybie ćwiczeniowym, działanie offline) jest realne, ale to optymalizacja na później. Żeby ją ułatwić, trzymaj złoty zestaw testów jako dane w JSON, niezależne od języka. Słowniki i licencje (Morfeusz/SGJP) sprawdź przed publicznym wdrożeniem.

## Generowanie mowy

Ma być zapisana per dyktando, pocięta na odpowiednie kawałki. 
Przetestowanie (subiektywne) sensownych rozwiązań na rynku pod katem polskiego, które najlepiej sobie radzą z trudnymi wyrazami np: `chrząszcz`. 

## Bazy danych

- **PostgreSQL** jako jedyna baza: tabele relacyjne dla użytkowników, grup, przypisań, podejść i błędów, a JSONB dla adnotacji i surowych wpisów. Agregaty (heatmapa klasy, profil ucznia) to zwykły `GROUP BY` po tabeli błędów. Do milionów wierszy wystarczą indeksy, potem tabele zagregowane lub widoki zmaterializowane.
- **Object storage** na pliki .mp3 



## Wdrożenie i rollout

- Wszystko w kontenerach. Lokalnie `docker compose` z tym samym stosem co produkcja.
- CI (GitHub Actions): testy, w tym złoty zestaw silnika, potem build obrazu i automatyczny deploy. Środowiska: lokalne, staging, produkcja.
- Migracje schematu wstecznie zgodne (dodajesz, migrujesz, dopiero potem usuwasz), bo schemat będzie się zmieniał.
- Funkcje (przypisania, role) wdrażane za flagami, zanim będą widoczne dla użytkowników.

--- 

## Stack

| Warstwa | Wybór | Uwagi |
|---|---|---|
| Silnik | Python, biblioteka + CLI | bez zmian, bez zależności od frameworka |
| API | **FastAPI** | JSON, schematy w Pydantic, z których wychodzi OpenAPI |
| Baza | **PostgreSQL** + **SQLAlchemy** + **Alembic** | migracje od pierwszego dnia, JSONB na adnotacje |
| Frontend | **React + TypeScript + Vite** | React Router, TanStack Query do zapytań; bez Reduxa |
| Typy | generowane z OpenAPI (np. `openapi-typescript`) | jedno źródło prawdy dla kontraktu front–back |
| Stylowanie | zwykły CSS lub Tailwind | nie bierz biblioteki komponentów, dopóki nie boli |
| Auth | e-mail + hasło, sesja w ciasteczku | gotowa biblioteka do FastAPI albo kilkadziesiąt linii własnych, sprawdź aktualny stan bibliotek |
| Audio | pliki generowane offline, serwowane statycznie | ścieżki zapisane w bazie |
| Testy | pytest (silnik, API na testowej bazie), opcjonalnie Vitest | |
| Uruchamianie | **docker compose** (api + db) lokalnie i w produkcji | front buduje się do statycznych plików serwowanych przez API lub przez prosty reverse proxy |

## Ważne decyzje architektoniczne

1. **Tekst dyktanda nigdy nie trafia do przeglądarki przed złożeniem odpowiedzi.** Endpoint dyktanda zwraca tylko metadane i adresy audio kawałków. Tekst wzorcowy zostaje na serwerze i jest używany wyłącznie przez silnik. W trybie ćwiczeniowym serwer po wysłaniu wpisu dla kawałka zwraca diff tego kawałka. Inaczej użytkownik zajrzy w zakładkę sieci.
2. **Silnik działa po stronie serwera.** Każde sprawdzenie to jedno wywołanie API. Latencja w trybie ćwiczeniowym jest akceptowalna, a nie musisz utrzymywać dwóch implementacji.
3. **Źródłem prawdy dla dyktand są pliki w repo** (JSON/YAML). Skrypt `seed` wczytuje je do bazy. Dzięki temu dyktanda są wersjonowane w gicie, a baza to tylko stan wykonania.
4. **Limit odsłuchów w sprawdzianie:** w MVP licz po stronie klienta i zapisuj w podejściu. Wymuszenie po stronie serwera (audio przez endpoint, który liczy żądania) zostaw na później i opisz jako ograniczenie.

## Schemat bazy (MVP)

- `users` (id, e-mail, hasło zahaszowane, data utworzenia)
- `dictations` (id, slug, tytuł, poziom, wersja, metadane JSONB, status)
- `chunks` (id, dictation_id, kolejność, tekst wzorcowy, ścieżka audio)
- `attempts` (id, user_id, dictation_id, tryb, status, czas rozpoczęcia i zakończenia, liczba odsłuchów)
- `attempt_chunks` (attempt_id, chunk_id, surowy wpis użytkownika)
- `errors` (attempt_chunk_id, kategoria, oczekiwane, wpisane, pozycja, pewność)

Statystyki per reguła to zwykły `GROUP BY kategoria` po `errors`. Statystyki per słowo wymagają pola „wyraz wzorcowy" w `errors`, więc zapisz je od razu, bo dodanie później boli.

## Endpointy (szkic)

- `POST /auth/register`, `POST /auth/login`, `POST /auth/logout`
- `GET /dictations`, `GET /dictations/{id}` (bez tekstu, z kawałkami audio)
- `POST /attempts` (start podejścia, tryb), `POST /attempts/{id}/chunks/{n}` (wpis kawałka), `POST /attempts/{id}/finish`
- `GET /attempts/{id}` (wynik z diffem i błędami)
- `GET /me/stats` (agregaty per reguła i per słowo), `GET /me/attempts`

## Widoki React

- Lista dyktand
- Ekran dyktanda: odtwarzacz kawałków (powtórz, dalej, tempo), pole wpisu, ekranowa klawiatura, licznik odsłuchów
- Ekran wyniku: diff z podświetleniem, lista błędów z kategorią i wyjaśnieniem
- Historia podejść
- Statystyki: błędy per reguła, najczęstsze słowa
- Logowanie/rejestracja

## Etapy

**Etap 0. Specyfikacja**: definicja błędu z przypadkami granicznymi, taksonomia v1, format dyktanda, złoty zestaw par.
*Kryterium:* dokument + dane testowe.

**Etap 1. Silnik** : normalizacja, wyrównanie dwupoziomowe, klasyfikacja regułowa, literówki, CLI.
*Kryterium:* przechodzi złoty zestaw, z precision/recall per kategoria.

**Etap 2. Audio**: skrypt generujący kawałki i krótki test kilku silników TTS na trudnych wyrazach.
*Kryterium:* jedno dyktando z dobrym audio.

**Etap 3. Backend**: schemat, migracje, seed dyktand, endpointy, testy API na testowej bazie. Sprawdzenie ręcznie przez dokumentację OpenAPI lub skrypt.
*Kryterium:* całe dyktando da się przejść samym API, a tekst wzorcowy nie wycieka w żadnej odpowiedzi (test).

**Etap 4. Frontend szkieletowy**: lista, odtwarzacz, pole wpisu, ekran wyniku. Bez klawiatury i statystyk.
*Kryterium:* jedno dyktando działa end-to-end w przeglądarce.

**Etap 5. MVP**: konta, historia, statystyki per reguła i słowo, tryb ćwiczeniowy vs sprawdzian, tempo, ekranowa klawiatura, kilkanaście dyktand, wdrożenie.
*Kryterium:* pierwsi testerzy przechodzą dyktando bez Twojej pomocy.

**Później:** ocena trudności z danych, adaptacyjny dobór, wyjaśnianie przez wymianę morfologiczną, role, klasy, panel nauczyciela, edytor i generator.

## Struktura repozytorium

```
dyktando/
├── engine/            # biblioteka Pythona (bez zależności od API)
├── api/               # FastAPI, modele, migracje (alembic)
├── web/               # React + TS (Vite)
├── data/
│   ├── dictations/    # JSON/YAML
│   └── gold/          # złoty zestaw
├── tts/               # skrypt generujący audio
├── tests/
└── docker-compose.yml
```

## Co się zmieniło względem poprzedniej wersji i na co uważać

- React i Postgres dokładają pracy infrastrukturalnej (migracje, auth, build frontendu). Żeby nie zjadło to części badawczej, trzymaj kolejność: **silnik i złoty zestaw przed backendem**, a frontend na początku zrób brzydki, ale kompletny.
- Auth jest najłatwiejszym miejscem do przeinwestowania. Weź gotowe rozwiązanie i nie rozbudowuj (bez resetu hasła, weryfikacji e-maila i ról w MVP).
- Konta zbierają dane osobowe (e-mail), więc dopisz do planu krótką notkę o RODO. Przy użytkownikach-dzieciach to ważniejsze, więc na etapie pilotażu rozważ konta anonimowe lub pseudonimy.

Mogę teraz rozpisać Etap 0: definicję błędu, taksonomię v1 i schemat JSON dyktanda, albo szczegółowo schemat bazy z typami i indeksami.