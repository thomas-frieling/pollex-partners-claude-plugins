---
name: vorgang-verlauf
description: >
  Verlauf eines einzelnen Vorgangs oder einer genannten Familie in den letzten Tagen.
  Greift bei Formulierungen wie „Was ist bei Vorgang AB1234 passiert?“,
  „Was war diese Woche bei Familie Müller?“, „Zeig den Verlauf von …“,
  „Welche Bank hat sich zu Antrag … gemeldet?“. Nicht für den Überblick über alle
  Vorgänge (dafür tagesueberblick) und nicht für Konten oder Anmeldung (dafür zugang-und-konten).
---

# Vorgangsverlauf

Aufgabe: die Ereignisse eines genannten Vorgangs oder eines genannten Kunden der letzten Tage als Zeitleiste wiedergeben.

## Ablauf

1. Fehlt sowohl eine Vorgangsnummer als auch ein Kundenname, einmal nachfragen. Nicht raten.
2. Rufe `list_recent_events` einmal auf:
   - `seit`: `168h`, wenn der Berater „diese Woche“, „zuletzt“ oder nichts zum Zeitraum sagt. Der Server kennt nur Stunden (`24h`, `168h`) oder ein ISO-Datum, kein `7d`. Das Maximum sind 8 Tage (`192h`). Verlangt er mehr, sag dass der Server höchstens 8 Tage liefert, und setze `seit` auf `192h`.
   - `nur_relevante`: false
   - `datenkontext`: `ECHT_GESCHAEFT`
   - `format`: `json`
   - `konto` nur setzen, wenn der Berater ein Konto nennt (Schlüssel oder Bezeichnung). Sonst weglassen.
3. Filtere die Fälle im Feld `cases` selbst:
   - Vorgangsnummer: exakter Vergleich mit `vorgangsNummer` (Groß-/Kleinschreibung ignorieren).
   - Kundenname: Teiltreffer in `applicantNames`. Mehrere Fälle → die Nummern und Kurznamen aufzählen und fragen, welcher gemeint ist. Nichts vermischen.
4. Kein Treffer: das sagen. Nicht das Fenster still vergrößern und nicht ein zweites Mal mit anderen Parametern aufrufen, außer der Berater nennt einen anderen Zeitraum oder ein Konto.

## Antwort

- Eine chronologische Zeitleiste, ältestes Ereignis zuerst. Pro Zeile: Zeitpunkt, Kategorie, Meldung, Akteur.
- Die Bank aus `antraege` über `antragsNummer` zuordnen. Ohne Antrag die Bank weglassen, nicht die erste Bank des Vorgangs unterstellen.
- Offene Punkte nur nennen, wenn sie in `meldung` direkt stehen. Keine eigenen To-dos, keine Einschätzung, ob etwas fehlt.
- `abgeschnitten` oder `unvollstaendig` ist wahr → einen Satz am Ende, dass die Liste unvollständig ist.
- `clamped` ist wahr → einen Satz, dass der Zeitraum auf 8 Tage begrenzt wurde.

## Regeln

- Nichts versenden, nichts in Europace anlegen oder ändern, keine E-Mail und keinen Antrag auslösen.
- `TEST_MODUS` nur, wenn der Berater ausdrücklich Testdaten oder den Testmodus verlangt. Sonst immer `ECHT_GESCHAEFT`.
- Nichts erfinden. Nur Felder aus der Toolantwort: `vorgangsNummer`, `applicantNames`, `antraege`, `ereignisse` (`zeitpunkt`, `kategorie`, `meldung`, `akteur`, `antragsNummer`). Ist ein Feld null oder leer, die Angabe weglassen.
- Kundennamen so lassen, wie der Server sie liefert. Nicht zum vollen Namen ergänzen.
- Kommt ein Fehlertext zurück, nicht erneut aufrufen. Sag in einem Satz, was der Berater tun kann:
  - „Zu viele Abfragen. Bitte eine Minute warten.“ → eine Minute warten, nicht selbst wiederholen.
  - „Konto nicht gefunden.“ → nach der Kontenbezeichnung fragen. Konten nennt `get_profile`.
  - „Scope ticker:read fehlt.“ → die Verbindung zum Baufi Copilot neu anmelden.
  - „Dieses Tool steht nur Finanzierungsmaklern zur Verfügung.“ → das Konto hat nicht die Rolle `LOAN_BROKER`.
  - „datenkontext muss ECHT_GESCHAEFT oder TEST_MODUS sein.“ oder „format muss kompakt oder json sein.“ → das ist ein Aufruffehler. Korrigiere die Parameter einmal, dann nicht weiterprobieren.
  - „seit muss eine Dauer wie 24h oder ein Datum sein.“ → den Zeitraum beim Berater nachfragen.
