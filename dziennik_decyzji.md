# Dziennik decyzji analitycznych

Każda decyzja: **co zauważyłem → co zrobiłem → dlaczego**.

## Czyszczenie danych

| # | Problem | Decyzja | Uzasadnienie |
|---|---|---|---|
| 1 | Power Query uznał `numer_linii` za liczbę, przez co `N01`/`N05` dawały błędy | Ręcznie ustawiony typ Tekst | Numer linii to identyfikator, a nie wartość do liczenia |
| 2 | Spacje i różna wielkość liter (`" 105"`, `"n01"`, `"Nocna "`) | Przycinanie + ujednolicenie wielkości liter | Bez tego relacje między tabelami nie łączą wszystkich wierszy |
| 3 | 60 zduplikowanych kursów, 12 zduplikowanych kontroli | Usunięte po kluczu (`id_kursu`, `id_kontroli`) | Duplikaty zawyżałyby liczbę pasażerów i kontroli |
| 4 | 494 kursy bez liczby pasażerów (awaria licznika) | Uzupełnione średnią dla tej samej linii i godziny + flaga `czy_uzupelniono` | Kursy się odbyły; usunięcie lub wpisanie 0 zaniżyłoby stratę |
| 5 | 6 kontroli z liczbą gapowiczów większą niż liczba skontrolowanych | Wartość zamieniona na null, wiersz zachowany | Kontrola się odbyła i ma przypisane mandaty, ale liczba gapowiczów jest niewiarygodna |
| 6 | 13 wariantów zapisu 4 statusów płatności | Przycinanie, małe litery, zamiana wariantów | Bez tego miary statusów liczyłyby tylko część mandatów |
| 7 | Kwota wpłaty jako tekst `"150,00 zł"` | Usunięcie `" zł"`, konwersja na liczbę | Tekstu nie da się sumować |
| 8 | Puste kwoty i daty wpłat | Pozostawione jako puste | To informacja (mandat niezapłacony), a nie błąd |

## Modelowanie i obliczenia

| # | Decyzja | Uzasadnienie |
|---|---|---|
| 9 | Wyłączona automatyczna data/godzina, własna tabela kalendarza | Jedna spójna tabela dat, lżejszy model |
| 10 | Założenia (cena biletu, mandaty, stawka kontrolera) w osobnej tabeli `Parametry` | Zmiana założenia w jednym miejscu |
| 11 | Mianownik wskaźnika gapowiczów tylko z kontroli z wiarygodnymi danymi | Uwzględnienie 6 błędnych kontroli zaniżałoby wskaźnik |
| 12 | Wskaźnik linii liczony z całego okresu (`REMOVEFILTERS` na dacie i godzinie) | Godziny bez kontroli dawałyby zerową stratę. **Uproszczenie:** wskaźnik nie zmienia się w ciągu dnia |
| 13 | Strata liczona przez `SUMX` po liniach | Każda linia ma własny wskaźnik zamiast średniej dla całej sieci |
| 14 | Anulowane mandaty wyłączone z mianownika ściągalności | Anulowany mandat nigdy nie był należny |
| 15 | Parametr What-if „narzut czasu pracy kontrolera” | Koszt z samego czasu w autobusie jest zaniżony; parametr pozwala sprawdzić odporność wniosku |
| 16 | Wskaźnik ukrywany przy mniej niż 30 skontrolowanych | Wyniki z bardzo małej próby (np. 100% przy 2 osobach) wprowadzają w błąd |

## Ograniczenia
- Dane są symulowane.
- Wskaźniki dla linii nocnych opierają się na małej próbie (N01: 71 os., N05: 134 os.).
- Założenie, że kontrole są losowe. Jeśli kontrolerzy wybierają „podejrzane” kursy, wskaźnik jest zawyżony.
