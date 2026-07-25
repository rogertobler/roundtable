# Session S-0001: Produktdefinition und Architektur

## Metadaten

| Feld | Wert |
| --- | --- |
| Session-ID | `S-0001` |
| Datum | `2026-07-25` |
| Status | abgeschlossen |
| Verantwortlich | Roger Tobler |
| Beteiligte | Roger Tobler, Codex, externe Reviews durch Claude und Gemini |
| Ausgangscommit | Projektinitialisierung vor `302f5f1` |
| Abschlusscommit | wird mit diesem Sessionbericht ergänzt |

## Ziel

Roundtable als Produkt vollständig definieren, die zentrale
Messenger-/tmux-Architektur festlegen, externe technische Zweitmeinungen
einholen und einen belastbaren Ausgangsstand für die Implementierung schaffen.

## Ausgangslage

Die Ausgangsidee war ein „Multi-Agent Telegram Gateway“, über das mehrere
parallel laufende Claude-Code-, Codex- und spätere Agentensessions mobil
gestartet, beobachtet und gesteuert werden können.

Zu Beginn war noch nicht abschließend geklärt:

- ob Roundtable primär Terminal-, Agenten- oder Messengerprodukt ist,
- ob Agenten über APIs, strukturierte Protokolle oder echte CLIs angebunden
  werden,
- wie Replies und freie Nachrichten geroutet werden,
- wie Approvals behandelt werden,
- wie lokale und mobile Bedienung denselben Kontext behalten,
- unter welchem GitHub-Konto das öffentliche Projekt liegt,
- welche ähnliche Software bereits existiert,
- welcher Tech Stack für eine allgemein installierbare Version geeignet ist.

## Besprochene Punkte

### Gemeinsame Inbox statt Agentenauswahl pro Nachricht

- Fragestellung: Wie können mehrere Claude-, Codex- und weitere Sessions in
  einem einzigen privaten Chat bedient werden?
- Wesentliche Argumente: Der Benutzer soll nicht vor jeder Antwort erneut
  Agent oder Projekt auswählen. Die beantwortete Messenger-Nachricht besitzt
  bereits eine eindeutige Ursprungssession.
- Ergebnis: Jede von Roundtable gesendete routbare Nachricht speichert
  Transport-, Chat-, Nachrichten- und Session-ID. Eine native Reply geht immer
  an diese Ursprungssession.

### Default-Session für freie Nachrichten

- Fragestellung: Wohin geht eine Nachricht ohne Reply?
- Diskutierte Begriffe: Zunächst wurde von einer „aktiven Session“
  gesprochen. Dieser Begriff hätte nahegelegt, dass neue Aktivität oder
  Agentennachrichten das Ziel automatisch verändern.
- Ergebnis: Der verbindliche Begriff ist `Default-Session`.
- Regel: Die erste erfolgreich erstellte Session wird Default, wenn noch kein
  Default existiert. Weitere Sessions, Statuswechsel oder Agentennachrichten
  ändern ihn nicht. Danach kann ausschließlich der Benutzer den Default
  wechseln.
- Sicherheitsregel: Eine unbekannte oder ungültige Reply fällt niemals auf den
  Default zurück.

### Agent gehört zur Session

- Fragestellung: Wird global zwischen Claude und Codex umgeschaltet?
- Ergebnis: Nein. Agent, Modell, Projekt, Arbeitsverzeichnis und
  Berechtigungsprofil gehören zu jeder einzelnen Session.
- Konsequenz: Claude und Codex können gleichzeitig in verschiedenen Sessions
  betrieben werden. Mehrere Sessions desselben Agenten sind ebenfalls möglich.

### Roundtable ist ein Messengerprodukt

- Präzisierung des Produktverantwortlichen: Roundtable ist nicht das Terminal.
  Die primäre Oberfläche ist immer ein gemeinsamer Chat, zunächst Telegram und
  später beispielsweise WhatsApp.
- Das Terminal bleibt dennoch direkt zugänglich, weil es dieselbe laufende
  Agenten-CLI zeigt.
