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

## CCBot

Link:

- <https://github.com/six-ddc/ccbot>

Relevanz:

- Telegram-Steuerung für Claude Code,
- Verbindung zu tmux,
- Sitzungen und Verlauf,
- Benachrichtigungen,
- Freigabeprompts mit Inline-Interaktionen,
- Telegram Topics als Strukturierung.

Abgrenzung:

CCBot ist eine wichtige Referenz für Claude, tmux und Telegram. Roundtable
definiert jedoch:

- mehrere Agententypen als gleichwertige Sessioneigenschaft,
- eine gemeinsame private Inbox ohne Topics als notwendige Grundlage,
- persistentes Reply-Mapping,
- Default-Session für freie Nachrichten,
- plattformübergreifende Runtime-Backends einschließlich Windows,
- kanalunabhängigen Core.

Zu prüfen:

- tmux Output Capture,
- Hook-Nutzung,
- Approval-Zuordnung,
- Sessionpersistenz,
- Umgang mit Telegram-Limits.

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
  + Default-Session für freie Nachrichten
  + inhaltstreue Freigaben und Antworten
  + austauschbare Transport-, Agent- und Runtime-Adapter
  + Linux, macOS und Windows
```

Diese Kombination ist die Produktgrenze. Roundtable sollte bestehende
Open-Source-Projekte als technische Referenz nutzen, aber nicht deren
Claude-only-, Topic-only- oder Unix-only-Annahmen in den Core übernehmen.

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
Reply-Priorität und keine geratenen Zustellungen bestehen.
