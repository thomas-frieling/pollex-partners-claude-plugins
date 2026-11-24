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

Ursache anhand der Profilfelder, in dieser Reihenfolge:

- `status` ist nicht aktiv, oder `legal` ist unvollständig → Zugang in der App ist nicht fertig. Claude kann das nicht nachholen.
- `roles` enthält nicht `LOAN_BROKER` → Ereignisse gibt es nur für Finanzierungsmakler. Das Profil selbst bleibt sichtbar.
- `ep2ApiAccounts` ist leer → kein Europace-Konto hinterlegt. Nenne keine erfundenen Konten.
- Sonst die Konten aufzählen. Pro Eintrag nur `label` und `key`. `rootId`, `partnerId` und `clientType` nur nennen, wenn der Berater danach fragt.

## Antwort

- Zuerst der Name (`name`) und die E-Mail (`email`), dann der Status, dann die Kontenliste.
- Kurz: höchstens ein kurzer Absatz plus eine Liste. Kein Rollen-Vortrag, wenn der Berater nur die Konten wollte.
- Fehlt `billingAccountName`, den Abrechnungskonten-Namen weglassen.

## Regeln

- Nichts versenden, nichts in Europace anlegen oder ändern, keine E-Mail und keinen Antrag auslösen.
- `TEST_MODUS` nur, wenn der Berater ausdrücklich Testdaten oder den Testmodus verlangt. Sonst immer `ECHT_GESCHAEFT`. Konten aus dem Profil gelten für beide Datenkontexte. Erfinde keine Testdaten.
- Nichts erfinden. Nur Felder aus `get_profile`: `name`, `email`, `roles`, `status`, `legal`, `billingAccountName`, `ep2ApiAccounts`. Ist ein Feld null oder leer, sag dass die Angabe fehlt.
- Kundennamen und Kontenbezeichnungen so lassen, wie der Server sie liefert.
- Kommt „Nicht angemeldet.“ oder ein Verbindungsfehler, nicht erneut aufrufen. Sag, dass der Berater die Verbindung zum Baufi Copilot in Claude neu anmelden muss.
- Kommt bei einem anderen Skill „Scope ticker:read fehlt.“, „Zu viele Abfragen. Bitte eine Minute warten.“, „Konto nicht gefunden.“ oder „Dieses Tool steht nur Finanzierungsmaklern zur Verfügung.“, dieselbe Handlungsempfehlung geben und den Aufruf nicht wiederholen. `format` `kompakt` oder `json` und der Parameter `seit` gehören zu `list_recent_events`, nicht zu diesem Skill.
