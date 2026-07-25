# Bestehende Lösungen und Abgrenzung

Stand der Recherche: 25. Juli 2026

Dieses Dokument ist keine vollständige Marktanalyse. Es hält Projekte fest, die
für Architektur, Bedienung oder Positionierung von Roundtable relevant sind.
Vor einer konkreten Übernahme von Code, Protokollen oder Ideen müssen Lizenz,
aktueller Entwicklungsstand und Sicherheitsmodell erneut geprüft werden.

## HeyAgent

Links:

- <https://www.heyagent.dev/>
- <https://github.com/gergomiklos/heyagent>

Relevanz:

- Telegram-Brücke für Claude Code und Codex,
- lokaler beziehungsweise selbst kontrollierter Betrieb,
- Nachrichten und Antworten,
- Starten oder Fortsetzen von Sessions,
- Providerwechsel und Anhänge.

Abgrenzung:

Roundtable stellt nicht einen global gewählten Provider in den Mittelpunkt,
sondern eine zentrale Inbox aus vielen gleichzeitig laufenden Sessions. Der
Agent ist pro Session festgelegt. Reply-Routing muss Claude-, Codex- und spätere
Agentensessions innerhalb desselben Chats zuverlässig trennen.

Zu prüfen:

- Start- und Resume-Strategien,
- Telegram-Rendering,
- Agentenerkennung,
- Lizenz und wiederverwendbare Komponenten.

## ccgram

Link:

- <https://github.com/alexei-led/ccgram>

Relevanz:

- engster bekannter Wettbewerber,
- Telegram-zu-tmux/herdr-Tunnel,
- Claude Code, Codex, Gemini, Pi und Shell gleichzeitig,
- parallele Sessions,
- Output-Monitoring und Tasteneingaben,
- Freigabeprompts und optionale Claude-Hooks,
- Session Recovery, Worktrees, Voice und Web-Dashboard,
- Telegram Forum Topics als Sessionmodell.

Zum Prüfzeitpunkt:

- Release `v4.3.11`,
- aktiv gepflegt,
- MIT-Lizenzdatei im Repository.

Abgrenzung:

ccgram bestätigt, dass ein dünner Tunnel über echte Multiplexer-Sessions
praktisch funktioniert. Roundtable verwendet denselben fundamentalen Gedanken,
definiert aber eine andere Chat-UX:

- ein einziger privater Bot-Chat als gemeinsame Inbox,
- keine Telegram-Gruppe, Forum Topics oder Bot-Adminrechte als Voraussetzung,
- persistentes Reply-Mapping,
- Reply auf Agentennachricht routet zur Ursprungssession,
- freie Nachricht routet zur Default-Session,
- erste Session wird initialer Default,
- Telegram ist nur erster Transport; WhatsApp soll denselben Router nutzen,
- Windows via WSL/tmux und später natives Session Backend,
- formalisierte Projekt-, Benutzer- und Pfadgrenzen.

Die Differenzierung ist damit nicht „Telegram steuert tmux“, sondern:

> Ein gemeinsamer Multi-Agent-Chat multiplexiert viele echte CLI-Sessions über
> Reply-Routing und Default-Session.

Zu prüfen:

- Architektur und Tests von `pipe-pane`/`capture-pane`,
- Hook-Nutzung,
- Approval-Zuordnung,
- Sessionpersistenz,
- Umgang mit Telegram-Limits,
- welche MIT-lizenzierten Teile sinnvoll wiederverwendbar sind,
- ob Fork, Teilübernahme oder eigenständige Implementierung langfristig
  sinnvoller ist.

## Codex Telegram Bridge

Link:

- <https://github.com/ssamssae/codex-telegram-bridge>

Relevanz:

- Telegram-Fernsteuerung einer sichtbaren Codex-TUI in tmux,
- Messengertext landet in derselben sichtbaren CLI-Session,
- Codex-JSONL als strukturierter Hinweis auf User- und Final-Answer-Ereignisse,
- `capture-pane` für sichtbare Approvals und Auswahlmenüs,
- Cursorpersistenz, Restart-Backfill und Benutzer-Services,
- WSL/tmux sowie ein eingeschränkter nativer Windows-Exec-Modus.

Abgrenzung:

Das Projekt ist bewusst Codex-spezifisch und steuert eine Codex-Session. Es
besitzt nicht Roundtables gemeinsame Inbox mit Reply-Routing zwischen mehreren
gleichzeitigen Claude-, Codex- und späteren Agentensessions.

