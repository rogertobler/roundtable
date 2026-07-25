# Roundtable: Kurzbeschreibung für externes AI-Review

## Produkt in einem Satz

Roundtable ist ein lokaler Multi-Agent-Chat-Router: Ein gemeinsamer
Messenger-Chat tunnelt Nachrichten zu mehreren gleichzeitig laufenden nativen
AI-CLIs; Replies gehen an die Ursprungssession, freie Nachrichten an die
Default-Session.

## Ausgangslage

Ein Entwickler betreibt häufig mehrere Coding-Agenten gleichzeitig:

- Claude Code implementiert eine Funktion.
- Codex überprüft einen anderen Branch.
- Eine weitere Claude-Session arbeitet in einem zweiten Projekt.
- Eine Session wartet auf eine Freigabe.

Jeder Agent läuft als echte interaktive CLI auf dem lokalen Rechner oder einem
privaten Server. Roundtable macht einen privaten Telegram-Bot-Chat zur
gemeinsamen Inbox und Steuerungsoberfläche für alle diese Sessions. Später soll
derselbe Router auch WhatsApp und weitere Messenger bedienen.

Roundtable ist kein eigener KI-Agent, kein Agenten-SDK und kein
Terminalprodukt. Es ist ausschließlich Chat-Router, Sessionverwaltung und
bidirektionaler Tunnel.

## Kernmodell

Jede Session besitzt:

- eindeutige ID und Namen,
- Agent, zum Beispiel Claude Code oder Codex,
- Modell,
- Projekt und Arbeitsverzeichnis,
- optionalen Git-Worktree,
- Berechtigungsprofil,
- Benachrichtigungsmodus,
- echte dauerhafte CLI-Session.

In der ersten Implementierung wird beim Sessionstart im Hintergrund eine
detached tmux-Session angelegt. Darin startet Roundtable die bereits lokal
installierte und authentifizierte Agenten-CLI.

```text
Session A -> tmux -> Claude Code
Session B -> tmux -> Codex
Session C -> tmux -> Claude Code
```

Alle Sessions können gleichzeitig laufen.

## Gemeinsamer Chat

Alle abonnierten Sessions schreiben in denselben privaten Telegram-Chat:

```text
Claude · Backend Feature
Soll ich die Migration jetzt ausführen?

Codex · Backend Review
Ich habe zwei mögliche Fehler gefunden.

Claude · Website
Die mobile Navigation ist vorbereitet.
```

Die Herkunft jeder Nachricht wird dauerhaft als
`externe_message_id -> session_id` gespeichert.

Eine Session benötigt keinen eigenen Chat und kein Telegram Forum Topic.

## Routing

### Reply

Antwortet der Benutzer auf eine Agentennachricht, wird die Antwort exakt an die
tmux-Session dieser Nachricht gesendet:

```text
Reply auf Claude-Nachricht A -> tmux-Session A
Reply auf Codex-Nachricht B  -> tmux-Session B
```

Die Default-Session spielt bei einer Reply keine Rolle.

### Freie Nachricht

Schreibt der Benutzer ohne Reply und ohne explizites Sessionziel, geht die
Nachricht an die Default-Session.

```text
Nachricht ohne Referenz -> Default-Session
```

Die erste erfolgreich erstellte Session wird automatisch Default, solange noch
kein Default existiert. Später erstellte Sessions ändern ihn nicht. Der
Benutzer kann den Default manuell wechseln.

Kann kein eindeutiges Ziel bestimmt werden, sendet Roundtable nichts und zeigt
eine Sessionauswahl.

## Tunnelprinzip

Roundtable führt keinen eigenen Agenten-Chat.

```text
Telegram/WhatsApp
      ↓ Benutzernachricht
Roundtable Reply Router
      ↓ literal Text + Taste
tmux
      ↓
native Claude-/Codex-CLI

native CLI-Ausgabe
      ↓
tmux Output Capture
      ↓
Roundtable
      ↓
Telegram/WhatsApp
```

Jede Messengereingabe muss im echten nativen CLI-Verlauf sichtbar sein. Jede
Agentenantwort im Messenger muss aus der Ausgabe dieser CLI stammen.

Öffnet der Benutzer später:

```text
tmux attach-session -t <session>
```

sieht er denselben laufenden Claude-/Codex-Chat und kann direkt dort
weiterschreiben. Lokale Eingaben und Messengereingaben erreichen denselben
Agentenprozess und denselben Kontext.

## Keine parallelen Agentenprotokolle

Roundtable darf Agenten-Hooks oder strukturierte Ereignisse optional als
Sensoren verwenden:

- Session gestartet,
- Turn abgeschlossen,
- Approval wartet,
- Agent beendet.

Sie dürfen jedoch:

- keinen zweiten Gesprächskontext erzeugen,
- keine Benutzernachricht an tmux vorbeiführen,
- keine Antwort außerhalb der nativen CLI beziehen,
- keine Freigabe automatisch entscheiden.

