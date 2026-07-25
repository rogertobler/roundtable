# Roundtable: Anfrage für eine technische Zweitmeinung

Baseline: 25. Juli 2026

## Auftrag an den Reviewer

Bitte prüfe das folgende Produkt- und Architekturkonzept kritisch aus Sicht
eines erfahrenen Softwarearchitekten, Security Engineers und Entwicklers
interaktiver CLI- und Messaging-Systeme.

Gesucht ist keine allgemeine Bestätigung der Produktidee. Gesucht sind:

- falsche oder fragile Annahmen,
- technisch schwer lösbare Stellen,
- fehlende Zustände und Race Conditions,
- Sicherheitsprobleme,
- bessere Implementierungsvarianten innerhalb des festgelegten Produktmodells,
- eine realistische MVP-Abgrenzung,
- bestehende Projekte oder Bibliotheken, die wir prüfen sollten.

Bitte antworte auf Deutsch. Recherchiere zeitabhängige Aussagen nach
Möglichkeit neu, bevor du sie bewertest, und nenne direkte Primärquellen mit
Prüfdatum. Übernimm die unten genannten Marktannahmen nicht ungeprüft.

Einige Produktentscheidungen sind bewusst festgelegt. Bitte unterscheide in
deiner Antwort klar zwischen:

1. Problemen innerhalb der geplanten Architektur,
2. sinnvollen Verbesserungen, die das Produktmodell erhalten,
3. alternativen Produkten oder Architekturen, die zwar interessant sind, aber
   das festgelegte Produktmodell ersetzen würden.

## Produkt in einem Satz

Roundtable ist ein lokaler Multi-Agent-Chat-Router: Ein gemeinsamer
Messenger-Chat steuert mehrere gleichzeitig laufende native AI-CLIs; eine
Reply wird an die Session der beantworteten Nachricht geroutet, eine freie
Nachricht an die festgelegte Default-Session.

## Das zu lösende Problem

Entwickler verwenden zunehmend mehrere Coding-Agenten parallel:

- Claude Code implementiert ein Feature.
- Codex reviewt einen anderen Branch.
- Eine weitere Claude-Session untersucht einen Fehler.
- Eine Session wartet auf eine Rückfrage oder Freigabe.
- Andere Sessions arbeiten in völlig anderen Projekten.

Diese Agenten laufen heute als interaktive CLI-Prozesse auf einem lokalen
Rechner, in WSL oder auf einem privaten Server. Der Benutzer möchte sie auch
unterwegs bedienen, ohne für jede Rückfrage eine SSH-Verbindung und ein
Terminal öffnen zu müssen.

Roundtable macht einen privaten Telegram-Chat zur gemeinsamen mobilen Inbox und
Steuerungsoberfläche. Später soll derselbe Core weitere Messenger wie WhatsApp
unterstützen.

## Was Roundtable ausdrücklich ist

Roundtable ist:

- ein Chat-Router,
- ein Session-Manager,
- ein bidirektionaler Tunnel zwischen Messenger und nativer CLI,
- eine gemeinsame Inbox für mehrere Agent-Sessions,
- eine lokale Routing-, Zustell- und Audit-Schicht.

Roundtable ist nicht:

- ein eigener AI-Agent,
- ein eigener Agenten-Chat,
- ein Ersatz für Claude Code oder Codex,
- ein Agenten-SDK,
- ein Webterminal,
- eine zweite Quelle für den Gesprächskontext,
- eine Cloud, durch die Quellcode oder Agentenkontext laufen müssen.

## Nicht verhandelbare Produktentscheidungen

Die folgenden Regeln definieren das Produkt. Vorschläge dürfen ihre technische
Umsetzung verbessern, sollen sie aber nicht stillschweigend ersetzen.

### 1. Ein gemeinsamer Chat

Mehrere Claude-, Codex- und spätere Agent-Sessions erscheinen gemeinsam in
einem privaten Messenger-Chat.