Zu prüfen:

- Trennung von JSONL-Ereignissen und sichtbarem TUI-Prompt,
- Eingabe- und Composer-Behandlung,
- Echo-, Cursor- und Restart-Logik,
- Approval-/Picker-Erkennung,
- plattformspezifische Dienstinstallation.

## cc-telegram-bridge

Link:

- <https://github.com/cloveric/cc-telegram-bridge>

Relevanz:

- Claude Code und Codex CLI über Telegram,
- mehrere isolierte Bot-/Agenteninstanzen,
- Session Resume,
- Agent-Bus und Multi-Agent-Workflows,
- unterschiedliche Approval-Semantik der verwendeten CLI-Modi.

Abgrenzung:

Das Projekt führt Agenten über native CLI-Harnesses und getrennte
Botinstanzen. Roundtable definiert einen gemeinsamen Chat, in dem die
Ursprungsnachricht per Reply-Mapping die konkrete Session auswählt. Für Codex
verwendet Roundtable bewusst die interaktive TUI im tmux-Tunnel und nicht den
einmaligen `codex exec`-Pfad.

Zu prüfen:

- Session- und Queue-Lebenszyklus,
- Recovery- und Kostenbegrenzung,
- Agent-zu-Agent-Audit,
- Unterschiede zwischen interaktiver CLI und Exec-Modus.

## amux

Link:

- <https://amux.cc/>

Relevanz:

- Multiplexer für Agenten in tmux,
- Claude Code, Codex und konfigurierbare weitere Agenten,
- parallele lokale Terminalarbeit.

Abgrenzung:

amux fokussiert die lokale Terminal-/TUI-Orchestrierung. Roundtable fokussiert
die mobile, kanalübergreifende Inbox und das nachrichtenbasierte Routing zu
dauerhaften Sessions.

Zu prüfen:

- Agentenabstraktion,
- Session Discovery,
- Multiplexing und Statusdarstellung,
- lokale Bedienung paralleler Agenten.

## dmux

Link:

- <https://dmux.ai/>

Relevanz:

- parallele Agentensessions,
- tmux,
- Git-Worktrees,
- strukturierte Arbeit an mehreren Aufgaben.

Abgrenzung:

dmux ist vor allem für lokale Orchestrierung und parallele Worktrees relevant.
Roundtable ergänzt eine mobile Inbox, Reply-Routing, Transportadapter,
Default-Session und plattformübergreifende Fernsteuerung.

Zu prüfen:

- Worktree-Lebenszyklus,
- Branch-Namenskonventionen,
- Session-zu-Projekt-Zuordnung,
- parallele Agenten-UX.

## Claude Code Remote Control

Link:

- <https://code.claude.com/docs/en/remote-control>

Relevanz:

- offizieller Remotezugriff auf lokale Claude-Code-Sessions,
- Bedienung über Browser oder mobile Oberfläche,
- bestehender Claude-Prozess und Dateizugriff bleiben auf dem lokalen Rechner,
- synchronisierter Transcript wird laut offizieller Dokumentation über
  Anthropic-Dienste gespeichert.

Abgrenzung:

Die Funktion ist Claude-spezifisch. Roundtable soll Claude Code, Codex und
weitere Terminal-Agenten gleichzeitig über eine gemeinsame Inbox steuern und
ist nicht an das Ökosystem eines einzelnen Agentenanbieters gebunden.

Zu prüfen:

- Erwartungen der Benutzer an mobile Sessionsteuerung,
- Sicherheits- und Pairingmodell,
- Verhalten bei Verbindungsverlust,
- mögliche Überschneidungen mit nativen Agentenfunktionen.

## Claude Code Channels

Link:

- <https://code.claude.com/docs/en/channels>

Relevanz:

- offizieller Telegram-/Discord-/iMessage-Kanal für eine laufende
  Claude-Code-Session,
- lokaler Claude-Kontext,
- Pairing und Sender-Allowlist,
- optionale Weiterleitung von Freigaben.

Abgrenzung:

