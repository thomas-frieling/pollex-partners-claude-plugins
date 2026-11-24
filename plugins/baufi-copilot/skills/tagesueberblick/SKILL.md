---
name: tagesueberblick
description: >
  Tages- und Morgenüberblick über die letzten Vorgangsereignisse des angemeldeten
  Finanzierungsmaklers, plus passende Mails, wenn in Claude ein Mail-Connector
  verbunden ist. Greift bei Formulierungen wie „Was ist heute passiert?“,
  „Was gibt's Neues?“, „Was ist seit gestern passiert?“, „Morgenüberblick“,
  „Was muss ich heute wissen?“. Nicht für einen einzelnen Vorgang oder eine einzelne Familie
  (dafür vorgang-verlauf) und nicht für Konten oder Anmeldung (dafür zugang-und-konten).
---

# Tagesüberblick

Aufgabe: die letzten Ereignisse über alle Konten des angemeldeten Beraters kurz zusammenfassen und, wenn ein Postfach verbunden ist, die passenden Mails desselben Zeitraums dazu legen.

## Ablauf

1. Rufe `list_recent_events` genau einmal auf. Lass die Standardwerte stehen:
   - `seit` weglassen (das sind die letzten 24 Stunden)
   - `nur_relevante`: true
   - `datenkontext`: `ECHT_GESCHAEFT`
   - `format`: `kompakt`
   - `konto` weglassen (alle Konten des Beraters)
2. Ereignisse nur aus dem Tooltext. Er ist bereits nach Wichtigkeit sortiert. Klassifikation, Kategorien und Wichtigkeit kommen vom Server. Übernimm sie, bewerte sie nicht neu.
3. Mails nur, wenn in Claude ein Mail-Connector verbunden ist (zum Beispiel Microsoft 365 oder Gmail). Dann im Postfach lesen, im selben Zeitraum wie die Zeile „Seit:“ im Tooltext. Suche nach Mails zu Finanzierung: Banken, Kunden, Unterlagen, Rückfragen, Zusagen, Fristen. Newsletter, Werbung und Mails ohne diesen Bezug weglassen.
4. Ist kein Mail-Connector verbunden, keine Mail-Suche starten. Den Mail-Teil weglassen und in einem Satz sagen, dass Mails nicht geprüft wurden.
5. Ordne jede gefundene Mail einem Vorgang aus dem Tooltext zu. Es zählen Kundenname, Inhalt von Betreff und Text, Vorgangsnummer und Antragsnummer. Die Absenderadresse allein reicht nicht. Passt nichts, die Mail keinem Vorgang zuschreiben.
6. Will der Berater den vollständigen Verlauf eines einzelnen Vorgangs, `get_case_events` mit `vorgang` aufrufen (Nummer oder Name). Meldungen in `list_recent_events` sind gekürzt.

## Antwort

Drei Blöcke, in dieser Reihenfolge. Darunter die Vorgänge, pro Kunde und Vorgangsnummer ein Absatz. Ereigniszeilen bleiben im Wortlaut des Tools, Mails als kurzer Zusatz (Absender, Betreff, was verlangt oder zugesagt wird).

- Rückmeldung nötig: Zeilen unter WICHTIG, deren Meldung eine Antwort, eine Unterlage oder eine Frist verlangt, plus Mails mit Rückfrage, fehlender Unterlage oder Frist.
- Läuft: die übrigen Zeilen unter WICHTIG und die Zeilen unter WEITERE, die die Meldung nicht als abgeschlossen ausweist, plus Mails, die den Vorgang nur fortsetzen.
- Erledigt: was Meldung oder Mail als Zusage, Abschluss oder erledigt ausweist.

Mails ohne passenden Vorgang stehen im passenden Block ganz unten, jede als eine Zeile, ohne erfundenen Vorgang.

- Höchstens etwa 10 Zeilen über alle Blöcke. Danach ein Satz, dass es weitere Ereignisse gibt und der Berater „mehr zeigen“ sagen kann. Erst dann einen zweiten Aufruf mit `format` `json` machen, und nur wenn er das verlangt.
- Steht im Tooltext, dass die Vorgangsliste abgeschnitten ist oder dass Ereignisse nicht geladen werden konnten, diesen Hinweis am Ende wörtlich wiederholen.
- Ist der Zeitraum leer und gibt es keine passende Mail, sag das in einem Satz. Schlage nichts vor, was nicht im Tooltext oder in einer gelesenen Mail steht.
- War ein Connector verbunden und die Suche lieferte nichts Passendes, ein Satz: keine passenden Mails in diesem Zeitraum.

## Regeln

- Nur lesen. Keine Mail senden, antworten, weiterleiten, verschieben, löschen oder als gelesen markieren. Nichts in Europace anlegen oder ändern, keinen Antrag auslösen.
- `TEST_MODUS` nur, wenn der Berater ausdrücklich Testdaten oder den Testmodus verlangt. Sonst immer `ECHT_GESCHAEFT`.
- Nichts erfinden. Namen, Banken, Zeiten und Meldungen nur aus dem Tooltext, Mail-Angaben nur aus der gelesenen Mail. Ist ein Feld leer oder „—“, sag dass die Angabe fehlt.
- Kundennamen so lassen, wie der Server sie liefert.
- Kommt ein Fehlertext zurück, nicht erneut aufrufen. Sag in einem Satz, was der Berater tun kann:
  - „Zu viele Abfragen. Bitte eine Minute warten.“ → eine Minute warten, nicht selbst wiederholen.
  - „Konto nicht gefunden.“ → nach der Kontenbezeichnung fragen. Konten nennt `get_profile`.
  - „Scope cases:read fehlt.“ → die Verbindung zum Baufi Copilot neu anmelden.
  - „Dieses Tool steht nur Finanzierungsmaklern zur Verfügung.“ → das Konto hat nicht die Rolle `LOAN_BROKER`.
  - „datenkontext muss ECHT_GESCHAEFT oder TEST_MODUS sein.“ oder „format muss kompakt oder json sein.“ → das ist ein Aufruffehler. Korrigiere die Parameter einmal, dann nicht weiterprobieren.
  - „seit muss eine Dauer wie 24h oder ein Datum sein.“ → den Zeitraum beim Berater nachfragen.
