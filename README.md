# Pollex Partners — Claude-Plugins

Marktplatz für Claude. Er enthält das Plugin **baufi-copilot**: den Connector zum Baufi Copilot und Skills für typische Fragen eines Finanzierungsmaklers.

## Marktplatz hinzufügen

In Claude: **Einstellungen → Plugins → Marktplatz hinzufügen**, dann dieses Repository angeben:

`thomas-frieling/pollex-partners-claude-plugins`

Danach das Plugin **baufi-copilot** installieren. Claude fragt einmal nach der Anmeldung beim Baufi Copilot (dasselbe Konto wie in der App). Updates kommen von selbst, sobald eine neue Version hier liegt.

## Was du fragen kannst

- Was ist heute passiert? Was gibt's Neues? Morgenüberblick.
- Was ist bei Vorgang AB1234 diese Woche passiert? Was war bei Familie Müller?
- Welche Europace-Konten habe ich? Warum sehe ich keine Ereignisse?

Claude liest nur. Es verschickt nichts und ändert nichts in Europace. Testdaten (`TEST_MODUS`) holt es nur, wenn du ausdrücklich danach fragst.