Channels ist Claude-spezifisch und befindet sich in Research Preview.
Roundtable verbindet mehrere Claude-, Codex- und weitere CLI-Sessions in einem
gemeinsamen Chat. Channels ist ein alternativer direkter Nachrichtenpfad und
daher kein Bestandteil des Roundtable-Tunnels. Die Implementierung ist als
Referenz für Pairing, Sender-Allowlist und Messenger-UX relevant. Davon
getrennte Claude-Hooks können optional als Sensoren dienen, ohne Nachrichten an
tmux vorbeizuführen.

## Technische Referenzen

### tmux Control Mode

Links:

- <https://github.com/tmux/tmux/wiki/Control-Mode>
- <https://man.openbsd.org/tmux>

Control Mode liefert strukturierte tmux-Kommandorückgaben und
Lebenszyklusereignisse. `%output` enthält dennoch rohe Pane-Bytes und keine
Agentenantwortgrenzen. Roundtable vergleicht ihn im Spike mit `pipe-pane`,
statt ihn ungeprüft als vollständige Outputlösung festzulegen.

tmux User Options, stabile Session-/Pane-IDs, `pane_dead`, `pane_pid`,
Client-Metadaten und kontrollierte Buffer-Pastes sind direkte Bausteine für
Reattach und Eingabesicherheit.

### Codex App Server

Link:

- <https://github.com/openai/codex/blob/main/codex-rs/app-server/README.md>

Der App Server besitzt einen strukturierten JSON-RPC-Dialog einschließlich
serverinitiierter Approval-Anfragen. Er widerlegt die pauschale Annahme, Codex
könne nur pro Turn Freigaben vermitteln. Roundtable verwendet ihn trotzdem
nicht als Hauptpfad, weil dann nicht mehr die echte interaktive Codex-TUI in
tmux der einzige Gesprächsverlauf wäre. Er bleibt Referenz für Zustände und
Approval-Lebenszyklen.

### Terminalemulation

Links:

- <https://github.com/asciinema/asciinema>
- <https://github.com/asciinema/avt>

Asciinema ist primär Recorder, Streamer und Player. Die dazugehörige
Virtual-Terminal-Implementierung `avt` ist für die Screen-Rekonstruktion
direkter relevant. Vor einer Nutzung sind Lizenz, Rust-/Go-Integration,
Alternate Screen, Unicode und Resize-Verhalten zu prüfen.

### Go-tmux-Wrapper

Link:

- <https://github.com/owenthereal/tmux>

Der Wrapper kann als API-Referenz untersucht werden. Sein geringer
Release-/Nutzungsnachweis reicht jedoch nicht für die Vorentscheidung
„produktionsreif“. Ein kleiner interner, streng typisierter tmux-Adapter kann
weniger Risiko tragen, insbesondere wenn Control Mode separat implementiert
werden muss.

## Schlussfolgerung

Einzelne Teile der Roundtable-Idee existieren bereits:

- Telegram-Brücken,
- Claude-Fernsteuerung,
- tmux-basierte Agentensessions,
- parallele Agentenmultiplexer,
- Worktree-Orchestrierung.

Die eigenständige Kombination von Roundtable ist:

```text
gemeinsame Messaging-Inbox
  + mehrere parallele Sessions
  + Agent und Modell pro Session
  + Reply-Routing zur Ursprungssession
  + erste Session als initialer Default
  + manuell änderbare Default-Session für freie Nachrichten
  + inhaltstreue Freigaben und Antworten
  + echter nativer CLI-Verlauf in jeder Session
  + austauschbare Transport- und Session-Backends
  + Linux, macOS und Windows
```

Diese Kombination ist die Produktgrenze. Roundtable sollte bestehende
Open-Source-Projekte als technische Referenz nutzen. Insbesondere darf
ccgrams Topic-pro-Session-Modell nicht unbemerkt Roundtables zentralen
Reply-Router ersetzen.

## Vorgehen bei weiterer Recherche

Für jedes relevante Projekt soll eine kurze technische Bewertung angelegt
werden:

- unterstützte Agenten,
- Sessionmodell,
- Output Capture,
- Eingaberouting,
- Approval-Behandlung,
- Persistenz,
- Plattformen,
- Sicherheitsmodell,
- Lizenz,
- wiederverwendbare Komponenten,
- zuletzt geprüfte Version und Datum.

Rechercheergebnisse beeinflussen die Implementierung, nicht die verbindlichen
Roundtable-Invarianten. Insbesondere bleiben Agent-pro-Session,
Reply-Priorität, initialer Default, echter CLI-Tunnel und keine geratenen
Zustellungen bestehen.
