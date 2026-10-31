# POC licencjatu 

Webowa aplikacja dla najmłodszych uczących się zasad ortografii oraz dorosłych, chcących przestestować się na wysokim poziomie trudności. 

Użytkownik testując swoje umiejętności pisania, nie może widzieć tekstu, musi on być mu przeczytany. Należy więc użyć jakiegoś modelu text to speech. Tekst musi być sensownie dzielony na kawałki które moga być przeczytane naraz. Najprościej byłoby zdanie po zdaniu, ale może nie być to najlepsze. 

Ciekawe rzeczy do poszukania (mały research jako część pracy): 
+ jak ocenić trudność takiego 'dyktanda' -- potrzebne jest jakiś aparat statystyczny / własny model językowy 
+ zliczanie błędów -- nie jest to prosta odległość levenshteina
+ wyjaśnianie błędów -- niektóre z zasad są do wyjaśnienia i można by je wdrożyć (np po p używamy rz, w pióro jest ó bo wymienia się na pierze) 
+ adaptacyjność -- program mógłby zbierać informacje o użytkowniku i sugerować dalsze kroki w nauce, zapamiętywać problematyczne wyrazy itp  


Przydatne funckcjonalności: 
+ Konta nauczyciel–uczeń, klasy, zadawanie dyktand z terminem, agregat błędów per reguła dla klasy (heatmapa) i profil ucznia, eksport CSV, konfigurowalne progi ocen.
+ Edytor własnych dyktand z automatycznym oznaczaniem pułapek i wariantów. Generowanie “na zamówienie”: “10 wyrazów z ó wymiennym, poziom X”.
+ Tryb ćwiczeniowy (informacja zwrotna po każdym kawałku) kontra sprawdzian (na końcu, limit powtórzeń odsłuchu), regulacja tempa.
+ Ekran po dyktandzie: diff z podświetleniem i wyjaśnieniem reguły.
+ Ekranowa klawiatura z polskimi znakami dla młodszych użytkowników.

Parę pprawek: istnieją pewne rozwiązania, lepsze i gorsze: 
+ speecher.org -- super interface, szerokie narzędize, tylko 16 tekstów, bardzo multi-jezkowe, nie konkretnie dla polskiego 
+ dyktanda.net -- dosyć podstawowe z nienajlepszym interface'm 


## mvp + dwa elementy badawcze 
+ mvp: dytkando ze sluchy, wpisywanie, diff, podstawowe statystyki, konta nauczyciel-uczen 
+ modul madawczy A; automatyczna detekscja, grupwoanie i klsyfkiacja bledow ortografcicznych h
+ modul badawczy B: wyjasnianie beldow, adaptatcyjny dobor kolejnych cwiczen 


| Komponent           | Odpowiedzialność                                          | Priorytet     |
| ------------------- | --------------------------------------------------------- | ------------- |
| Konta i dane | profil, statstyki, dane, indywidualny uzytkownik| MVP           |
| Biblioteka dyktand  | Tekst, metadane, reguły, poziom, autor, status publikacji | MVP           |
| Odtwarzacz audio    | TTS, segmenty, tempo, powtórzenia, limit odsłuchów        | MVP           |
| Silnik porównania   | Normalizacja, wyrównanie, diff, kandydaci błędów          | MVP + badania |
| Ocena trudności  | na podstawie tekstu dyktanda    | Iteracja 2           |
| Klasyfikator błędów | Kategorie ortograficzne, pewność, wyjaśnienie             | Badania       |
| Profil ucznia       | Historia, słabe reguły, powtórki, rekomendacje            | Iteracja 2    |
| Autoryzacja i role  | Uczeń, nauczyciel, rodzic/opiekun; klasy i przypisania    | Iteracja 2           |
| Panel nauczyciela   | Zadania, terminy, heatmapa klasy, eksport CSV             | Iteracja 2    |
| Generator dyktand   | „10 wyrazów z ó wymiennym, poziom X”                      | Iteracja 3    |


