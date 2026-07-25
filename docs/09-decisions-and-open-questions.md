# Entscheidungen und offene Fragen

Dieses Dokument hält Produkt- und Architekturentscheidungen fest. Der Status
`verbindlich` bedeutet, dass Implementierungen davon nur nach einer
dokumentierten neuen Entscheidung abweichen sollen. `Vorgeschlagen` bedeutet,
dass ein technischer Spike die Entscheidung noch bestätigen muss.

## Getroffene Entscheidungen

### D-001: Produktname Roundtable

Status: verbindlich

Das Tool heißt Roundtable. Der Name beschreibt die gemeinsame Oberfläche, an
der mehrere unterschiedliche Agenten gleichzeitig teilnehmen.

### D-002: Agent pro Session

Status: verbindlich

Agententyp, Modell und Agentenkonfiguration werden pro Session gespeichert.
Roundtable besitzt keinen global ausgewählten Agenten.

Folge:

- Claude und Codex laufen gleichzeitig.
- Mehrere Sessions desselben Agenten sind erlaubt.
- Agentenauswahl ist Bestandteil des Session-Startablaufs.

### D-003: Reply-Routing

Status: verbindlich

Eine Reply wird anhand der externen ID der beantworteten Nachricht geroutet.
Inhalt oder Gesprächston werden nicht zur Zielbestimmung interpretiert.

### D-004: Default statt aktive Session

Status: verbindlich

Es gibt eine explizite Default-Session für freie Nachrichten. Roundtable ändert
sie nicht aufgrund letzter Aktivität oder zuletzt empfangener Agentenausgabe.

Priorität:

```text
Reply > explizites Sessionziel > Default > Benutzerauswahl
```

### D-005: Inhaltstreues Routing

Status: verbindlich

Roundtable schreibt Agenten- und Benutzertexte nicht um und lässt keine KI im
kritischen Nachrichtenpfad entscheiden.

Erlaubt ist ausschließlich technische Transportaufbereitung. Raw Output bleibt
lokal verfügbar.

### D-006: Approvals 1:1

Status: verbindlich

Der originale Approval-Prompt wird weitergegeben. Benutzerantworten werden
inhaltstreu an dieselbe Session gesendet. Buttons sind deterministische
Abkürzungen für sichtbare Text- oder Tasteneingaben.

Roundtable erteilt standardmäßig keine automatische Freigabe.

### D-007: Telegram zuerst

Status: verbindlich

Die erste Oberfläche ist ein privater Telegram-Bot-Chat. Telegram Topics sind
keine Voraussetzung. WhatsApp folgt als Transportadapter.

### D-008: Transportunabhängiger Core

Status: verbindlich

Telegram-spezifische IDs werden in Transporttabellen gekapselt. Der Core
arbeitet mit internen Benutzern, Sessions, Nachrichten und Interaktionen.

### D-009: tmux ist Backend, nicht Gesamtabstraktion

Status: verbindlich

tmux ist die erste Linux-/macOS-Runtime, aber nicht Teil des
Roundtable-Domänenmodells. Windows benötigt ConPTY oder WSL.

### D-010: Plattformen

Status: verbindlich

Roundtable soll auf Linux, macOS und Windows installierbar sein und sowohl lokal
als auch auf einem VPS funktionieren.

### D-011: Lokale Persistenz

Status: vorgeschlagen

SQLite speichert Konfiguration, Mapping, Idempotenz, Status, Audit und Outbox.
Große Raw-Outputs liegen in referenzierten lokalen Dateien.

### D-012: Separate Session Hosts

Status: vorgeschlagen

Wo tmux nicht eingesetzt wird, hält ein separater Session-Host-Prozess das PTY.
So kann der Router neu starten, ohne die Agenteninstanz zwangsläufig zu beenden.

### D-013: Go als Implementierungssprache

Status: vorgeschlagen

Go bietet gute Distribution als Einzelbinary und eignet sich für Daemon,
Telegram, SQLite und plattformspezifische Dienste. Ein Spike muss insbesondere
PTY-/ConPTY-Bibliotheken, Terminalparser und Packaging bestätigen.

### D-014: Keine Cloudpflicht

Status: verbindlich

Der Kernbetrieb ist local-first. Ein späterer Cloud- oder Relay-Dienst darf
optional sein, aber Quellcode und Raw Output müssen nicht an ihn übertragen
werden.

### D-015: Keine stille Zielauswahl

Status: verbindlich

Ist das Ziel nicht eindeutig, wird nicht gesendet. Roundtable fragt nach oder
meldet den Fehler.

### D-016: Repository-Ownership

Status: verbindlich

Roundtable startet als öffentliches persönliches Open-Source-Repository unter
`rogertobler/roundtable`.

Das Repository gehört nicht zur GitHub-Organisation `inscribe-GmbH`, solange
Roundtable nicht durch eine spätere ausdrückliche Entscheidung zu einem
Firmenprodukt von InScribe wird.

Falls das Projekt wächst, kann es später in eine eigene Roundtable-Organisation
übertragen werden. Projektverlauf, Issues, Pull Requests, Stars und
Weiterleitungen sollen bei einem solchen Transfer erhalten bleiben.

## Noch offene Produktfragen

### OQ-P-001: Default-Geltungsbereich

Zu entscheiden:

- eine Default-Session pro Benutzer insgesamt,
- pro Transportidentität,
- oder pro Chat.

Aktuelle Empfehlung: pro Benutzer und Transport-/Chatkontext. Bei nur einem
privaten Telegram-Chat verhält sich dies wie eine einzelne Default-Session,
bleibt aber für spätere Kanäle korrekt.