- CLI-Snapshots und Grundtasten im Messenger sind Diagnose- und
  Fallbackfunktionen, keine zweite Terminaloberfläche.

### Echte CLI in tmux statt zweitem Agentendialog

- Diskutierte Alternative: Direkte Agenten-APIs, Claude-Hooks oder der Codex
  App Server als eigener Nachrichtenpfad könnten strukturierte Antworten und
  Approvals liefern.
- Produktanforderung: Wer die tmux-Session lokal öffnet, muss den normalen
  nativen CLI-Chatverlauf einschließlich der Messenger-Eingaben sehen und dort
  weiterarbeiten können.
- Ergebnis: Jede Roundtable-Session ist ein Tunnel zu genau einer echten
  interaktiven Agenten-CLI in tmux.
- Eingaben laufen immer über tmux in diese CLI.
- Agentenausgaben stammen immer aus dieser CLI-Session.
- Hooks oder strukturierte Events dürfen ausschließlich optionale Sensoren für
  Status, Turn-Ende oder sichtbare Approvals sein.

### Approvals

- Produktanforderung: Roundtable darf Freigaben weder interpretieren noch
  selbst entscheiden.
- Ergebnis: Der sichtbare Approval-Prompt wird inhaltstreu weitergeleitet. Eine
  freie Reply wird 1:1 an dieselbe tmux-Session gesendet.
- Deterministische Buttons dürfen später bekannte Text- oder Tastenfolgen
  abkürzen, aber nur nach Prüfung des aktuellen Prompts.
- MVP-Schnitt: Approval-Buttons und automatische Promptklassifikation wurden
  in Phase 2 verschoben. Der erste MVP verwendet normale routbare Nachrichten
  und freie Replies.

### Inhaltstreue und technische Aufbereitung

- „1:1“ bedeutet, dass Roundtable Inhalte nicht durch ein LLM umformuliert,
  zusammenfasst, übersetzt oder beantwortet.
- Terminalartefakte wie ANSI-Sequenzen, Spinner, Cursorbewegungen und
  wiederholt überschriebene Zeilen dürfen technisch normalisiert werden.
- Der vollständige Raw Output bleibt lokal verfügbar.
- Secret-Redaktion für Messenger- und Diagnosetexte ist Best Effort und keine
  Sicherheitsgarantie.

### Plattformstrategie

- Zielplattformen: Linux, macOS und Windows sowie lokaler Rechner oder VPS.
- Erste Runtime: tmux unter Linux und macOS.
- Erster Windows-Weg: WSL mit Roundtable, tmux und Agenten-CLIs in derselben
  Distribution.
- Native Windows-Unterstützung über ConPTY bleibt eine spätere Runtime mit
  derselben Tunnelsemantik.

### Repository-Ownership

- Diskutierte Optionen: Repository unter der GitHub-Organisation
  `inscribe-GmbH` oder unter dem persönlichen Konto `rogertobler`.
- Berücksichtigte Argumente: persönliche Sichtbarkeit, Open-Source-Identität,
  mögliche spätere Community, Unabhängigkeit von einem Firmenprodukt und
  spätere Übertragbarkeit.
- Entscheidung: Das öffentliche Projekt liegt unter
  `rogertobler/roundtable`.
- Eine spätere Übertragung in eine eigene Organisation bleibt möglich.

### Bestehende Lösungen

- Recherchiert wurden unter anderem ccgram, HeyAgent, amux, dmux, Claude Code
  Remote Control, Claude Code Channels, Codex Telegram Bridge und
  cc-telegram-bridge.
- Ergebnis der Diskussion: Existierende Software ist kein Grund, Roundtable
  nicht zu bauen.
- Sie dient als technische Referenz und als Quelle für bekannte Fehlerfälle.
- Roundtable wird für den eigenen täglichen Einsatz gebaut und soll durch
  konsequentes Dogfooding eine bessere User Experience erreichen.