## Kolejność prac 

### Decyzje projektowe 
+ co jest jednym bł3edem 
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
| inny          | nie sklasyfikowane automatycznie               |


### trudność dyktanda: 

koncepcja oceny truności: 

osobna trudność językowa i dyktandyczna(?) 


### adaptacyjność 
**statystyki**: 
+ reguła 
+ słowo 
+ typ błędu 

**profil słabości/siły** 
+ najczestsze typyy błedów 
+ słow ktore są powtarzalnym probelmem 
 itp 

Prawdziwwa daptacyjność: 
+ (prościej) algorytm wyboru dyktanda z dostępnych h
+ (trudniej) generacja dopasowanego indywidualnie dyktanda 


### typy ról 

| Rola           | Uprawnienia                                                   |
| -------------- | ------------------------------------------------------------- |
| Uczeń / indywidualny użytkownik         | Rozwiązuje dyktanda, widzi własny profil i historię           |
| Nauczyciel     | Tworzy klasy, zadaje dyktanda, widzi agregaty, eksportuje CSV |
| Rodzic/opiekun | Opcjonalnie: podgląd postępów dziecka                         |
| Admin          | Zarządza regułami, słownikiem wyjątków, treścią               |


# od nowa 


## Uwagi do tamtego planu

1. **Niespójność MVP.** Wypunktowany MVP zawiera „konta nauczyciel–uczeń", a tabela wrzuca role i autoryzację do iteracji 2. Proponuję rozstrzygnąć tak: MVP ma tylko użytkownika indywidualnego, ale schemat danych od razu przewiduje grupy i przypisania (patrz niżej).
2. **Klasyfikator błędów to nie tylko „badania".** Dla polskiej ortografii prosty klasyfikator regułowy jest tani i daje największą wartość produktu (diff z wyjaśnieniem), czyli główną przewagę nad dyktanda.net. W badania wchodzą dopiero przypadki trudne: wyjaśnianie przez wymianę morfologiczną, niejednoznaczne edycje, interpunkcja.
3. **Brakuje kategorii „literówka".** Trzeba odróżnić pomyłkę regułową (ó↔u) od zwykłego przeklepania. Edycja pasująca do znanej pary pomyłek (ó/u, rz/ż, ch/h, ą/om, ę/em, ś/si, ź/zi, ć/ci) to błąd ortograficzny, a pozostałe edycje o odległości 1 to literówka. Warto też rozbić „nie" na szerszą kategorię pisowni łącznej i rozdzielnej (by, -że, przedrostki). W przykładzie „rz_z" jest chyba pomyłka: „morze → morze" powinno być „może → morze".
4. **Dwupoziomowa taksonomia:** domena (ortografia, interpunkcja, wielka litera, łączna/rozdzielna) i reguła w niej. Przechowywana jako **dane w tabeli**, a nie enum w kodzie, z wersjonowaniem.

## Kluczowa decyzja architektoniczna

**Przechowuj surowe dane, a wszystko pochodne licz na nowo.** Zapisuj dokładnie to, co użytkownik wpisał, per kawałek, razem z liczbą odsłuchów. Wynik analizy zapisuj z polem `analyzer_version`. Wtedy każde ulepszenie silnika błędów, klasyfikatora czy modelu trudności można przeliczyć wstecz na całej historii, bez zmiany architektury. Z tego powodu iteracyjny rozwój jest tu tani.

Druga zasada: **silnik analizy jako czysta biblioteka** (wejście: tekst wzorcowy z adnotacjami i tekst użytkownika; wyjście: lista błędów), bez I/O i bez bazy. Można ją testować zbiorem złotych przypadków i rozwijać niezależnie od reszty. Python jest tu naturalny (Morfeusz 2 / SGJP do odmiany, spaCy). Wersja w Rust → WASM do natychmiastowej informacji zwrotnej w trybie ćwiczeniowym byłaby ciekawa, ale to optymalizacja na później.

