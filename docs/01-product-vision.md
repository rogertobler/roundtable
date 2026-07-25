# Produktvision

## Ausgangslage

Coding-Agenten wie Claude Code und Codex arbeiten am wirkungsvollsten in echten,
interaktiven Terminal-Sessions. Dort besitzen sie ihren laufenden Kontext, sehen
das Projekt, führen Befehle aus und warten bei Bedarf auf Rückfragen oder
Freigaben.

In der Praxis entstehen schnell mehrere parallele Sessions:

- Ein Claude-Agent implementiert eine Funktion.
- Ein Codex-Agent prüft denselben oder einen anderen Branch.
- Eine weitere Session untersucht einen Produktionsfehler.
- Eine Session wartet auf eine Entscheidung.
- Eine andere führt gerade eine lange Test-Suite aus.

Ohne Roundtable muss der Benutzer regelmäßig verschiedene Terminals,
SSH-Verbindungen oder Rechner kontrollieren. Mobile Nutzung ist umständlich,
und bei vielen parallelen Sessions ist leicht unklar, zu welcher Session eine
Frage gehört.

## Zielbild

Roundtable macht aus einem privaten Messaging-Chat eine gemeinsame Inbox und
Fernsteuerung für alle Agent-Sessions eines Benutzers.

Der Benutzer kann unterwegs eine neue Session anlegen, Agent, Modell, Projekt
und Namen auswählen und anschließend über denselben Chat mit mehreren Sessions
gleichzeitig arbeiten. Jede Agentennachricht kennt ihre Ursprungssession. Eine
Telegram-Reply wird automatisch an diese Session zurückgesendet.

Freie Nachrichten ohne Reply werden an eine ausdrücklich festgelegte
Default-Session gesendet. Die Default-Session ist eine Komfortfunktion und
überschreibt niemals die Zuordnung einer Reply.

## Produktversprechen

Roundtable soll folgende Erfahrung ermöglichen:

1. Auf Linux, macOS oder Windows installieren.
2. Telegram-Bot sicher mit dem eigenen Benutzer verbinden.
3. Lokale Projekte freigeben.
4. In Telegram eine neue Claude- oder Codex-Session zusammenklicken.
5. Mehrere Sessions gleichzeitig arbeiten lassen.
6. Fragen und Ergebnisse aller abonnierten Sessions in einer Inbox empfangen.
7. Durch einfaches Antworten immer die richtige Session erreichen.
8. Den Rechner später öffnen und dieselben Sessions im echten Terminal
   weiterverwenden.

## Zielgruppen

### Einzelentwickler

Entwickler, die mehrere lokale oder auf einem VPS laufende Coding-Agenten
parallel nutzen und diese auch vom Mobiltelefon aus steuern wollen.

### Technische Betreiber

Benutzer, die Agenten für Analysen, Tests, Wartungsaufgaben oder kontrollierte
Operationen auf einem eigenen Server einsetzen.

### Kleine Teams, später

Teams, die Sessions, Rollen, Freigaben und Projekte gemeinsam verwalten. Dieser
Mehrbenutzermodus ist nicht Voraussetzung für die erste Version, beeinflusst
aber die Mandanten- und Berechtigungsgrenzen der Architektur.

## Kernanwendungsfälle

### Parallele Implementierung und Review

Claude implementiert eine Funktion in einem Worktree. Codex prüft parallel
einen Diff oder arbeitet in einem zweiten Worktree. Nachrichten beider Agenten
erscheinen im selben Chat und bleiben über Replies getrennt.

### Mobiles Fortsetzen einer lokalen Aufgabe

Eine Session wird am Laptop gestartet. Der Benutzer verlässt den Arbeitsplatz,
erhält später eine Rückfrage in Telegram und beantwortet sie dort. Nach der
Rückkehr kann er die Session wieder direkt im Terminal öffnen.

### Betrieb auf einem VPS

Roundtable und die Agenten laufen dauerhaft auf einem privaten Server. Telegram
ist die mobile Bedienoberfläche. Ein Neustart des Roundtable-Routers darf die
laufenden Session Hosts nicht unnötig beenden.

### Schneller Session-Start

Der Benutzer wählt im Telegram-Menü:

```text
Agent: Claude Code
Modell: Opus
Projekt: DumbleScore
Arbeitsmodus: Neuer Worktree
Name: Import Fix
Berechtigungsprofil: Standard
```

Roundtable validiert die Auswahl, richtet Arbeitsverzeichnis und Runtime ein,
startet den Agenten und bestätigt die neue Session.

### Freigaben unterwegs

Ein Agent zeigt eine originale Freigabeanfrage. Roundtable leitet sie
inhaltstreu weiter. Der Benutzer antwortet mit Text oder verwendet eine
Schaltfläche, deren Wirkung als konkrete Terminaleingabe sichtbar und
deterministisch ist.

## Abgrenzung

Roundtable ist nicht:

- ein eigener LLM- oder Coding-Agent,
- eine autonome Entscheidungsinstanz,
- eine zentrale Cloud, die Quellcode besitzen muss,
- ein vollständiger grafischer Terminal-Emulator in Telegram,
- ein Ersatz für Git, Worktrees oder bestehende Agent-CLIs,
- eine Garantie, dass jede unbekannte Terminal-TUI ohne Adapter perfekt
  fernbedienbar ist.

Roundtable kann Workflows koordinieren, darf aber Agentenausgaben nicht
eigenmächtig beantworten oder Freigaben erteilen, sofern der Benutzer dies
nicht später über eine ausdrücklich konfigurierte Regel erlaubt.

## Produktprinzipien

### Session-zentriert

Die Session ist die zentrale Einheit. Agent, Modell, Projekt, Runtime,
Arbeitsverzeichnis, Berechtigungsprofil, Abonnement und Verlauf gehören zu ihr.

### Transportunabhängig

Telegram ist der erste Transport, aber nicht Bestandteil der Kerndomäne.
WhatsApp, Discord, Slack, Web oder eine native App sollen später dieselben
Sessions ansprechen können.

### Agentenunabhängig

Claude Code und Codex erhalten Agent-Adapter. Andere interaktive Programme
können später über denselben Vertrag eingebunden werden.

### Local-first

Roundtable läuft beim Benutzer oder auf dessen Server. Die lokale Datenbank ist
die maßgebliche Quelle für Sessions, Routing und Auditdaten. Ein optionaler
Cloud-Dienst darf später Komfort bieten, aber keine Voraussetzung für den
Grundbetrieb sein.

### Inhaltstreu

Roundtable führt keine LLM-basierte automatische Zusammenfassung im
Nachrichtenpfad durch. Technische Aufbereitung ist erlaubt:

- ANSI- und Cursorsteuerzeichen entfernen,
- wiederholt aktualisierte Terminalzeilen zusammenführen,
- große Inhalte verlustfrei in mehrere Nachrichten oder Dateien aufteilen,
- Codeblöcke transportgerecht darstellen,
- die unveränderte Roh-Ausgabe separat speichern.

### Erklärbare Aktionen

Vor jeder Aktion mit Seiteneffekt muss klar sein:

- welche Session betroffen ist,
- welcher Agent angesprochen wird,
- welches Projekt und Arbeitsverzeichnis verwendet werden,
- welche Eingabe oder Taste gesendet wird,
- ob die Aktion protokolliert wird,
- ob eine Bestätigung erforderlich ist.

## Erfolgsmerkmale

Roundtable erfüllt seine Kernidee, wenn ein Benutzer mindestens eine
Claude-Session und eine Codex-Session gleichzeitig betreiben, ihre Nachrichten
in demselben privaten Telegram-Chat empfangen und beide ausschließlich über
Replies fehlerfrei getrennt steuern kann.

Weitere wichtige Qualitätsmerkmale:

- Eine Antwort darf nie still an die falsche Session gehen.
- Nach einem Router-Neustart müssen bestehende Zuordnungen wiederherstellbar
  sein.
- Eine fehlende oder mehrdeutige Zuordnung führt zu einer Auswahl, nicht zu
  einer geratenen Zustellung.
- Der Standardbetrieb darf keine Secrets in Telegram offenlegen.
- Installation und Update müssen für Nicht-Projektentwickler handhabbar sein.
- Fehler müssen konkrete, sichere Wiederherstellungsaktionen anbieten.