- Die zentrale UX-Grenze bleibt ein privater gemeinsamer Chat mit
  Reply-/Default-Routing über viele gleichzeitige Sessions und Agenten.

### Externe Reviews

Zwei ausführliche Reviews durch Claude und Gemini bestätigten die
grundsätzliche technische Machbarkeit und die UX-Differenzierung. Beide hoben
die tmux-I/O-Schicht als größtes Risiko hervor:

- rohe Terminalausgabe besitzt keine sicheren Agentennachrichtgrenzen,
- TUI-Redraws und Alternate Screen erschweren die Darstellung,
- Eingabe-Echo kann als Agentenausgabe erscheinen,
- lokale und mobile Eingaben können sich vermischen,
- ein Core-Absturz kann mitten in einer mehrstufigen Zustellung auftreten,
- Reattach darf keine fremde tmux-Session übernehmen.

### Bewertung der Review-Empfehlungen

Übernommen:

- Terminal-I/O wird vor Produktimplementierung durch einen Phase-0-Spike
  bewiesen.
- Lokales und mobiles Schreiben in dasselbe Pane ist nicht gleichzeitig
  erlaubt.
- Die Agenten-CLI läuft direkt als Pane-Prozess ohne interaktiven
  Shell-Fallback.
- Reattach verwendet stabile tmux-IDs und Roundtable-eigene User Options mit
  UUID und Runtime-Generation.
- Pane-Writes besitzen persistierte Zwischenzustände.
- Nach einem unklaren Teilabsturz gilt `delivery_uncertain`; es gibt keinen
  automatischen Retry.
- Terminalmaße werden pro Runtime deterministisch gespeichert.
- Multiline, Unicode, Echo, Bracketed Paste, TUI-Redraws und Crash-Recovery
  erhalten explizite Experimente.

Nicht ungeprüft übernommen:

- tmux Control Mode löst das Outputproblem nicht allein. `%output` bleibt
  roher Terminaloutput und besitzt keine Agentenantwortgrenzen.
- Bracketed Paste wird nur verwendet, wenn das Pane über
  `bracket_paste_flag` meldet, dass die Anwendung den Modus angefordert hat.
- Globales `stty -echo`, erzwungenes `TERM=dumb` oder `CI=true` würden das
  native CLI-Verhalten verändern und sind keine Standardstrategie.
- Ein einfacher Text-Echo-Canceller könnte legitime identische Agentenausgabe
  entfernen.
- `capture-pane` kann nach einem Teilabsturz keine Exactly-once-Zustellung
  beweisen.
- Raw Logs werden nicht zerstörend redigiert. Sie werden stattdessen durch
  Rechte, Retention, optionale Verschlüsselung oder Deaktivierung geschützt.
- Viele Terminalzeilen allein sind kein verlässliches Signal für Kosten oder
  eine Endlosschleife und lösen nicht automatisch `SIGSTOP` aus.

### Lokale und mobile Schreibsperre

- Wenn ein normaler interaktiver tmux-Client attached ist, bleiben neue
  Messenger-Eingaben in der Queue.
- Der Benutzer kann lokal detachen.
- Alternativ kann er eine mobile Übernahme ausdrücklich bestätigen.
- Die Übernahme detached normale tmux-Clients kontrolliert und führt danach
  den vollständigen Preflight erneut aus.
- Bleibt ein interaktiver Client attached, beginnt der Write nicht.

### Tech-Stack-Diskussion

Empfohlener Stack:

- Go als Implementierungssprache,
- modularer Monolith und einzelnes Binary,
- Cobra für die CLI,
- `go-telegram/bot` für Telegram,
- SQLite im WAL-Modus über den CGO-freien Treiber `modernc.org/sqlite`,
- eingebettete SQL-Migrationen mit goose,
- TOML-Konfiguration,
- Standardbibliothek `slog`,
- kleiner eigener typisierter tmux-Adapter,
- GitHub Actions, GoReleaser und nFPM für Releases.

Begründung:

- keine Python- oder Node-Runtime für Benutzer,
- einfache Cross-Compilation,
- geeignete Concurrency für parallele Sessions,
- gute Unterstützung für Systemdienste,
- einheitliche Distribution für Linux, macOS und später Windows,
- kontrollierbare Abhängigkeiten.

Bewusst nicht vorgesehen:

- keine Microservices,
- kein ORM,
- kein Webframework im Kern,
- keine Dependency-Injection-Library,
- keine Docker-Installation oder Docker-Runtime.

Der Tech Stack ist die klare Arbeitsrichtung für den nächsten Spike. Die
bestehende Entscheidung `D-013` bleibt bis zum erfolgreichen Go-/tmux-Spike
formal `vorgeschlagen`.

### Installations- und Setup-Erfahrung

Die Installation wurde als Teil des Produkts definiert:

```text
roundtable setup
roundtable doctor
roundtable service install
roundtable start
```

`setup` soll Telegram verbinden, Pairing durchführen, tmux und Agenten-CLIs
erkennen, lokale Authentifizierung prüfen, Projekte freigeben, den Dienst
installieren und eine Testnachricht senden.

Geplante Distribution:

- einzelnes Release-Binary,
- Install-Script,
- Homebrew,
- `.deb` und `.rpm`,
- WSL-orientierter Windows-Installer,
- `systemd --user` unter Linux,
- `launchd` unter macOS.

### Host-native Codebasis

Der Produktverantwortliche präzisierte nach der Stack-Diskussion:

- Docker ergibt als Standardweg keinen Sinn, weil die Agenten-CLI auf die
  Entwicklungsumgebung des Rechners zugreifen muss.
- Roundtable und direktes Arbeiten am Computer müssen dieselbe Codebasis
  verwenden.
- Änderungen müssen ohne Kopieren oder Synchronisieren sofort im lokalen
  Editor, Git-Worktree und in der Roundtable-Session sichtbar sein.
- Deshalb laufen Roundtable, tmux und Agenten standardmäßig host-nativ mit den
  vorhandenen lokalen Agenteninstallationen, Credentials und Werkzeugen.
- Isolation nutzt agenteneigene Sandboxfunktionen, Betriebssystemrechte und
  getrennte Worktrees, nicht eine Roundtable-Container-Runtime.

## Entscheidungen

### Produkt- und Routingmodell

- Status: verbindlich
- Entscheidung: Gemeinsamer Messenger-Chat, Agent pro Session,
  Reply-Routing zur Ursprungssession und Default-Session ausschließlich für
  freie Nachrichten.
- Begründung: Minimale Bedienlast bei gleichzeitig eindeutiger Zuordnung.
- Konsequenz: Nachrichteninhalt wird nie zur Zielbestimmung interpretiert.

### Einziger Agentendialog

- Status: verbindlich
- Entscheidung: Die native Agenten-CLI in tmux ist der einzige
  Agentengesprächskontext.
- Begründung: Messenger und lokales Terminal müssen denselben Verlauf und
  Prozess bedienen.
- Konsequenz: APIs und Hooks bleiben optionale Sensoren.

### Terminal-I/O-Sicherheitsmodell

- Status: verbindlich
- Entscheidung: Ein Writer pro Pane, direkter Agentenprozess,
  persistierte Write-Stufen, `delivery_uncertain` ohne automatischen Retry und
  markerbasiertes Reattach.
- Begründung: Verhindert Interleaving, Shell-Fehlzustellung,
  Doppelzustellung und falsches Reattach.
- Konsequenz: Der Phase-0-Spike ist Voraussetzung für den MVP.

### Repository

- Status: verbindlich
- Entscheidung: Öffentliches Repository unter `rogertobler/roundtable`.
- Begründung: Persönliches Open-Source-Projekt mit klarer Ownership und späterer
  Übertragbarkeit.

### Tech Stack

- Status: vorgeschlagen
- Entscheidung: Go-basierter modularer Monolith mit SQLite und tmux.
- Begründung: Beste Kombination aus einfacher Installation,
  Cross-Compilation, Parallelität und Dienstintegration.