## Model danych gotowy na rozszerzenia

- **Role jako relacja w grupie, nie globalny atrybut.** `users`, `groups` (typ: klasa / rodzina), `memberships(user, group, role)`. Nauczyciel–klasa i rodzic–dzieci to ten sam mechanizm. Użytkownik indywidualny nie ma żadnej grupy.
- **`assignments`** (dictation_version, odbiorca: grupa lub użytkownik, `due_at` nullable, ustawienia trybu). Podejście do dyktanda ma nullable `assignment_id`, więc swobodne ćwiczenie to po prostu przypadek bez przypisania. Widok „moje zadania" dla ucznia to zapytanie, które w MVP zwraca pustą listę.
- **`dictations` + niezmienne `dictation_versions`** (pola `owner_id`, `visibility`, `status`). Podejścia wskazują wersję, więc edycja tekstu nie psuje historii. W MVP właścicielem jest „system", a edytor użytkownika dojdzie później bez zmian schematu.
- **Autoryzacja przez jedną funkcję** `can(user, action, resource)` od pierwszego dnia, nawet jeśli w MVP prawie zawsze zwraca true.
- **RODO:** dla dzieci w Polsce zgoda na przetwarzanie danych jest wymagana poniżej 16. roku życia. Kolejny powód, by konto indywidualne mogło działać bez danych osobowych.

## Elementy trudne

**Definicja błędu i wyrównanie.** Zaproponuj wyrównanie na poziomie tokenów z operacjami 1:1, 1:2 i 2:1 (obsługuje „napewno" ↔ „na pewno"), potem wyrównanie znakowe wewnątrz pary słów, które daje konkretne edycje (ó→u). Jeden błąd to jedna para (słowo wzorcowe, słowo użytkownika) z jedną lub kilkoma kategoriami. Zasady oceny (czy dwa błędy w słowie to jeden, ile interpunkcyjnych równa się ortograficznemu) niech będą konfigurowalnym profilem oceniania, a nie zaszyte w silniku. Interpunkcję licz osobnym torem.

**Dzielenie na kawałki.** Klasyczna procedura: całość raz, potem frazy 2 razy, na końcu całość. Granice kawałków to dane w dyktandzie, generowane heurystyką (zdanie, a gdy dłuższe niż ok. 12 słów, podział na przecinkach i spójnikach), z ręczną korektą. Dzięki temu nie jest to decyzja kodu, tylko treści.

**TTS.** Teksty są statyczne, więc **generuj audio z góry** dla każdego kawałka i trzymaj w storage. Wtedy koszt i opóźnienie nie zależą od liczby użytkowników, a każdy plik można odsłuchać przed publikacją. Potrzebne będzie pole z wymową (SSML lub zapis fonetyczny) dla liczb, skrótów i nazw własnych. Do sprawdzenia: polskie głosy neuronowe w chmurze (Azure, Google) versus open source (Piper) oraz licencje na dystrybucję nagrań. Tempo najprościej regulować odtwarzaniem z inną prędkością, a dopiero potem osobnym renderem.

**Trudność.** Rozdziel dwa wymiary: trudność językowa (rzadkość słów, długość zdań) i ortograficzna (gęstość i waga pułapek). Na starcie wagi z heurystyk: częstość słowa, kategoria reguły, wyjątki. Gdy zbierzesz dane, prawdziwym aparatem statystycznym jest model typu **IRT/Rasch**: zdolność ucznia θ i trudność pułapki b, z czego trudność dyktanda to oczekiwana liczba błędów dla danego θ. Surprisal z polskiego modelu językowego może być jedną z cech do prognozy b, ale na zimny start wystarczą heurystyki.

**Adaptacyjność.** Wersja prosta: wektor słabości per reguła i słowo (Elo albo powtórki w stylu FSRS) i wybór dyktanda z biblioteki maksymalizującego pokrycie słabych reguł przy poziomie ok. 70–80% poprawnych. Wersja trudna (generowanie na zamówienie) dopiero na końcu.