Fehlen Hooks, funktioniert der Tunnel weiter. Status- und Prompt-Erkennung kann
dann weniger präzise sein.

## Session-Erstellung

Der Benutzer klickt sich im Messenger durch:

1. Agent auswählen.
2. Modell auswählen.
3. Projekt auswählen.
4. Projektverzeichnis oder Worktree auswählen.
5. Sessionname festlegen.
6. Berechtigungsprofil auswählen.
7. Optional eine Startaufgabe eingeben.
8. Zusammenfassung bestätigen.

Roundtable prüft:

- tmux ist installiert,
- ausgewählte CLI ist installiert,
- CLI ist lokal authentifiziert,
- Projektpfad ist erlaubt,
- Modell- und Startargumente sind gültig.

Danach werden tmux-Session, Output Capture und Agenten-CLI als direkter
Pane-Prozess ohne Shell-Fallback gestartet. Roundtable setzt interne
Session-/Runtime-Marker als tmux User Options und feste Terminalmaße.

## Inhaltstreue Ein- und Ausgabe

Messengertext wird literal in die CLI geschrieben, beispielsweise über einen
tmux-Buffer mit anschließendem Paste und separatem Enter. Roundtable darf
Messengertext niemals als Shellcode interpretieren.

Vor jedem Write prüft Roundtable Runtime-Marker, Pane-Zustand,
Vordergrundprozess und normale interaktive tmux-Clients. Bracketed Paste wird
nur verwendet, wenn die Anwendung diesen Modus aktiviert hat. Multiline wird
pro Agentenversion getestet und bei unbekannter Semantik nicht automatisch
ausgeführt.

Eine mobile Übernahme detached normale lokale Clients erst nach ausdrücklicher
Bestätigung und prüft vor dem Write erneut.

Die CLI-Ausgabe wird inhaltstreu weitergeleitet. Erlaubte technische
Aufbereitung:

- ANSI- und Cursorsteuerung verarbeiten,
- Spinner stabilisieren,
- überschreibende Zeilen zusammenführen,
- schnelle Outputstücke bündeln,
- lange Inhalte teilen oder als Datei senden,
- bekannte Secrets nach Best Effort maskieren.

Roundtable verwendet kein LLM zum Umschreiben oder Zusammenfassen des
Nachrichtenpfads.

Für die Erfassung sind vorgesehen:

- `tmux pipe-pane` oder tmux Control Mode als im Spike zu vergleichender
  fortlaufender Raw Stream,
- `tmux capture-pane` für den aktuellen Screen State,
- lokale append-only Logs mit Output-Cursor.

Control Mode liefert strukturierte tmux-Ereignisse, aber auch dort bleibt
`%output` roher Terminaloutput ohne Agentenantwortgrenzen.

## Rückfragen und Freigaben

Approvals bleiben echte CLI-Interaktionen:

1. Roundtable erkennt im Output einen Prompt.
2. Der unveränderte Prompt wird in den Messenger gesendet.
3. Eine Reply geht an dieselbe tmux-Session.
4. Freitext oder bekannte Tastenfolge wird zurückgesendet.

Buttons dürfen nur angeboten werden, wenn ihre konkrete Terminalwirkung
bekannt ist. Vor dem Senden prüft Roundtable, ob der CLI-Prompt noch aktuell
ist. Alte Buttons dürfen keine zweite Eingabe auslösen.

## Sessionaktionen

- Nachricht senden,
- als Default setzen,
- abonnieren oder stummschalten,
- Status aktualisieren,
- optionalen CLI-Snapshot anzeigen,
- Raw Output und Verlauf anzeigen,
- Enter, Escape, Tab, Pfeiltasten und `Ctrl+C` senden,
- Session unterbrechen, fortsetzen oder stoppen,
- lokalen tmux-Attach-Befehl anzeigen,
- Session umbenennen oder archivieren.

CLI-Snapshots sind Diagnosefunktionen. Der normale Arbeitsfluss bleibt der
gemeinsame Messenger-Chat.

## Plattformen

### Linux

Roundtable und Agenten laufen direkt in tmux. Der Core läuft als
`systemd --user` Dienst.

### macOS

Roundtable verwendet ebenfalls tmux und läuft als `launchd` User Agent.

### Windows

Die erste Unterstützung verwendet WSL mit Roundtable, tmux und Agenten-CLIs in
derselben Distribution.

Später kann ein nativer ConPTY Session Host tmux ersetzen. Er muss dieselbe
Tunnel-Semantik liefern: dauerhafte CLI, Raw Output, literal Eingabe und lokal
attachbarer identischer Verlauf.

## Architektur

```text
Transport Adapter
  Telegram
  später WhatsApp

Message Router
  Reply Mapping
  explizites Sessionziel
  Default-Session

Session Tunnel
  Input Queue
  Output Collector
  tmux Backend

Agent Definition
  CLI-Erkennung
  Startbefehl
  Modellargument
  optionale Output-/Hook-Hinweise
```