Es ist nicht erforderlich, pro Session einen eigenen Chat oder ein Telegram
Forum Topic anzulegen.

### 2. Der Agent gehört zur Session

Agent und Modell sind keine globale Einstellung.

Beispiel:

```text
Session "Backend Feature"
  Agent: Claude Code
  Modell: Sonnet

Session "Backend Review"
  Agent: Codex
  Modell: ein ausgewähltes OpenAI-Modell

Session "Website"
  Agent: Claude Code
  Modell: Opus
```

Alle Sessions dürfen gleichzeitig laufen.

### 3. Die native CLI ist die einzige Agentenkonversation

Beim Start einer Session erzeugt Roundtable im Hintergrund eine echte
dauerhafte CLI-Session:

```text
Roundtable Session
  -> tmux Session
     -> lokal installierte und authentifizierte Agenten-CLI
```

Roundtable führt keinen parallelen API-, SDK- oder App-Server-Chat.

Alle Messenger-Eingaben werden in diese CLI geschrieben. Alle
Agentenantworten stammen aus der Ausgabe dieser CLI.

### 4. Lokales Terminal und Messenger sehen denselben Kontext

Der Benutzer kann sich lokal an die von Roundtable verwaltete tmux-Session
anhängen:

```text
tmux attach-session -t <session-name>
```

Er muss dort denselben laufenden Agentenprozess und denselben nativen Verlauf
sehen, einschließlich der über Telegram gesendeten Eingaben.

Er kann anschließend lokal weiterschreiben. Die daraus entstehenden
Agentenantworten sollen wiederum vom Roundtable-Tunnel erkannt und entsprechend
dem Abonnement in den Messenger geleitet werden.

### 5. Reply-Routing hat immer Vorrang

Jede von Roundtable gesendete routbare Messenger-Nachricht wird dauerhaft einer
Session zugeordnet:

```text
transport
chat_id
external_message_id
session_id
internal_message_id
message_type
created_at
```

Antwortet der Benutzer mit der nativen Reply-Funktion des Messengers auf eine
Nachricht, geht seine Antwort an genau die Session dieser Nachricht.

```text
Reply auf Nachricht aus Session A -> Session A
Reply auf Nachricht aus Session B -> Session B
```

Die Default-Session darf eine vorhandene Reply-Zuordnung niemals
überschreiben.

Ist die Reply-Zuordnung unbekannt, veraltet, nicht zugänglich oder mehrdeutig,
darf Roundtable nicht auf die Default-Session zurückfallen. Die Nachricht
bleibt ungesendet, und der Benutzer erhält eine klare Fehlermeldung mit
Sessionauswahl.

### 6. Freie Nachrichten gehen an die Default-Session

Eine Nachricht ohne Reply und ohne anderes explizites Sessionziel geht an die
Default-Session:

```text
Freie Nachricht -> Default-Session
```

Default-Regeln:

- Die erste erfolgreich erstellte Session wird automatisch Default, wenn noch
  kein Default existiert.
- Später erstellte Sessions verändern den Default nicht.
- Neue Agentennachrichten verändern den Default nicht.
- Statuswechsel verändern den Default nicht.
- Der Benutzer kann den Default manuell auf eine andere Session setzen.
- Pro Benutzer und Transportkontext existiert höchstens ein Default.
- Ist der Default gestoppt, gelöscht oder unzugänglich, wird keine andere
  Session geraten.

Es gibt bewusst keine automatisch wechselnde "aktive Session".

### 7. Inhalte werden inhaltstreu getunnelt

Benutzereingaben werden nicht umformuliert, übersetzt, ergänzt oder
zusammengefasst. Sie werden als literaler Text und die erforderliche
Abschlusstaste in die native CLI geschrieben.

Agentenausgaben werden ebenfalls nicht durch ein LLM umgeschrieben.

Erlaubt ist ausschließlich transporttechnische Aufbereitung:

- ANSI- und Cursorsteuerung verarbeiten,
- Spinner und überschreibende Statuszeilen stabilisieren,
- schnelle Outputstücke bündeln,
- Inhalte verlustfrei auf mehrere Messenger-Nachrichten aufteilen,
- sehr große Inhalte vollständig als Datei bereitstellen,
- Codeblöcke für den Messenger korrekt escapen,
- bekannte Secrets nach Best Effort maskieren.

Die vollständige rohe Ausgabe bleibt lokal verfügbar.

### 8. Approvals bleiben echte CLI-Interaktionen

Roundtable entscheidet keine Freigabe selbst.

Der sichtbare CLI-Prompt wird möglichst unverändert in den Messenger
weitergeleitet. Die Antwort des Benutzers wird als Text oder eindeutig
zugeordnete Tastenfolge an dieselbe CLI-Session zurückgeschickt.

Beispiel:

```text
Codex · Backend Review

Codex möchte folgenden Befehl ausführen:

npm test

[Einmal erlauben]
[Ablehnen]
```

Buttons sind nur eine Abkürzung für eine bekannte konkrete CLI-Eingabe:

```text
Einmal erlauben -> Taste "1" + Enter
Ablehnen        -> Taste "3" + Enter
```

Vor der Zustellung muss Roundtable prüfen, ob der Prompt noch aktuell ist.
Veraltete Buttons dürfen keine Eingabe senden.

Eine freie Reply auf den Approval-Prompt wird unverändert an dieselbe Session
gesendet. Roundtable interpretiert nicht semantisch, ob sie Zustimmung oder
Ablehnung bedeutet.

### 9. Hooks sind ausschließlich optionale Sensoren

Claude-Hooks oder andere strukturierte Agentenereignisse dürfen optional
Hinweise liefern:

- CLI ist bereit,
- Turn ist abgeschlossen,
- Rückfrage wartet,
- Approval wartet,
- Agent wurde beendet.

Diese Hinweise dürfen helfen, die passende Stelle der tmux-Ausgabe schneller
und zuverlässiger zu erkennen.

Sie dürfen nicht:

- einen zweiten Agentendialog erzeugen,
- eine Benutzernachricht an der CLI vorbeiführen,
- eine Agentenantwort außerhalb der CLI-Ausgabe liefern,
- eine Freigabe automatisch entscheiden.

Der grundlegende Tunnel muss auch ohne Hooks funktionieren.

## Beispielablauf

Der Benutzer erstellt über Telegram:

```text
Neue Session
Agent: Claude Code
Modell: Sonnet
Projekt: DumbleScore
Name: Backend Feature
Aufgabe: Implementiere den neuen Import.
```

Roundtable:

1. validiert Benutzer, Projekt, Pfad, Agent und Modell,
2. prüft, ob tmux und Claude Code installiert sind,
3. prüft die lokale Agentenauthentifizierung,
4. legt eine detached tmux-Session an,
5. startet Claude Code im Projektverzeichnis,
6. startet die Output-Erfassung,
7. wartet konservativ auf Eingabebereitschaft,
8. sendet die Startaufgabe in die CLI,
9. macht diese erste Session zum Default, falls noch kein Default existiert.

Danach erstellt der Benutzer eine zweite Session:

```text
Agent: Codex
Modell: ausgewähltes OpenAI-Modell
Projekt: DumbleScore
Name: Backend Review
Aufgabe: Prüfe den aktuellen Importcode unabhängig.
```

Beide Sessions senden in denselben Telegram-Chat:

```text
Claude · Backend Feature
Soll ich zusätzlich eine Migration erstellen?

Codex · Backend Review
Ich habe zwei mögliche Fehler im aktuellen Diff gefunden.
```

Der Benutzer antwortet auf die Claude-Nachricht:

```text
Prüfe zuerst, ob die Migration rückwärtskompatibel ist.
```

Diese Antwort geht ausschließlich an die Claude-tmux-Session.