**Treści.** Początkowe teksty: domena publiczna (Wolne Lektury) plus własne. Automatyczne oznaczanie pułapek słownikiem i regułami daje pierwszą wersję adnotacji do ręcznej korekty. Unikaj kopiowania dyktand z konkursów i serwisów.

## Kolejność prac

| Etap | Zawartość | Kryterium ukończenia |
|---|---|---|
| 0. Specyfikacja | definicja błędu, taksonomia v1, format danych (tekst + adnotacje + kawałki), 20–30 dyktand z ręcznymi adnotacjami, złoty zestaw par (wzorzec, wpis, oczekiwane błędy) | dokument + dane testowe |
| 1. Silnik | biblioteka + CLI: normalizacja, wyrównanie, klasyfikacja regułowa, literówki | przechodzi złoty zestaw |
| 2. Szkielet end-to-end | pipeline audio, odtwarzacz z kawałkami, pole wpisu, ekran wyniku z diffem | działa jedno dyktando od początku do końca |
| 3. MVP | konta indywidualne, historia i statystyki per reguła i słowo, tryb ćwiczeniowy vs sprawdzian, ekranowa klawiatura | pierwsi testerzy |
| 4. Iteracja 2 | grupy, role, przypisania z terminem, panel nauczyciela, heatmapa, CSV, ocena trudności z danych | pilotaż w jednej klasie |
| 5. Iteracja 3 | edytor dyktand, adaptacyjny dobór, wyjaśnianie przez wymianę morfologiczną, generator | |

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

Teksty są statyczne, więc **nie używaj TTS w czasie rzeczywistym**. Zrób potok na etapie tworzenia treści:

1. Tekst z adnotacjami → podział na kawałki → nadpisania wymowy (SSML) dla liczb, skrótów i nazw własnych.
2. Synteza każdego kawałka osobno oraz całości.
3. Normalizacja głośności, przycięcie ciszy, kodowanie (Opus lub MP3).
4. Zapis do object storage pod nazwą będącą hashem (tekst + głos + SSML + wersja dostawcy). Dzięki temu proces jest idempotentny i przegenerowuje tylko to, co się zmieniło.
5. Manifest (id kawałków, URL, czasy trwania) trafia do wersji dyktanda.

