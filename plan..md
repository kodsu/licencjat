Zgoda: MVP można zrobić bez danych użytkowników. Wszystko, co ich wymaga (ocena trudności z danych, profil, adaptacyjność), przechodzi do późniejszych iteracji. Poniżej plan z założeniem „jak najmniej części ruchomych".

## Zakres MVP

Jedna ścieżka: **wybierz dyktando → odsłuchaj → wpisz → zobacz błędy z kategorią i wyjaśnieniem.**

- Bez kont, ról, klas, historii i statystyk.
- Biblioteka dyktand to kilkanaście plików z góry przygotowanych przez Ciebie.
- Tryb ćwiczeniowy i sprawdzian: ten sam silnik, różni się tylko tym, kiedy pokazujesz wynik i czy jest limit odsłuchów. To kilka linijek logiki, więc może wejść do MVP.
- Interpunkcja poza zakresem (tylko ortografia, wielkie litery, pisownia łączna/rozdzielna).

## Stos technologiczny

| Warstwa | Wybór | Uwagi |
|---|---|---|
| Silnik | **czysty Python**, biblioteka + CLI | `difflib` albo własne DP do wyrównania, bez zewnętrznych zależności na start |
| Testy | **pytest** + złoty zestaw w JSON | testy parametryzowane danymi, więc zestaw jest niezależny od języka |
| Web | **FastAPI + szablony Jinja2** | jedna aplikacja, bez osobnego frontendu |
| Frontend | **zwykły HTML + trochę JS** (lub htmx) | `<audio>`, `<textarea>`, przycisk. React niepotrzebny |
| Audio | **skrypt Pythona generujący pliki mp3/wav offline** | pliki serwowane jako statyczne |
| Dane dyktand | **pliki JSON/YAML w repozytorium** | tekst, kawałki, metadane, adnotacje pułapek |
| Baza | **żadna** w MVP (ewentualnie SQLite później) | brak kont, więc brak potrzeby |
| Wdrożenie | pojedynczy kontener lub zwykły hosting dla aplikacji Pythona | Docker dopiero gdy będzie potrzebny |

Dlaczego nie Streamlit/Gradio: są szybkie w prototypie, ale utrudniają to, czego potrzebujesz (ukrycie tekstu, limit odsłuchów, własny odtwarzacz z kawałkami, ekranowa klawiatura). FastAPI z szablonami jest tylko trochę bardziej pracochłonne, a daje pełną kontrolę.

## Etapy

**Etap 0. Specyfikacja (dokument + dane)**
- Definicja błędu z przypadkami granicznymi (napewno, dwa błędy w jednym wyrazie, powtórzony błąd, literówka a błąd ortograficzny).
- Taksonomia v1 (ó_u, rz_ż, ch_h, nie, wielka litera, pominięcie, dodanie, inny).
- Format dyktanda: tekst, podział na kawałki, lista pułapek z kategorią.
- Złoty zestaw: kilkadziesiąt do ~100–200 par (wzorzec, wpis, oczekiwane błędy). Część wpisów wygeneruj mutacjami, część wpisz ręcznie.
- *Kryterium ukończenia:* dokument + plik JSON ze złotym zestawem.

**Etap 1. Silnik (biblioteka + CLI)**
- Normalizacja i tokenizacja.
- Wyrównanie dwupoziomowe: tokeny, a w niedopasowanych tokenach znaki.
- Klasyfikacja regułowa po parze znaków w wyrównaniu (ó↔u, rz↔ż, ch↔h) oraz osobne reguły kontekstowe dla „nie" i wielkiej litery.
- Wszystko, czego nie rozpoznasz, ląduje w kategorii „inny/literówka".
- *Kryterium:* przechodzi złoty zestaw, z podanym recall/precision per kategoria.

**Etap 2. Audio**
- Skrypt: dyktando (JSON) → pocięte kawałki → pliki audio.
- Mały test odsłuchowy kilku silników TTS na trudnych wyrazach (rz/ż, ó/u, ch/h) i wybór jednego. Tu najbardziej ryzykowna jest wymowa, więc zrób to wcześnie.
- *Kryterium:* jedno dyktando ma komplet plików, które dobrze brzmią.

**Etap 3. Szkielet end-to-end**
- Strona z listą dyktand, odtwarzacz kawałków (powtórz, dalej), pole wpisu, ekran wyniku z diffem i wyjaśnieniem kategorii.
- Wyjaśnienia na początku jako statyczne teksty per kategoria (np. „ó, bo wymienia się na o/e"), bez analizy morfologicznej.
- *Kryterium:* jedno dyktando działa od początku do końca w przeglądarce.

**Etap 4. MVP**
- Kilkanaście dyktand, tryb ćwiczeniowy vs sprawdzian z limitem odsłuchów, regulacja tempa, ekranowa klawiatura z polskimi znakami.
- Wdrożenie w jednym miejscu, żeby mogli kliknąć testerzy.
- *Kryterium:* kilka osób przechodzi całe dyktando bez Twojej pomocy.

**Później (poza MVP)**
- Zapis podejść do bazy (SQLite → Postgres), konta, profil ucznia.
- Ocena trudności z danych, adaptacyjny dobór dyktand.
- Wyjaśnianie przez wymianę morfologiczną (Morfeusz/SGJP).
- Role, klasy, panel nauczyciela, edytor i generator dyktand.

## Jedna rzecz warto dodać już teraz, choć nie jest wymagana

Przy przechodzeniu dyktanda zapisuj anonimowo jeden rekord (id dyktanda, wpis, wykryte błędy, czas) do pliku lub tabeli SQLite. To kilka linijek kodu, bez kont i danych osobowych, a po paru miesiącach testów dostajesz pierwsze prawdziwe dane do oceny trudności. Jeśli Ci to nie pasuje, całkowicie pomiń, MVP działa bez tego.

## Struktura repozytorium

```
dyktando/
├── engine/          # normalizacja, wyrównanie, klasyfikacja
├── data/
│   ├── dictations/  # JSON: tekst, kawałki, pułapki
│   └── gold/        # złoty zestaw
├── tts/             # skrypt generujący audio
├── web/             # FastAPI + szablony + static
└── tests/
```

Mogę teraz rozpisać Etap 0: konkretną definicję błędu, taksonomię v1 i schemat JSON dyktanda z adnotacjami.