SQLite speichert Routing, Default, Sessions, Input-Idempotenz, Output-Cursor,
Abonnements und Audit. Die Datenbank ist nicht der maßgebliche
Agentengesprächskontext.

Pane-Writes besitzen persistierte Zwischenzustände. Ist nach einem Absturz
unklar, ob Text oder Enter angekommen sind, wird `delivery_uncertain`
angezeigt und niemals automatisch erneut gesendet.

## Sicherheit

- nur freigegebene Messenger-Benutzer,
- Pairing über lokalen Einmalcode,
- standardmäßig privater Bot-Chat,
- kanonische Projekt- und Pfad-Allowlist,
- Betrieb ohne Root-/Administratorrechte,
- literal Eingaben ohne Shell-Interpolation,
- direkter Agentenprozess ohne Shell-Fallback,
- kein gleichzeitiges lokales und mobiles Schreiben in dasselbe Pane,
- Reattach nur mit übereinstimmenden tmux-IDs und Roundtable-Markern,
- sichere Tokenablage,
- Prompt- und Runtime-Prüfung vor Approval-Buttons,
- keine geratenen Sessionziele,
- agenteneigene Sandbox- und Approvalmodi,
- optional OS-/Container-Sandbox für starke Schreibgrenzen,
- vollständiges Audit kritischer Aktionen.

## Benachrichtigungen und Backpressure

Pro Session:

- alle stabilen Ausgaben,
- nur Rückfragen,
- nur Freigaben,
- nur Abschluss,
- nur Fehler,
- stumm.

Telegram-Ausgaben werden pro Chat begrenzt, gebündelt und bei Bedarf als Datei
gesendet. Ein Outputstau darf die native CLI nicht blockieren.

## Abgrenzung zu ccgram

ccgram ist der engste bekannte Wettbewerber und tunnelt Telegram Forum Topics
zu tmux/herdr-Sessions.

Roundtables zentrale Differenzierung:

```text
ccgram:
  ein Telegram Topic = eine Session

Roundtable:
  ein privater Chat = gemeinsame Inbox
  Reply = Ursprungssession
  freie Nachricht = Default-Session
```

Weitere Ziele sind mehrere Transportkanäle, ein formales Projektmodell und
Windows-Unterstützung.

## Nicht verhandelbare Regeln

1. Roundtable ist ein Multi-Agent-Chat-Router.
2. Agent und Modell gehören zur Session.
3. Jede Session ist eine echte native CLI.
4. Messenger und lokales Terminal bedienen denselben Agentenprozess.
   Sie schreiben kontrolliert abwechselnd, nicht gleichzeitig.
5. Replies gehen an die tmux-Session der beantworteten Nachricht.
6. Freie Nachrichten gehen an die Default-Session.
7. Die erste Session wird initialer Default; spätere ändern ihn nicht.
8. Roundtable führt keinen zweiten Agentenkontext.
9. Inhalte und Approvals werden nicht umformuliert oder selbst entschieden.
10. Hooks sind nur optionale Sensoren.
11. Bei unklarer Zuordnung wird nicht geraten.
12. Telegram ist der erste, aber nicht der einzige geplante Transport.
13. Unklare Teilzustellungen werden nicht automatisch wiederholt.

## Fragen an den Reviewer

Bitte analysiere kritisch:

1. Ist ein zuverlässiger inhaltstreuer Tunnel über tmux für Claude Code und
   Codex praktisch umsetzbar?
2. Welcher gemessene Collector-Mix aus `pipe-pane`, tmux Control Mode,
   `capture-pane`, Raw Cursor und Screen State ist robust?
3. Wie erkennt man stabile neue Agentenausgabe, ohne einen zweiten
   Agentenkanal einzuführen?
4. Wie lassen sich Texte inklusive Sonderzeichen und Mehrzeiligkeit sicher
   literal in tmux einfügen?
5. Welche Tests und UX braucht die festgelegte Schreibsperre zwischen lokaler
   und mobiler Bedienung?
6. Wie sollte Output-Backpressure für einen einzigen Telegram-Chat aussehen?
7. Welche Teile von ccgram wären sinnvoll wiederverwendbar, ohne dessen
   Topic-pro-Session-Modell zu übernehmen?
8. Welche Sicherheitsrisiken des Tunnelmodells fehlen?
9. Welche Funktionen gehören zwingend in den ersten Linux-MVP?
10. Ist Go für Core, Telegram, SQLite und tmux-Steuerung die richtige Wahl?
11. Wie sollte der Windows-WSL-Installationsweg gestaltet werden?
12. Welche Tests beweisen, dass Messenger und lokales Terminal wirklich
    denselben nativen Agentenkontext sehen?

Bitte priorisiere Antworten als:

- kritisch vor Implementierungsbeginn,
- wichtig für den ersten MVP,
- später sinnvoll.
