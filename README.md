# Calendar-from-ICS

A simple page that shows a weekly calendar with events loaded from an `.ics` (iCalendar) file. The whole app is a single `index.html` with no build step — libraries are loaded from a CDN.

## How it works

1. The page fetches `/calendar-proxy/calendar.ics` (with a `?t=<timestamp>` parameter to bypass the cache).
2. [ical.js](https://github.com/kewisch/ical.js) parses the file and expands the events for the date range currently visible in the calendar.
3. [FullCalendar](https://fullcalendar.io/) renders the result.
4. The data is fetched again every 60 seconds, so changes to the `.ics` file show up without reloading the page.

## Supported iCalendar features

- single events
- recurring events (`RRULE`) with all rule parts, including `BYDAY`, `COUNT`, `UNTIL`, `INTERVAL` and `BYMONTHDAY`
- excluded occurrences (`EXDATE`) and additional ones (`RDATE`)
- moved or modified occurrences of a series (`RECURRENCE-ID`)
- time zones defined in the file (`VTIMEZONE`) — times don't shift after daylight saving changes
- cancelled events and occurrences (`STATUS:CANCELLED`) are skipped

Recurring events get a "(Recurring)" suffix in their title.

## Running locally

The page has to be served over HTTP — opening `index.html` directly from disk (`file://`) won't allow it to fetch the `.ics` file.

1. Put your calendar file at `calendar-proxy/calendar.ics`.
2. Start any static file server in the project directory, e.g.:

   ```bash
   python3 -m http.server 8000
   ```

3. Open http://localhost:8000/.

The file path is absolute (`/calendar-proxy/calendar.ics`), so the app must be served from the server's root.

## Configuration

Everything is set in `index.html`:

| What | Where | Default |
| --- | --- | --- |
| `.ics` file URL | `icsUrl` constant | `/calendar-proxy/calendar.ics` |
| Refresh interval | `setInterval(...)` | 60 s |
| Visible hours | `slotMinTime`, `slotMaxTime` | 07:00–17:00 |
| Hidden days | `hiddenDays` | Saturday and Sunday |
| Default view | `initialView` | week (`timeGridWeek`) |
| Page title | `<title>` and `<h2>` | `NAME` (placeholder) |

Events outside the visible hours or on hidden days are not shown. The calendar has no "all-day" row, so all-day events (`DTSTART;VALUE=DATE`) are not shown either.

## Dependencies (CDN)

- FullCalendar 7.1.3 (the `all/global.js` bundle with a global `FullCalendar`, plus the `classic` theme; since v7 styles are no longer bundled, so `skeleton.css`, `themes/classic/theme.css` and `themes/classic/palette.css` are loaded separately)
- ical.js 2.2.1 (an ES module imported in `<script type="module">`; since 2.x the library no longer exposes a global `ICAL`)

## Privacy

If `calendar-proxy/calendar.ics` contains a real calendar, don't commit it to the repository — add it to `.gitignore`.