Der Benutzer antwortet auf die Codex-Nachricht:

```text
Zeige mir beide Fehler mit Dateipositionen.
```

Diese Antwort geht ausschließlich an die Codex-tmux-Session.

Später schreibt der Benutzer ohne Reply:

```text
Führe jetzt die Tests aus.
```

Diese Nachricht geht an die festgelegte Default-Session. Die zweite Session hat
den Default beim Erstellen nicht automatisch übernommen.

## Vorgesehene Kernarchitektur

```text
                         +------------------+
                         | Telegram         |
                         | später WhatsApp  |
                         +---------+--------+
                                   |
                         Transport Adapter
                                   |
                         +---------v--------+
                         | Message Router   |
                         | Reply / Default  |
                         +----+--------+----+
                              |        |
                         SQLite        |
                     Routing/Audit     |
                                       v
                              +--------+---------+
                              | Session Manager  |
                              +--------+---------+
                                       |
                              Tunnel Dispatcher
                                       |
                              +--------v---------+
                              | tmux Backend      |
                              +--------+---------+
                                       |
                              +--------v---------+
                              | Claude/Codex CLI |
                              +------------------+

tmux Backend -> Output Collector -> Output Processor -> Transport Adapter

Lokales Terminal <------------------------------------> dieselbe tmux-Session
```

## Geplante Komponenten

### Transport Adapter

Verantwortlich für:

- Messenger-Updates empfangen,
- Benutzer und Chat identifizieren,
- Reply-Metadaten normalisieren,
- Text, Dateien und Buttons senden,
- externe Nachrichten-IDs zurückliefern,
- Telegram- oder WhatsApp-spezifische Limits behandeln.

Der Transport Adapter entscheidet nicht über die Zielsession.

### Message Router

Zielauflösung:

```text
if message is a reply:
    resolve exact message-to-session mapping
    if mapping invalid:
        do not send
else if message has an explicit valid session target:
    use that session
else if valid default exists:
    use default
else:
    do not send; request explicit selection
```

### Session Manager

Verantwortlich für:

- Sessiondefinitionen,
- Projekt- und Pfadprüfung,
- Agent- und Modellauswahl,
- CLI-Start,
- tmux-Lifecycle,
- Reattach nach Core-Neustart,
- Sessionstatus,
- Stoppen, Fortsetzen und Archivieren.

### Agent Definition

Eine Agentenintegration soll dünn bleiben und beschreiben:

- ausführbarer CLI-Befehl,
- Installation und Versionserkennung,
- Modellargumente,
- erlaubte Startargumente,
- Resume-Verhalten,
- Bereitschaftserkennung,
- bekannte Grundtasten,
- optionale Outputmuster,
- optionale Hook-Hinweise.

Sie ist kein alternativer Nachrichtenkanal.

### Session Backend

Der erste Backendvertrag wird durch tmux erfüllt:

- dauerhafte interaktive CLI starten,
- literalen Text schreiben,
- Tasten senden,
- fortlaufende rohe Ausgabe liefern,
- aktuellen Screen State liefern,
- Prozesszustand erkennen,
- Session nach Core-Neustart wiederfinden,
- lokales Attach ermöglichen.

Spätere Backends, beispielsweise ConPTY, müssen dieselbe sichtbare Semantik
erhalten.

### Output Collector und Output Processor

Vorgesehene Quellen:

- `tmux pipe-pane` für einen fortlaufenden Raw Stream,
- `tmux capture-pane` für den aktuellen Screen State,
- append-only Output-Datei mit persistiertem Cursor,
- optional Agenten-Hooks als reine Hinweise.

Offen ist insbesondere, wie daraus zuverlässig stabile neue
Agentennachrichten entstehen, ohne:

- Teile zu verlieren,
- dieselbe Ausgabe doppelt zu senden,
- Nutzereingaben fälschlich als Agentenantwort zu senden,
- Spinner und Vollbild-Neuzeichnungen zu fluten,
- zu warten, bis lange Turns vollständig beendet sind.

