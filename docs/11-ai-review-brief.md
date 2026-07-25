# Roundtable: Kurzbeschreibung für externes AI-Review

## Ausgangslage

Roundtable soll eine lokale, plattformübergreifende Steuerungs- und
Kommunikationsebene für mehrere gleichzeitig laufende Terminal-Agenten werden.

Ein Benutzer kann beispielsweise parallel betreiben:

- eine Claude-Code-Session, die eine Funktion implementiert,
- eine Codex-Session, die den Code überprüft,
- eine weitere Claude-Session in einem anderen Projekt,
- eine Session, die auf eine Freigabe wartet.

Alle Sessions laufen als echte interaktive Terminalprozesse auf dem lokalen
Rechner oder einem privaten Server. Roundtable verbindet diese Sessions mit
einem privaten Telegram-Bot-Chat. Später sollen weitere Messaging-Kanäle wie
WhatsApp hinzukommen.

Roundtable ist kein eigener KI-Agent. Es ist ein Router, Session-Manager,
Terminal-Gateway und eine gemeinsame mobile Inbox.

## Zentrales Produktversprechen

Jede Agent-Session besitzt ihren eigenen:

- Agenten, zum Beispiel Claude Code oder Codex,
- Modellwert,
- Namen,
- Projektkontext,
- Arbeitsordner oder Git-Worktree,
- Terminalprozess,
- Status,
- Berechtigungsmodus,
- Benachrichtigungsmodus.

Unterschiedliche Agenten und mehrere Instanzen desselben Agenten können
gleichzeitig laufen.

```text
Reply auf eine Agentennachricht -> Ursprungssession
Freie Nachricht ohne Reply      -> Default-Session
```

Der Benutzer arbeitet dadurch mit mehreren Agenten in einem einzigen
Telegram-Chat, ohne bei jeder Antwort Agent oder Projekt neu auswählen zu
müssen.

## Reply-basiertes Routing

Roundtable speichert für jede an Telegram gesendete Nachricht die zugehörige
Session-ID.

Beispiel:

```text
Claude Code · Backend Feature

Soll ich die Datenbankmigration jetzt ausführen?
```

Antwortet der Benutzer per Telegram-Reply:

```text
Prüfe zuerst, ob die alte Spalte noch verwendet wird.
```

wird diese Antwort exakt an die Claude-Session `Backend Feature` gesendet.

Eine Antwort auf eine Nachricht der Codex-Session wird entsprechend an Codex
gesendet. Der Nachrichteninhalt wird nicht zur Zielbestimmung interpretiert;
entscheidend ist ausschließlich die gespeicherte ID der beantworteten
Telegram-Nachricht.

Kann eine Nachricht nicht eindeutig zugeordnet werden, darf Roundtable nicht
raten. Stattdessen muss der Benutzer eine Session auswählen.

## Default-Session

Der Benutzer kann eine Session ausdrücklich als Default festlegen.

Eine Telegram-Nachricht ohne Reply und ohne explizites Sessionziel wird an
diese Default-Session gesendet. Die Default-Session ändert sich nicht
automatisch durch Aktivität, neue Agentennachrichten oder Statuswechsel.

Replies haben immer Vorrang vor der Default-Session.

Gibt es keine gültige Default-Session, zeigt Roundtable ein Auswahlmenü mit den
verfügbaren Sessions.

## Session-Erstellung über Telegram

Eine neue Session soll vollständig über ein geführtes Telegram-Menü erstellt
werden können:

1. Agent auswählen, zum Beispiel Claude Code oder Codex.
2. Modell oder Agentenprofil auswählen.
3. Projekt auswählen.
4. Arbeitsmodus auswählen:
   - vorhandenes Projektverzeichnis,
   - bestehender Git-Worktree,
   - neuer Git-Worktree,
   - nur lesen.
5. Sessionname festlegen.
6. Berechtigungsprofil auswählen.
7. Optional eine erste Aufgabe eingeben.
8. Zusammenfassung bestätigen und Session starten.

Später soll optional auch eine natürliche Kurzform möglich sein:

```text
Neue Claude-Session mit Opus für DumbleScore,
Name Backend Import Fix
```

Vor dem tatsächlichen Start wird daraus immer ein sichtbarer Sessionentwurf
erstellt, den der Benutzer bestätigt.

## Agentenausgaben

Agentenausgaben werden inhaltstreu an Telegram weitergegeben. Roundtable darf
Texte nicht selbst umformulieren, beantworten oder durch ein Sprachmodell
zusammenfassen.

Erlaubt ist ausschließlich technische Transportaufbereitung:

