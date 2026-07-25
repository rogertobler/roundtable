# Bestehende Lösungen und Abgrenzung

Stand der Recherche: Juli 2026

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

Der Vorgänger <https://github.com/six-ddc/ccbot> bleibt als historische
Referenz relevant.

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
- bestehender Claude-Kontext bleibt auf dem lokalen Rechner.

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
gemeinsamen Chat. Channels und Hooks können als technische Referenz oder
optionale Sensoren dienen, dürfen aber keinen zweiten Nachrichtenpfad neben der
nativen CLI erzeugen.

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
