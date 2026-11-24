---
name: tagesueberblick
description: >
  Tages- und Morgenüberblick über die letzten Vorgangsereignisse des angemeldeten
  Finanzierungsmaklers. Greift bei Formulierungen wie „Was ist heute passiert?“,
  „Was gibt's Neues?“, „Was ist seit gestern passiert?“, „Morgenüberblick“,
  „Was muss ich heute wissen?“. Nicht für einen einzelnen Vorgang oder eine einzelne Familie
  (dafür vorgang-verlauf) und nicht für Konten oder Anmeldung (dafür zugang-und-konten).
---

# Tagesüberblick

Aufgabe: die letzten Ereignisse über alle Konten des angemeldeten Beraters kurz zusammenfassen. Kein Postfach öffnen und keine Mails suchen.

## Ablauf

1. Rufe `list_recent_events` genau einmal auf. Lass die Standardwerte stehen:
   - `seit` weglassen (das sind die letzten 24 Stunden)
   - `nur_relevante`: true
   - `datenkontext`: `ECHT_GESCHAEFT`
   - `format`: `kompakt`
   - `konto` weglassen (alle Konten des Beraters)
2. Ereignisse nur aus dem Tooltext. Er ist bereits nach Wichtigkeit sortiert. Klassifikation, Kategorien und Wichtigkeit kommen vom Server. Übernimm sie, bewerte sie nicht neu.
3. Will der Berater den vollständigen Verlauf eines einzelnen Vorgangs, `get_case_events` mit `vorgang` aufrufen (Nummer oder Name). Meldungen in `list_recent_events` sind gekürzt.
4. Rufe danach `tagesueberblick_anzeigen` genau einmal auf. Es zeigt nur an, es liest und schreibt nichts. Übergib `zeitraum` (die Zeile „Seit:“ aus dem Tooltext) und alle Einträge in `eintraege`. Pro Eintrag: `block` (`rueckmeldung`, `laeuft` oder `erledigt` nach den Blöcken unten), `kunde` so wie geliefert, `bank` wenn bekannt, `titel` (was passiert ist, ein Satz), `details` mit höchstens 3 kurzen Zeilen inklusive was zu tun ist, `quelle` immer `ereignis`, `vorgang_id` (die Vorgangsnummer, wenn bekannt). `deep_link` nur setzen, wenn ein Tool einen Link geliefert hat. Innerhalb von `rueckmeldung` die dringendsten Einträge zuerst.

## Antwort

Zuerst zeigt `tagesueberblick_anzeigen` die Einträge. Danach nur 1–2 Sätze, die die Liste nicht wiederholen: was am dringendsten ist, oder dass nichts anliegt. Die Blöcke nicht als Liste ausschreiben.

Fehlt das Tool oder liefert es einen Fehler, stattdessen wie folgt als Text antworten. Drei Blöcke, in dieser Reihenfolge. Darunter die Vorgänge, pro Kunde und Vorgangsnummer ein Absatz. Ereigniszeilen bleiben im Wortlaut des Tools.

- Rückmeldung nötig: Zeilen unter WICHTIG, deren Meldung eine Antwort, eine Unterlage oder eine Frist verlangt.
- Läuft: die übrigen Zeilen unter WICHTIG und die Zeilen unter WEITERE, die die Meldung nicht als abgeschlossen ausweist.
- Erledigt: was die Meldung als Zusage, Abschluss oder erledigt ausweist.

- Höchstens etwa 10 Zeilen über alle Blöcke. Danach ein Satz, dass es weitere Ereignisse gibt und der Berater „mehr zeigen“ sagen kann. Erst dann einen zweiten Aufruf mit `format` `json` machen, und nur wenn er das verlangt.
- Steht im Tooltext, dass die Vorgangsliste abgeschnitten ist oder dass Ereignisse nicht geladen werden konnten, diesen Hinweis am Ende wörtlich wiederholen.
- Ist der Zeitraum leer, sag das in einem Satz. Schlage nichts vor, was nicht im Tooltext steht.

## Regeln

- Nur lesen. Kein Postfach öffnen, keine Mail suchen, nichts senden. Nichts in Europace anlegen oder ändern, keinen Antrag auslösen. `tagesueberblick_anzeigen` ist nur Anzeige.
- Nichts anzeigen, was nicht in einem Ereignis steht. `titel` und `details` nur daraus.
- `TEST_MODUS` nur, wenn der Berater ausdrücklich Testdaten oder den Testmodus verlangt. Sonst immer `ECHT_GESCHAEFT`.
- Nichts erfinden. Namen, Banken, Zeiten und Meldungen nur aus dem Tooltext. Ist ein Feld leer oder „—“, sag dass die Angabe fehlt.
- Kundennamen so lassen, wie der Server sie liefert.
- Kommt ein Fehlertext zurück, nicht erneut aufrufen. Sag in einem Satz, was der Berater tun kann:
  - „Zu viele Abfragen. Bitte eine Minute warten.“ → eine Minute warten, nicht selbst wiederholen.
  - „Konto nicht gefunden.“ → nach der Kontenbezeichnung fragen. Konten nennt `get_profile`.
  - „Scope cases:read fehlt.“ → die Verbindung zum Baufi Copilot neu anmelden.
  - „Dieses Tool steht nur Finanzierungsmaklern zur Verfügung.“ → das Konto hat nicht die Rolle `LOAN_BROKER`.
  - „datenkontext muss ECHT_GESCHAEFT oder TEST_MODUS sein.“ oder „format muss kompakt oder json sein.“ → das ist ein Aufruffehler. Korrigiere die Parameter einmal, dann nicht weiterprobieren.
  - „seit muss eine Dauer wie 24h oder ein Datum sein.“ → den Zeitraum beim Berater nachfragen.