- ANSI- und Cursorsteuerzeichen entfernen,
- Spinner und ständig überschriebene Statuszeilen stabilisieren,
- schnell aufeinanderfolgende Ausgabestücke zusammenführen,
- lange Inhalte verlustfrei auf mehrere Nachrichten verteilen,
- vollständige Ausgaben als Text- oder Diff-Datei senden,
- Markdown und Codeblöcke für Telegram korrekt darstellen.

Die unveränderte Terminalausgabe bleibt lokal verfügbar.

## Rückfragen und Freigaben

Wenn ein Agent eine Frage oder Freigabe anzeigt, wird der originale Prompt an
Telegram gesendet.

Beispiel:

```text
Codex · Backend Review

Codex möchte folgenden Befehl ausführen:

npm test

[Einmal erlauben]
[Ablehnen]
[Terminal anzeigen]
```

Roundtable entscheidet die Freigabe nicht selbst. Eine Schaltfläche ist nur
eine deterministische Abkürzung für eine konkrete Terminaleingabe, zum Beispiel
`1` plus Enter oder eine definierte Tastenfolge.

Vor dem Senden wird geprüft, ob der Prompt noch aktuell ist. Alte oder bereits
beantwortete Buttons dürfen keine zweite Eingabe auslösen.

Eine freie Textantwort auf den Prompt wird unverändert an dieselbe Session
weitergegeben.

## Sessionübersicht

Telegram zeigt alle Sessions mit mindestens:

- Sessionname,
- Agent und Modell,
- Projekt,
- Status,
- Default-Markierung,
- Benachrichtigungsmodus,
- letzter Aktivität,
- ausstehender Rückfrage oder Freigabe.

Vorgesehene Zustände:

- startet,
- bereit,
- arbeitet,
- wartet auf Eingabe,
- wartet auf Freigabe,
- abgeschlossen,
- unterbrochen,
- gestoppt,
- Fehler,
- Verbindung verloren.

## Sessionaktionen

Pro Session sind unter anderem vorgesehen:

- Nachricht senden,
- als Default festlegen,
- abonnieren oder stummschalten,
- Benachrichtigungsmodus ändern,
- Terminalbild anzeigen,
- vollständige Ausgabe und Verlauf anzeigen,
- Enter, Escape, Tab und Pfeiltasten senden,
- `Ctrl+C` senden,
- Agent unterbrechen oder fortsetzen,
- Session neu starten,
- Session stoppen oder archivieren,
- lokal im echten Terminal öffnen.

Roundtable soll primär eine Agentensteuerung bleiben und kein vollständiger
grafischer Terminalersatz werden.

## Projekte und parallele Arbeit

Projekte werden lokal vorkonfiguriert und freigegeben. Eine Projektdefinition
kann enthalten:

- erlaubter Hauptpfad und Unterverzeichnisse,
- Git-Repository,
- erlaubte Agenten,
- Standard-Agent und Standardmodell,
- Standardberechtigungen,
- Umgebungsvariablen-Referenzen,
- Benachrichtigungseinstellungen.

Wenn mehrere Agenten am selben Repository arbeiten, sollen getrennte
Git-Worktrees und Branches unterstützt werden. Roundtable zeigt jederzeit,
welche Session in welchem Pfad und Branch arbeitet.

## Plattformen und Installation

Roundtable soll für folgende Betriebsarten installierbar sein:

- Linux-Desktop,
- Linux-VPS,
- macOS,
- Windows nativ,
- Windows mit WSL.

Geplante Runtime-Backends:

- Linux und macOS zunächst über tmux,
- Windows nativ über ConPTY und einen separaten Session-Host-Prozess,
- Windows mit WSL optional über tmux.

Die Terminal-Runtime ist austauschbar. tmux darf deshalb nicht fest in die
Produktlogik eingebaut sein.

Ein separater Session Host oder tmux soll dafür sorgen, dass laufende Agenten
bei einem Neustart des Telegram-Routers möglichst weiterlaufen.

Die Installation soll später über ein ausführbares Programm und einen
Einrichtungsassistenten funktionieren:

```text
roundtable install
roundtable setup
roundtable doctor
roundtable start
```

## Geplante Architektur

Roundtable trennt drei Adapterarten:

```text
TransportAdapter
  Telegram
  später WhatsApp, Web, Discord oder andere Kanäle

AgentAdapter
  Claude Code
  Codex
  später Gemini CLI, Aider, OpenCode oder Generic Terminal

RuntimeBackend
  tmux
  ConPTY Session Host
  WSL/tmux
  später Remote Nodes oder Container
```

Der Roundtable Core verwaltet:

- Benutzer und Autorisierung,
- Projekte,
- Sessions,
- Reply-Routing,
- Default-Session,
- Eingabe-Queues,
- Status und Interaktionen,
- Benachrichtigungen,
- Verlauf und Audit.

SQLite ist als lokale Datenbank vorgesehen. Große rohe Terminalausgaben können
in lokalen Dateien liegen und über SQLite referenziert werden.

## Sicherheit

Da Agenten Dateien verändern und Befehle ausführen können, gelten mindestens:

- nur explizit freigegebene Telegram-Benutzer,
- standardmäßig nur private Chats,
- sichere Pairing-Prozedur,
- Projekt- und Verzeichnis-Allowlist,
- kein Betrieb als Root oder Administrator,
- keine Shell-Auswertung von Telegramtexten,
- sichere Speicherung von Bot-Token und API-Schlüsseln,
- Maskierung bekannter Secrets vor Telegram-Ausgaben,
- nachvollziehbare Freigabehistorie,
- idempotente Nachrichten und Buttons,
- keine automatische Produktionsänderung,
- keine geratenen Nachrichtenziele.

Vorgesehene Berechtigungsprofile:

- Nur lesen,
- Standard,
- Vertrauenswürdig innerhalb klarer Projektgrenzen,
- Eingeschränkt mit häufigen Bestätigungen.

## Benachrichtigungen

Pro Session sind folgende Modi geplant:

- alle stabilen Ausgaben,
- nur Rückfragen,
- nur Freigaben,
- nur Abschlussmeldungen,
- nur Fehler,
- stumm.

„Stumm“ beendet die Session nicht. Der lokale Verlauf bleibt erhalten.

## Spätere Erweiterungen

- WhatsApp und weitere Messaging-Kanäle,
- Datei-, Bild- und Screenshotübertragung,
- Sprachnachrichten und Transkription,
- automatische Git-Worktrees,
- mehrere lokale oder entfernte Rechner,
- Web-Dashboard,
- mehrere Benutzer, Teams und Rollen,
- Agent-zu-Agent-Übergaben,
- Implementierungs- und Review-Pipelines,
- geplante und wiederkehrende Aufgaben,
- GitHub- und GitLab-Integration,
- Kosten- und Tokenübersichten,
- Monitoring- und Deployment-Integrationen.

## Nicht verhandelbare Kernregeln

1. Der Agent gehört zur Session und ist keine globale Einstellung.
2. Claude und Codex müssen gleichzeitig bedienbar sein.
3. Replies werden an die Ursprungssession geroutet.
4. Freie Nachrichten gehen nur an die explizite Default-Session.
5. Bei unklarer Zuordnung wird niemals geraten.
6. Roundtable schreibt Agenten- oder Benutzerinhalte nicht um.
7. Freigaben werden nicht selbstständig entschieden.
8. Transport, Agent und Terminal-Runtime bleiben austauschbar.
9. Linux, macOS und Windows gehören zum Zielprodukt.
10. Der Grundbetrieb bleibt local-first.

## Fragen an den Reviewer

Bitte analysiere dieses Konzept kritisch und konkret:

1. Welche wichtigen Benutzerabläufe oder Funktionen fehlen?
2. Wo siehst du die größten technischen Risiken?
3. Welche Annahmen über Claude Code, Codex, tmux oder ConPTY könnten falsch
   oder zu fragil sein?
4. Wie würdest du das Reply-Routing und die Eingabe-Idempotenz absichern?
5. Wie sollte zuverlässige Approval-Erkennung gestaltet werden, ohne Prompts
   unsicher zu interpretieren?
6. Welche Sicherheitsrisiken wurden übersehen?
7. Welche Funktionen sollten bereits in die erste nutzbare Version und welche
   bewusst später kommen?
8. Ist die Trennung zwischen Transport-, Agent- und Runtime-Adaptern sinnvoll?
9. Welche Technologie und Sprache würdest du für ein einfach installierbares
   Linux-/macOS-/Windows-Produkt wählen und warum?
10. Welche bestehenden Open-Source-Projekte oder Bibliotheken könnten als
    Grundlage dienen?
11. Was würde Roundtable gegenüber vorhandenen Telegram-Bridges und
    agentenspezifischen Remote-Control-Lösungen klar differenzieren?
12. Welche Änderungen würden das Produkt für externe Benutzer verständlicher,
    sicherer und attraktiver machen?

Bitte priorisiere deine Vorschläge nach:

- kritisch vor Implementierungsbeginn,
- wichtig für die erste Version,
- sinnvoll für spätere Versionen.
