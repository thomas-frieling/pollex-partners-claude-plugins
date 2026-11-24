---
name: zugang-und-konten
description: >
  Profil, Europace-Konten und die Erklärung, warum keine Ereignisse ankommen.
  Greift bei Formulierungen wie „Welche Europace-Konten habe ich?“, „Bin ich angemeldet?“,
  „Warum sehe ich keine Ereignisse?“, „Warum funktioniert der Tagesüberblick nicht?“,
  „Welches Konto ist hinterlegt?“. Nicht für den Inhalt der Ereignisse
  (dafür tagesueberblick oder vorgang-verlauf).
---

# Zugang und Konten

Aufgabe: erklären, mit welchem Konto Claude spricht und warum ein Ereignis-Tool nichts liefern kann.

## Ablauf

1. Rufe `get_profile` einmal auf. Keine Parameter.
2. Für „Warum sehe ich keine Ereignisse?“ zusätzlich nicht selbst `list_recent_events` aufrufen, wenn der Berater nur die Ursache wissen will. Lies die Felder unten und ordne die Ursache zu. Nur wenn er danach die Ereignisse sehen will, gilt der Skill tagesueberblick.

Ursache anhand des Profiltexts, in dieser Reihenfolge. Kommt das Tool mit `isError` zurück, diesen Text wiedergeben und nicht erneut aufrufen:

- „Kein Baufi-Copilot-Konto für dieses Login.“ → kein Konto für dieses Login.
- „Konto ist nicht freigegeben.“ → Freigabe in der App fehlt.
- „Rechtliche Zustimmung ist unvollständig.“ → in der Baufi-Copilot-App zustimmen. Fehlende Dokumente aus dem Text nennen.
- `status` ist nicht aktiv, oder die rechtliche Zustimmung ist unvollständig → Zugang in der App ist nicht fertig. Claude kann das nicht nachholen.
- `Rollen` enthält nicht `LOAN_BROKER` → Ereignisse gibt es nur für Finanzierungsmakler. Das Profil selbst bleibt sichtbar.
- `Europace-Konten: keine` → kein Europace-Konto hinterlegt. Nenne keine erfundenen Konten.
- Sonst die Konten aus den Zeilen „Schlüssel“ aufzählen. Nur Bezeichnung und Schlüssel.

## Antwort

- Zuerst der Name und die E-Mail, dann der Status, dann die Kontenliste.
- Kurz: höchstens ein kurzer Absatz plus eine Liste. Kein Rollen-Vortrag, wenn der Berater nur die Konten wollte.
- Fehlt die Zeile `Abrechnungskonto`, den Abrechnungskonten-Namen weglassen.

## Regeln

- Nichts versenden, nichts in Europace anlegen oder ändern, keine E-Mail und keinen Antrag auslösen.
- `TEST_MODUS` nur, wenn der Berater ausdrücklich Testdaten oder den Testmodus verlangt. Sonst immer `ECHT_GESCHAEFT`. Konten aus dem Profil gelten für beide Datenkontexte. Erfinde keine Testdaten.
- Nichts erfinden. Nur den Text von `get_profile`: Name, E-Mail, Status, Rollen, rechtliche Zustimmung, Abrechnungskonto, Europace-Konten mit Schlüssel. Ist eine Angabe leer, sag dass sie fehlt.
- Kundennamen und Kontenbezeichnungen so lassen, wie der Server sie liefert.
- Kommt „Nicht angemeldet.“ oder ein Verbindungsfehler, nicht erneut aufrufen. Sag, dass der Berater die Verbindung zum Baufi Copilot in Claude neu anmelden muss.
- Kommt bei einem anderen Skill „Scope cases:read fehlt.“, „Zu viele Abfragen. Bitte eine Minute warten.“, „Konto nicht gefunden.“ oder „Dieses Tool steht nur Finanzierungsmaklern zur Verfügung.“, dieselbe Handlungsempfehlung geben und den Aufruf nicht wiederholen.