- Bedingung: Der technische Spike muss tmux-I/O, Terminalverarbeitung und
  Packaging bestätigen.

### Host-native Workspace-Parität

- Status: verbindlich
- Entscheidung: Roundtable und direkte lokale Arbeit verwenden dieselbe
  freigegebene Codebasis und denselben Git-Worktree. Docker ist kein
  Roundtable-Installations- oder Runtime-Modell.
- Begründung: Nahtloser Wechsel zwischen Messenger und Rechner sowie sofort
  sichtbare gemeinsame Datei- und Git-Änderungen.
- Konsequenz: Installation, Diagnose und Sessionstart prüfen die reale
  Hostumgebung.

## Verworfene oder vertagte Alternativen

### Global aktiver Agent

- Verworfen, weil Claude und Codex gleichzeitig pro Session auswählbar sein
  müssen.

### Automatisch wechselnde aktive Session

- Verworfen, weil neue Agentenaktivität freie Nachrichten unbemerkt an ein
  anderes Ziel senden könnte.

### Topic pro Session

- Nicht als Roundtable-UX übernommen. Der Benutzer soll in einem normalen
  privaten Chat bleiben.

### Direkte Agenten-API als Hauptpfad

- Verworfen, weil dadurch ein zweiter Gesprächskontext neben der lokal
  sichtbaren CLI entstehen würde.

### Strukturierte Approval-APIs als Hauptpfad

- Verworfen. Approvals bleiben sichtbare native CLI-Interaktionen.

### Approval-Buttons im ersten MVP

- In Phase 2 verschoben, bis Prompt-Fingerprint, Stale-State-Prüfung und
  Tastenwirkung zuverlässig bewiesen sind.

### Gleichzeitige lokale und mobile Eingabe

- Verworfen. Oberflächen können denselben Kontext abwechselnd, aber nicht
  gleichzeitig schreibend bedienen.

### Native Windows-Runtime im ersten MVP

- Vertagt. Zuerst WSL/tmux, später ConPTY mit demselben Backendvertrag.

### Python oder TypeScript als Hauptimplementierung

- Aktuell nicht empfohlen, da Go die einfachere runtimefreie Distribution und
  Cross-Compilation bietet.

### Docker als Standardweg

- Verbindlich ausgeschlossen, weil lokale CLI-Authentifizierung, Projektpfade,
  Entwicklungswerkzeuge und direktes tmux-Attach dadurch unnötig kompliziert
  würden. Isolation bleibt Aufgabe der Agentensandbox, der Betriebssystemrechte
  und getrennter Worktrees.

## Durchgeführte Arbeiten

Die vollständige Produkt- und Architekturdokumentation wurde erstellt und
mehrfach auf den präzisierten Soll-Stand gebracht:

- `README.md`,
- `docs/README.md`,
- `docs/01-product-vision.md`,
- `docs/02-functional-requirements.md`,
- `docs/03-telegram-ux.md`,
- `docs/04-architecture.md`,
- `docs/05-domain-model.md`,
- `docs/06-agents-and-runtimes.md`,
- `docs/07-security-and-operations.md`,
- `docs/08-roadmap.md`,
- `docs/09-decisions-and-open-questions.md`,
- `docs/10-existing-solutions.md`,
- `docs/11-ai-review-brief.md`,
- `docs/12-ai-second-opinion-request.md`,
- `docs/history/`.

Zusätzlich wurden die Dokumente vollständig auf alte, widersprüchliche
Architekturvarianten geprüft. Normative Dokumente enthalten nur den aktuellen
Soll-Stand.

## Recherche und Quellen

### tmux

- <https://github.com/tmux/tmux/wiki/Control-Mode>
- <https://man.openbsd.org/tmux>
- Geprüft wurden Control Mode, `%output`, Flow Control, User Options,
  Session-/Pane-IDs, Client-Metadaten, `capture-pane`, `load-buffer`,
  `paste-buffer`, `bracket_paste_flag`, `remain-on-exit` und feste
  Fenstergrößen.