W przeglądarce wystarczy `<audio>` z prefetchem. Tempo to `playbackRate` z zachowaniem wysokości dźwięku, a przy dużych zmianach lepszy osobny render. Limit odsłuchów licz po stronie serwera (zdarzenie „play" w podejściu), bo licznik w kliencie łatwo obejść.

**Koszt syntezy jest zaniedbywalny.** Głosy neuronowe w głównych chmurach kosztują rzędu 16 USD za milion znaków, a wersje HD około 22–30 USD. Azure ma darmowy limit 500 tys. znaków miesięcznie, a Google 1 mln znaków miesięcznie dla głosów neuronowych. 100 dyktand po ok. 1500 znaków, z całością i kawałkami, to rzędu 300 tys. znaków, czyli kilka dolarów jednorazowo albo zero w darmowym limicie. Wybór dostawcy zależy więc od jakości polskich głosów (zrób test odsłuchowy) i warunków licencji na przechowywanie wygenerowanego audio. Schowaj dostawcę za interfejsem `TTSProvider`. Open source (Piper, XTTS) jest opcją, ale wymaga własnej infrastruktury i weryfikacji jakości po polsku.

Koszty zaczną rosnąć dopiero z edytorem własnych dyktand. Wtedy synteza powinna być asynchronicznym zadaniem z kwotą na użytkownika.

## Bazy danych

- **PostgreSQL** jako jedyna baza: tabele relacyjne dla użytkowników, grup, przypisań, podejść i błędów, a JSONB dla adnotacji i surowych wpisów. Agregaty (heatmapa klasy, profil ucznia) to zwykły `GROUP BY` po tabeli błędów. Do milionów wierszy wystarczą indeksy, potem tabele zagregowane lub widoki zmaterializowane.
- **Object storage** zgodny z S3 (Cloudflare R2, Backblaze, Hetzner, S3) na audio, z CDN.
- Słowniki morfologiczne to pliki ładowane przez silnik, nie baza.
- **Pomijaj** na starcie: Redis, Mongo, Elasticsearch, bazy wektorowe. Kolejka zadań (TTS, przeliczenia) może działać na Postgresie. Do badań eksportuj dane do Parquet i analizuj w DuckDB.

## Wdrożenie i rollout

- Wszystko w kontenerach. Lokalnie `docker compose` z tym samym stosem co produkcja.
- CI (GitHub Actions): testy, w tym złoty zestaw silnika, potem build obrazu i automatyczny deploy. Środowiska: lokalne, staging, produkcja.
- Migracje schematu wstecznie zgodne (dodajesz, migrujesz, dopiero potem usuwasz), bo schemat będzie się zmieniał.
- Funkcje (przypisania, role) wdrażane za flagami, zanim będą widoczne dla użytkowników.
- Etapy: prywatna alfa (ty i znajomi), zamknięta beta z jedną klasą lub nauczycielem, potem otwarcie.
- Podstawy operacyjne: codzienny zrzut Postgresa do object storage z próbnym odtworzeniem, Sentry i monitoring dostępności.
- Konta dzieci: tworzone przez nauczyciela z kodem klasy, bez wymogu adresu e-mail. Hosting w UE, mechanizm usuwania danych, zgody opiekunów dla młodszych użytkowników.

Skala nie jest problemem. Klasa 30 osób startująca jednocześnie to burza kilkudziesięciu żądań, a frontend i audio idą z CDN. Jeden mały serwer to wytrzyma.

## Czy hosting w chmurze, a nie na laptopie

Tak. Laptop nie da dostępności, certyfikatów, kopii zapasowych ani stabilnego adresu, a szkolne sieci często blokują nietypowe połączenia. Chmura w tym wariancie nie musi być droga.

**Wariant A: jeden VPS (Hetzner) z docker compose (aplikacja + Postgres + Caddy)** plus object storage i darmowy hosting statycznego frontendu. W zestawieniu z sierpnia 2026 małe instancje Hetznera (CX23, CAX11) kosztują około 5,5–6 € miesięcznie bez VAT, ale Hetzner podnosił ceny już cztery razy w 2026 roku, więc sprawdź aktualny cennik przed decyzją.

**Wariant B: PaaS z zarządzanym Postgresem** (np. Fly, Render, Neon, Supabase). Mniej administracji i automatyczne kopie, ale drożej. Tego nie sprawdzałem w aktualnych cennikach, więc traktuj widełki z tabeli jako szacunek.

| Pozycja | MVP / alfa (wariant A) | Pilotaż w szkołach |
|---|---|---|
| Serwer | ok. 6–7 € | większy VPS lub PaaS + zarządzany Postgres |
| Audio (jednorazowo) | ok. 0–5 USD | to samo, rośnie z edytorem |
| Object storage + CDN | ok. 0–2 € | kilka € (50 MB audio na 100 dyktand to drobiazg) |
| Domena | ok. 50 zł rocznie | to samo |
| Monitoring | darmowe plany | darmowe plany |
| **Razem** | **ok. 8–12 € (30–50 zł) miesięcznie** | **szacunkowo 25–50 €** |

Moja rekomendacja: zacząć od wariantu A w kontenerach, a do pilotażu z prawdziwymi uczniami przejść na zarządzany Postgres, bo kopie zapasowe danych dzieci wolisz mieć po stronie dostawcy. Przy kontenerach i dumpach to przeprowadzka na jeden wieczór, bez zmiany architektury.

Mogę teraz rozpisać etap 0 (definicja błędu, taksonomia v1, schemat JSON dyktanda) albo wstępny schemat tabel Postgresa pod model grup i przypisań.