### OQ-P-002: Bestätigung erfolgreicher freier Nachrichten

Soll jede freie Nachricht sichtbar mit dem Ziel bestätigt werden oder nur nach
Default-Wechsel und bei Unsicherheit?

Empfehlung: kompakte Bestätigung, die optional deaktiviert werden kann. Das
senkt das Risiko unbemerkter Fehlannahmen.

### OQ-P-003: Standard-Benachrichtigungsmodus

„Alles 1:1“ kann bei starkem Terminaloutput viele Nachrichten erzeugen.

Empfehlung:

- inhaltstreu alle stabilen Agentenausgaben senden,
- flüchtige Terminalframes zusammenführen,
- Raw Output lokal vollständig halten,
- pro Session abweichende Modi anbieten.

### OQ-P-004: Sessionnamen und Aliase

Zu entscheiden:

- Namen global eindeutig,
- nur pro Benutzer eindeutig,
- oder Mehrdeutigkeit mit zusätzlichem Alias.

Empfehlung: veränderbarer Anzeigename plus eindeutiger kurzer Alias pro
Benutzer.

### OQ-P-005: Löschen versus Archivieren

Empfehlung: „Stoppen“ und „Archivieren“ prominent; endgültiges Löschen nur in
erweiterten Aktionen mit Retention- und Worktree-Hinweis.

### OQ-P-006: Freie natürliche Session-Erstellung

Beispiel:

```text
Neue Claude-Session mit Modell Opus, Projekt DumbleScore, Name Import Fix
```

Zu entscheiden ist, ob dies:

- regelbasiert geparst,
- mit einem lokalen/externen LLM interpretiert,
- oder zunächst nur als Komfortsyntax mit anschließender Bestätigung angeboten
  wird.

Empfehlung: Menüs zuerst. Natürliche Eingabe später immer in einen sichtbaren
Sessionentwurf umwandeln und vor Start bestätigen.

## Noch offene technische Fragen

### OQ-T-001: Go-PTY-Bibliotheken

Im Spike prüfen:

- Unix PTY,
- ConPTY,
- Terminalscreen-Emulation,
- UTF-8 und Windows-Codepages,
- Resize und Signalverhalten,
- Cross-Compilation.

### OQ-T-002: tmux Output Capture

Festzulegen:

- `pipe-pane`-Logformat,
- Startzeitpunkt,
- Rotation,
- Cursorpersistenz,
- Verhalten bei lokalem Attach,
- Rekonstruktion stabiler Ausgaben.

### OQ-T-003: Telegram Update-Modus

Polling ist für lokale Installationen einfach. Webhooks reduzieren Latenz, sind
auf lokalen Rechnern aber aufwendiger. Empfehlung für die erste Version:
Long Polling; Webhook optional für Serverbetrieb.

### OQ-T-004: Agentenereignisse

Für jede unterstützte Claude-/Codex-Version prüfen:

- verfügbare Hooks oder strukturierte Events,
- Stabilität der TUI,
- Approval-Optionen,
- Resume-Verhalten,
- Modellermittlung,
- Exit- und Abschlusssemantik.

Der Adapter muss bei unbekannter Version sicher degradieren.

### OQ-T-005: Terminalparser

Zu entscheiden ist, welche etablierte Terminalemulation Raw Streams in einen
Screen Snapshot umwandelt. Ein eigener ANSI-Parser sollte vermieden werden,
wenn eine gepflegte Bibliothek die nötigen Plattformen unterstützt.

### OQ-T-006: Verschlüsselung lokaler Daten

Zu klären:

- Bedrohungsmodell für Datenbank und Raw Logs,
- Betriebssystem-Keychain für Schlüssel,
- Vollverschlüsselung oder selektive Felder,
- Backup und Rotation.

Dateirechte sind Mindestanforderung; Verschlüsselung kann für sensible
Installationen erforderlich sein.

### OQ-T-007: Session-Host-Protokoll

Zu definieren:

- lokale Authentifizierung,
- Protokollversion,
- Output-Cursor,
- Heartbeat,
- Eingabe-Idempotenz,
- Upgrade-Kompatibilität,
- Stop- und Adopt-Semantik.

### OQ-T-008: Remote Nodes

Die Kerndomäne enthält bereits `Node`, aber Netzwerkprotokoll und Trust-Modell
werden erst nach stabilem Einzelknotenbetrieb definiert.

### OQ-T-009: WhatsApp-Anbindung

Vor Implementierung prüfen:

- offizieller API-Zugang,
- Kosten und Hostinganforderungen,
- Reply-Metadaten,
- Button- und Dateifähigkeiten,
- zulässige Nachrichtenfenster,
- Self-hosted Alternativen und deren Kontorisiken.

Der Transportadapter muss Capability-Unterschiede sichtbar machen.

## Entscheidungsprozess

Neue wesentliche Entscheidungen erhalten:

```text
ID
Titel
Status
Datum
Kontext
Entscheidung
Alternativen
Konsequenzen
```

Änderungen an verbindlichen Grundprinzipien müssen zusätzlich in
`README.md`, Anforderungen und betroffenen Architekturkapiteln aktualisiert
werden.

## Erste technische Entscheidungen für den Spike

Vor Beginn der Produktimplementierung sollten in dieser Reihenfolge entschieden
werden:

1. Sprache und Projektstruktur,
2. Telegram-Bibliothek,
3. SQLite-Treiber und Migrationen,
4. tmux-Befehls- und Outputstrategie,
5. Terminalparser,
6. interner Adaptervertrag,
7. Dienst- und Datenpfade,
8. Secret-Store-Fallback,
9. Testfixtures für Claude und Codex,
10. Packaging des ersten Linux-Builds.