### Bestehende Projekte

- <https://github.com/alexei-led/ccgram>
- <https://github.com/gergomiklos/heyagent>
- <https://github.com/ssamssae/codex-telegram-bridge>
- <https://github.com/cloveric/cc-telegram-bridge>
- Diese Projekte validieren Teilaspekte des Tunnelansatzes, ändern aber nicht
  die Entscheidung, Roundtable mit eigener UX zu bauen.

### Codex App Server

- <https://github.com/openai/codex/blob/main/codex-rs/app-server/README.md>
- Der App Server unterstützt strukturierte Approval-Anfragen.
- Er bleibt technische Referenz, aber nicht Roundtables Hauptpfad.

### Vorgeschlagener Go-Stack

- <https://github.com/go-telegram/bot>
- <https://pkg.go.dev/modernc.org/sqlite>
- <https://github.com/spf13/cobra>
- <https://github.com/pressly/goose>
- <https://github.com/pelletier/go-toml>
- <https://goreleaser.com/>

## Verifikation

Durchgeführt:

- vollständiger Dokumentationsaudit auf alte Architekturbegriffe,
- Prüfung der Reply-/Default-Invarianten über alle Dokumente,
- Prüfung lokaler Markdown-Links,
- Prüfung eindeutiger Decision- und Open-Question-IDs,
- `git diff --check`,
- Abgleich technischer tmux-Aussagen mit offiziellen Quellen,
- Abgleich der Codex-Approval-Aussagen mit der offiziellen App-Server-
  Dokumentation.

Noch nicht durchgeführt:

- keine Produktimplementierung,
- kein realer tmux-I/O-Spike,
- keine Tests mit echten Claude-/Codex-TUIs,
- keine Telegram-Bot-Implementierung,
- kein Packaging-Test.

## Commits

- `302f5f1`: erste vollständige Produkt- und Architekturdokumentation
- `d6c2a25`: eigenständiger kompakter AI-Review-Brief
- `13807e4`: native CLI-/tmux-Tunnelarchitektur verbindlich beschrieben
- `d6a53db`: ausführlicher Zweitmeinungsauftrag und Baseline-Audit
- `d18099e`: Terminal-I/O-Architektur nach externem Review gehärtet
- Abschlusscommit dieser Session: wird nach Erstellung des Berichts ergänzt

## Offene Punkte

- Go-/tmux-Stack durch ausführbaren Phase-0-Spike bestätigen.
- Fortlaufende Outputquelle zwischen `pipe-pane` und Control Mode anhand
  gemessener Ergebnisse wählen.
- Terminalemulator für stabile Screen-Rekonstruktion evaluieren.
- Sichere Eingabematrix für Claude Code und Codex erstellen.
- Standard-Terminalgröße festlegen.
- Telegram Long Polling und Reply-Mapping als vertikalen Slice umsetzen.
- Packaging auf Linux amd64/arm64 und macOS amd64/arm64 beweisen.
- Projektlizenz festlegen.

## Nächster Schritt

Die nächste Session soll das Go-Projekt scaffolden und den Phase-0-Spike
beginnen:

1. Go-Modul, Verzeichnisstruktur, CI und Qualitätswerkzeuge anlegen.
2. Typisierten tmux-Adapter implementieren.
3. Kontrollierten Testagenten als deterministische CLI-Fixture bauen.
4. Sessionstart, direkte Pane-Prozesse, Marker, feste Größe, Output Capture und
   literal Eingabe testen.
5. Crash-, Reattach-, Echo-, Multiline- und lokale/mobile Konkurrenztests
   ausführen.
6. Erst nach erfolgreichem Spike den Telegram-Adapter anbinden.

## Nachträge

Dieser Bericht dokumentiert die erste zusammenhängende Produktsession
rückwirkend aus dem vollständigen Arbeitskontext und den entstandenen
Dokumenten. Künftige Berichte werden während der jeweiligen Session
fortlaufend gepflegt.
