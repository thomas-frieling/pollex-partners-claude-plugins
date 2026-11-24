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
2. Rufe `get_case_events` einmal auf:
   - `vorgang`: die Nummer oder der Name, den der Berater genannt hat.
   - `seit` nur setzen, wenn der Berater einen Zeitraum nennt. Sonst weglassen (die letzten 8 Tage). Der Server kennt Stunden (`24h`, `168h`) oder ein ISO-Datum, kein `7d`. Mehr als 8 Tage sagt der Server von sich aus.
   - `datenkontext`: `ECHT_GESCHAEFT`
   - `konto` nur setzen, wenn der Berater ein Konto nennt (Schlüssel oder Bezeichnung aus `get_profile`). Sonst weglassen.
3. Kommt „Bitte Vorgangsnummer angeben.“, die Kandidaten zeigen und fragen. Nicht mischen und nicht selbst einen wählen.
4. Kein Treffer: den Satz des Tools wiedergeben, inklusive des Hinweises auf die ersten 200 Vorgänge, falls er dasteht. Nicht noch einmal mit anderen Parametern aufrufen, außer der Berater nennt einen anderen Zeitraum oder ein Konto.

## Antwort

- Den Text des Tools als Zeitleiste wiedergeben, ältestes Ereignis zuerst. Kopf (Vorgang, Kunden, Konto, Anträge) behalten.
- Meldungen ungekürzt lassen. Nichts nach Europace verweisen, um den Wortlaut zu holen.
- Offene Punkte nur nennen, wenn sie in der Meldung direkt stehen. Keine eigenen To-dos.

## Regeln

- Nichts versenden, nichts in Europace anlegen oder ändern, keine E-Mail und keinen Antrag auslösen.
- `TEST_MODUS` nur, wenn der Berater ausdrücklich Testdaten oder den Testmodus verlangt. Sonst immer `ECHT_GESCHAEFT`.
- Nichts erfinden. Nur den Text von `get_case_events`. Ist eine Angabe „—“ oder leer, sie weglassen.
- Kundennamen so lassen, wie der Server sie liefert.
- Kommt ein Fehlertext zurück, nicht erneut aufrufen. Sag in einem Satz, was der Berater tun kann:
  - „Zu viele Abfragen. Bitte eine Minute warten.“ → eine Minute warten, nicht selbst wiederholen.
  - „Konto nicht gefunden.“ → nach der Kontenbezeichnung fragen. Konten nennt `get_profile`.
  - „Scope cases:read fehlt.“ → die Verbindung zum Baufi Copilot neu anmelden.
  - „Dieses Tool steht nur Finanzierungsmaklern zur Verfügung.“ → das Konto hat nicht die Rolle `LOAN_BROKER`.
  - „datenkontext muss ECHT_GESCHAEFT oder TEST_MODUS sein.“ → das ist ein Aufruffehler. Korrigiere `datenkontext` einmal, dann nicht weiterprobieren.
  - „Vorgang angeben.“ → nach Nummer oder Namen fragen.
  - „seit muss eine Dauer wie 24h oder ein Datum sein.“ → den Zeitraum beim Berater nachfragen.