### Persistenz

SQLite soll mindestens speichern:

- Benutzer und Pairing,
- Projekte und Pfad-Allowlist,
- Sessiondefinition und tmux-Runtime-ID,
- Agent, Modell und Arbeitsverzeichnis,
- Default-Session,
- Messenger-Nachricht-zu-Session-Zuordnung,
- eingehende Update-ID und Idempotenzstatus,
- Inputzustellstatus,
- Output-Cursor,
- Benachrichtigungsmodus,
- Approval-Interaktion und Ablaufstatus,
- Auditereignisse.

SQLite speichert Routing- und Auditkopien. Es ist nicht der maßgebliche
Gesprächskontext des Agenten.

## Eingabezustellung

Messengertext darf niemals durch Shell-Interpolation in tmux geschrieben
werden.

Eine mögliche Strategie:

1. Text als Daten in einen tmux-Buffer laden.
2. Buffer literal in das Ziel-Pane einfügen.
3. Abschlusstaste separat senden.
4. Zustellung idempotent protokollieren.

Zu klären sind unter anderem:

- mehrzeilige Texte,
- Unicode,
- Steuerzeichen,
- Backticks und Shell-Metazeichen,
- sehr lange Eingaben,
- Telegram-Dateien,
- bracketed paste,
- CLIs mit eigenem Multiline-Modus,
- gleichzeitige lokale Eingabe,
- Prozesswechsel im tmux-Pane.

Ein Telegram-Retry oder doppelter Callback darf dieselbe Eingabe nicht zweimal
in die CLI schreiben.

## Ausgabe und Backpressure

Telegram hat begrenzte Nachrichtengrößen und Rate-Limits. Mehrere parallele
Sessions dürfen den gemeinsamen Chat nicht fluten.

Gleichzeitig darf Transport-Backpressure die native CLI nicht blockieren.

Vorgesehen:

- Raw Output lokal sofort persistieren,
- pro Session Output-Cursor führen,
- stabile Ausgaben bündeln,
- routbare Fragen und Approvals priorisieren,
- große Inhalte als Datei senden,
- bei Messenger-Ausfall lokal weiter erfassen,
- Versand mit Retry und Idempotenz fortsetzen,
- klare Meldung bei dauerhaft fehlgeschlagener Zustellung.

Eine Drosselung des Messenger-Versands darf nicht automatisch den
Agentenprozess pausieren. Ein separater optionaler Activity- oder Kosten-Guard
kann später Sessions kontrolliert unterbrechen, muss aber transparent und
konfigurierbar sein.

## Recovery und Lebenszyklen

Ein Neustart des Roundtable-Core darf laufende tmux-Sessions nicht automatisch
beenden.

Nach Neustart soll Roundtable:

1. SQLite öffnen,
2. registrierte tmux-Sessions erkennen,
3. Runtime-Identität prüfen,
4. Output-Cursor wiederherstellen,
5. Output Capture erneut verbinden,
6. ausstehende Zustellungen sicher fortsetzen,
7. tote oder fremde Sessions als Fehler markieren.

Eine tmux-Session darf nicht allein anhand eines frei wählbaren Namens
übernommen werden. Es braucht eine belastbare Zuordnung, beispielsweise über
interne IDs, tmux-Metadaten, kontrollierte Logpfade und Prozessdaten.

## Sicherheit

Roundtable steuert Prozesse, die Dateien ändern und Befehle ausführen können.
Der Messengerzugang ist daher sicherheitskritisch.

Mindestens vorgesehen:

- nur explizit gekoppelte Messenger-Benutzer,
- lokale Pairing-Prozedur mit Einmalcode,
- standardmäßig nur private Chats,
- kanonische Projekt- und Pfad-Allowlist,
- Symlink- und Path-Traversal-Prüfung,
- Betrieb ohne Root- oder Administratorrechte,
- sichere Speicherung des Bot-Tokens,
- keine Shell-Auswertung von Messengertext,
- keine automatische Approval-Entscheidung,
- idempotente Nachrichten und Buttons,
- Ablauf und Zustandsprüfung interaktiver Buttons,
- Audit kritischer Benutzeraktionen,
- Maskierung bekannter Secrets nach Best Effort,
- klare Kommunikation, dass Secret-Masking keine vollständige Garantie ist.

Wichtige Grenze:

Eine Pfad-Allowlist im Roundtable-Core beschränkt die von Roundtable
ausgewählten Arbeitsverzeichnisse. Sie kann allein nicht garantieren, dass ein
Agentenprozess später nicht auf andere Pfade zugreift. Starke Grenzen benötigen
zusätzlich agenteneigene Sandbox-Modi, Betriebssystemrechte, Container oder
andere Isolation.

## Plattformstrategie

### Linux

Erster MVP:

- installierbarer lokaler Daemon; Go ist der führende, noch im Spike zu
  bestätigende Kandidat,
- tmux,
- SQLite,
- Telegram Long Polling,
- `systemd --user`,
- Claude Code und Codex,
- lokaler Rechner oder VPS.

### macOS

Danach:

- tmux,
- `launchd`,
- Keychain,
- Homebrew oder signiertes Paket.

### Windows

Erster unterstützter Weg:

- Roundtable, tmux und Agenten-CLIs gemeinsam innerhalb von WSL.

Später optional nativ:

- eigener ConPTY Session Host,
- Named-Pipe-Kommunikation,
- persistente Output-Logs,
- `roundtable attach` mit derselben nativen Sessionsemantik.

Ein nativer Windows-Weg darf nicht durch einen separaten Agenten-API-Chat
erkauft werden.

## Geplanter MVP

Der erste nutzbare Linux-MVP soll enthalten:

- private Telegram-Bot-Verbindung,
- sicheres Pairing eines Benutzers,
- vorkonfigurierte Projekt-Allowlist,
- Claude-Code- und Codex-Definition,
- Agent und Modell pro Session,
- Session über Telegram erstellen,
- tmux-Session und CLI starten,
- mehrere Sessions parallel,
- gemeinsame Inbox,
- persistentes Reply-Routing,
- automatische initiale Default-Session,
- manueller Default-Wechsel,
- freie Nachrichten an den Default,
- inhaltstreue Texteingabe,
- fortlaufende Output-Erfassung,
- einfache Status- und Fehlererkennung,
- Approval-Prompts inhaltstreu weiterleiten,
- freie Replies auf Approval-Prompts 1:1 zurücktunneln,
- `Ctrl+C`, Enter und Escape,
- CLI-Snapshot,
- lokaler Attach-Befehl,
- Session stoppen,
- Core-Neustart ohne Ende laufender tmux-Sessions,
- SQLite mit WAL,
- lokales Raw-Output-Log,
- Telegram-Backpressure und Retry,
- Audit der Zustellungen.

Nicht zwingend im ersten MVP:

- Approval-Buttons und automatische Promptklassifikation,
- WhatsApp,
- natives Windows,
- Web-Dashboard,
- Teams und RBAC,
- automatische Agent-zu-Agent-Workflows,
- automatische Merges,
- vollständige Kostenanalyse,
- Spracheingabe,
- umfangreiche GitHub-/GitLab-Integration.

## Bekannte ähnliche Lösungen

### ccgram

Primärquellen:

- <https://github.com/alexei-led/ccgram>
- <https://github.com/alexei-led/ccgram/releases>

`ccgram` ist der derzeit engste bekannte Vergleich. Es verbindet
Telegram-Forum-Topics mit tmux- oder herdr-Sessions und unterstützt unter
anderem Claude Code und Codex.

Die angestrebte Kernunterscheidung:

```text
ccgram:
  ein Forum Topic entspricht einer Session

Roundtable:
  ein privater Chat ist die gemeinsame Inbox
  Reply bestimmt die Ursprungssession
  freie Nachricht geht an die Default-Session
```

Roundtable zielt außerdem auf einen transportneutralen Core, ein formales
Projekt- und Berechtigungsmodell sowie einen dokumentierten
plattformübergreifenden Installationsweg.

Bitte prüfe ausdrücklich, ob diese Abgrenzung korrekt, relevant und
ausreichend ist.

### HeyAgent

Primärquelle:

- <https://github.com/gergomiklos/heyagent>

HeyAgent ist eine lokale bidirektionale Telegram-Brücke für Claude Code und
Codex. Nach aktuellem Stand nutzt es einen aktiven Provider und einen Chat pro
laufendem CLI-Prozess. Roundtable unterscheidet sich durch mehrere gleichzeitig
sichtbare Sessions im selben Chat und das persistente Reply-/Default-Routing.

### Offizielle Claude-Funktionen

Primärquellen:

- <https://code.claude.com/docs/en/remote-control>
- <https://code.claude.com/docs/en/channels>

Claude Code Remote Control und Channels bestätigen den Bedarf an mobiler
Steuerung lokaler Sessions, sind aber Claude-spezifisch. Channels ist ein
alternativer direkter Nachrichtenpfad und soll nicht parallel zum
Roundtable-tmux-Tunnel verwendet werden. Claude-Hooks können davon getrennt als
optionale Erkennungssensoren dienen.

Bitte behandle diese Marktangaben als zu verifizierende Ausgangspunkte und
suche nach weiteren aktuellen Lösungen.

## Konkrete Fragen

Bitte beantworte mindestens die folgenden Fragen.

### Produkt und Differenzierung

1. Löst das Reply-/Default-Modell ein relevantes Problem besser als
   Topic-pro-Session oder getrennte Bots?
2. Ist der gemeinsame Chat bei fünf, zehn oder mehr Sessions noch ergonomisch?
3. Welche unverzichtbaren UX-Zustände oder Abläufe fehlen?
4. Ist die Abgrenzung zu ccgram und agentenspezifischen Remote-Control-Lösungen
   belastbar?
5. Welches kleinste Feature-Set würde den Nutzen zuverlässig beweisen?

### tmux und native CLI

6. Ist die Annahme realistisch, dass Messenger und lokales tmux-Attach
   zuverlässig denselben sichtbaren CLI-Verlauf teilen?
7. Welche Probleme entstehen durch Alternate Screen, Vollbild-TUIs,
   Cursorbewegungen, Resize, Scrollback und Eingabe-Echo?
8. Wie sollten `pipe-pane`, `capture-pane` und persistente Raw-Logs kombiniert
   werden?
9. Wie erkennt Roundtable neue stabile Agentenausgabe ohne fragile
   Wortlautparser?
10. Wie kann lokaler Benutzerinput von Agentenoutput und Messenger-Echo
    unterschieden werden?
11. Was passiert, wenn die CLI innerhalb des Pane einen Unterprozess startet
    oder ihre TUI grundlegend ändert?

### Routing und Zustellung

12. Welche Race Conditions oder Fehlroutings sind im Reply-/Default-Modell noch
    möglich?
13. Welche Idempotenzschlüssel und Zustandsübergänge braucht die
    Eingabezustellung?
14. Wann darf eine Messenger-Nachricht als erfolgreich zugestellt gelten?
15. Wie verhindert man doppelte Eingaben nach Crash, Retry oder Telegram
    Redelivery?
16. Wie sollte gleichzeitige lokale und mobile Eingabe serialisiert oder
    zumindest sichtbar gemacht werden?

### Approvals

17. Wie kann ein Approval-Prompt erkannt werden, ohne einen parallelen
    Agentendialog einzuführen?
18. Welche Bedingungen müssen erfüllt sein, bevor ein Button eine Tastenfolge
    senden darf?
