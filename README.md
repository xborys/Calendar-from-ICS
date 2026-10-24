# Calendar-from-ICS

Prosta strona, która wyświetla kalendarz tygodniowy z wydarzeniami wczytanymi z pliku `.ics` (iCalendar). Całość to jeden plik `index.html` bez procesu budowania — biblioteki ładowane są z CDN.

## Jak to działa

1. Strona pobiera plik `/calendar-proxy/calendar.ics` (z parametrem `?t=<timestamp>`, żeby ominąć cache).
2. [ical.js](https://github.com/kewisch/ical.js) parsuje plik i rozwija wydarzenia do zakresu dat, który jest akurat widoczny w kalendarzu.
3. [FullCalendar](https://fullcalendar.io/) wyświetla wynik.
4. Co 60 sekund dane są pobierane ponownie, więc zmiany w pliku `.ics` pojawiają się bez przeładowania strony.

## Obsługiwane elementy formatu iCalendar

- pojedyncze wydarzenia i wydarzenia całodniowe (`DTSTART;VALUE=DATE`)
- wydarzenia cykliczne (`RRULE`) ze wszystkimi częściami reguły, m.in. `BYDAY`, `COUNT`, `UNTIL`, `INTERVAL`, `BYMONTHDAY`
- wykluczone wystąpienia (`EXDATE`) i dodatkowe (`RDATE`)
- przeniesione lub zmienione wystąpienia serii (`RECURRENCE-ID`)
- strefy czasowe z pliku (`VTIMEZONE`) — godziny nie przesuwają się po zmianie czasu
- odwołane wydarzenia i wystąpienia (`STATUS:CANCELLED`) są pomijane

Wydarzenia cykliczne mają w tytule dopisek „(Recurring)”.

## Uruchomienie lokalne

Strona musi być serwowana przez HTTP — otwarcie `index.html` bezpośrednio z dysku (`file://`) nie pozwoli pobrać pliku `.ics`.

1. Umieść plik kalendarza w `calendar-proxy/calendar.ics`.
2. Uruchom dowolny serwer statyczny w katalogu projektu, np.:

   ```bash
   python3 -m http.server 8000
   ```

3. Otwórz http://localhost:8000/.

Ścieżka do pliku jest absolutna (`/calendar-proxy/calendar.ics`), więc aplikacja musi być serwowana z głównego katalogu serwera.

## Konfiguracja

Wszystko ustawia się w `index.html`:

| Co | Gdzie | Domyślnie |
| --- | --- | --- |
| Adres pliku `.ics` | stała `icsUrl` | `/calendar-proxy/calendar.ics` |
| Częstotliwość odświeżania | `setInterval(...)` | 60 s |
| Widoczne godziny | `slotMinTime`, `slotMaxTime` | 07:00–17:00 |
| Ukryte dni | `hiddenDays` | sobota i niedziela |
| Domyślny widok | `initialView` | tydzień (`timeGridWeek`) |
| Tytuł strony | `<title>` i `<h2>` | `NAME` (do podmiany) |

Wydarzenia poza widocznymi godzinami i w ukryte dni nie są wyświetlane.

## Zależności (CDN)

- FullCalendar 6.1.15
- ical.js 1.4.0

## Prywatność

Jeśli `calendar-proxy/calendar.ics` zawiera prawdziwy kalendarz, nie commituj go do repozytorium — dodaj go do `.gitignore`.