19. Wie sollte Prompt-Ablauf beziehungsweise Stale-State-Prüfung implementiert
    werden?
20. Sollte der erste MVP Approval-Buttons anbieten oder zunächst ausschließlich
    freie Replies 1:1 tunneln?

### Sicherheit

21. Welche Angriffswege über Telegramtext, tmux, Projektpfade, Logdateien,
    Hooks oder manipulierte CLI-Ausgabe fehlen im Bedrohungsmodell?
22. Wie sollte das Pairing konkret gestaltet werden?
23. Welche Daten dürfen niemals in Telegram erscheinen?
24. Welche Sicherheitsversprechen kann Roundtable realistisch geben, und
    welche wären irreführend?
25. Welche Isolation ist für einen öffentlichen MVP angemessen?

### Betrieb und Plattformen

26. Ist Go für Core, Telegram, SQLite und tmux-Steuerung eine gute Wahl?
27. Welche Go-Bibliotheken oder vorhandenen Open-Source-Komponenten sind
    geeignet?
28. Wie sollte Recovery nach Core-, tmux-, Rechner- oder Netzwerkausfall
    aussehen?
29. Ist WSL/tmux als erster Windows-Weg für externe Benutzer akzeptabel?
30. Welche Installations- und Diagnosefunktionen müssen von Anfang an vorhanden
    sein?

## Gewünschtes Antwortformat

Bitte strukturiere die Zweitmeinung so:

### 1. Kurzurteil

- Ist das Produkt technisch realistisch?
- Ist die Differenzierung nachvollziehbar?
- Was ist das größte einzelne Risiko?

### 2. Kritische Probleme vor Implementierungsbeginn

Für jeden Punkt:

- Problem,
- konkretes Fehlerszenario,
- Auswirkung,
- empfohlene Entscheidung oder Experiment.

### 3. Notwendige Änderungen am MVP

Bitte als priorisierte Liste:

- muss hinein,
- sollte entfernt oder verschoben werden,
- kann unverändert bleiben.

### 4. Architekturvorschlag

Bitte mit:

- Komponenten,
- Verantwortlichkeiten,
- Datenfluss,
- Persistenz,
- Recovery,
- Idempotenz,
- Grenzen zwischen generischem Tunnel und agentenspezifischer Erkennung.

### 5. Technische Experimente

Bitte fünf bis zehn kurze Proof-of-Concept-Experimente nennen, die die größten
Annahmen früh validieren. Jedes Experiment soll enthalten:

- zu prüfende Annahme,
- minimaler Aufbau,
- messbares Erfolgskriterium,
- Abbruchkriterium.

### 6. Sicherheitsreview

Bitte Risiken nach Schweregrad ordnen und konkrete Gegenmaßnahmen nennen.

### 7. Bestehende Lösungen

Bitte nur konkrete, überprüfbare Projekte, Protokolle oder Bibliotheken nennen
und jeweils erklären:

- was wiederverwendbar ist,
- was nicht zu Roundtables Modell passt,
- welche Lizenz- oder Wartungsrisiken bestehen.

### 8. Offene Fragen

Bitte die wichtigsten Fragen aufführen, die vor der Implementierung durch
Produktentscheidung, Recherche oder Prototyp beantwortet werden müssen.

## Abschließender Hinweis

Bitte bewerte die festgelegte Architektur ehrlich. Wenn eine Kernannahme
praktisch nicht zuverlässig umsetzbar ist, benenne sie klar und schlage den
kleinsten Test vor, der dies beweist oder widerlegt.

Eine Empfehlung für direkte Agenten-APIs, einen eigenen Web-Chat oder
Topic-pro-Session darf als Alternative genannt werden. Sie beantwortet jedoch
nicht allein die eigentliche Frage, wie der beschriebene gemeinsame
Reply-/Default-Chat als Tunnel zu denselben lokal sichtbaren nativen
CLI-Sessions zuverlässig umgesetzt werden kann.
