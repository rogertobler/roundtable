# Umsetzungsroadmap

## Grundsatz

Roundtable soll den beschriebenen Gesamtumfang erreichen. Die Phasen reduzieren
nicht die Vision, sondern schaffen überprüfbare Zwischenschritte. Jede Phase
muss auf den stabilen Verträgen für Transport, Reply-Routing, Session-Tunnel
und dünne Agenten-CLI-Definitionen aufbauen.

## Phase 0: Technischer Spike

Ziel: Die risikoreichsten Terminal- und Routingannahmen praktisch bestätigen.

Umfang:

- minimale Telegram-Bot-Verbindung,
- aktuellen ccgram-Code und dessen tmux-Strategie technisch bewerten,
- eine tmux-Session starten,
- Claude Code und Codex jeweils testweise starten,
- Agenten-CLI jeweils direkt als Pane-Prozess ohne Shell-Fallback starten,
- Output über `pipe-pane` und tmux Control Mode vergleichend erfassen,
- Terminalbild über `capture-pane` lesen,
- feste virtuelle Terminalgröße konfigurieren und Attach/Detach testen,
- literal Text und Grundtasten über Buffer senden,
- Singleline, Multiline, Unicode, Backticks und große Eingaben prüfen,
- Bracketed Paste mit aktivierter und deaktivierter Anwendung testen,
- Eingabe-Echo von legitimer Agentenausgabe unterscheiden,
- Telegram-Nachrichten-ID einer Session zuordnen,
- Reply an die richtige Session senden,
- Claude und Codex gleichzeitig betreiben,
- typische Approval-Prompts beider Agenten aufzeichnen,
- Router neu starten und tmux-Session wiederfinden,
- Reattach mit tmux User Options, stabilen IDs und absichtlicher
  Namenskollision testen,
- Core während jeder Stufe einer Eingabezustellung hart beenden,
- Telegram-Eingabe nach lokalem `tmux attach` im nativen CLI-Verlauf sehen,
- lokale CLI-Eingabe beobachten und ihre Agentenantwort in Telegram sehen,
- gleichzeitige lokale/mobile Eingabe provozieren und die Schreibsperre prüfen,
- 30-minütige reale Claude-/Codex-Turns mit Spinnern, Toolausgaben,
  Auswahlmenüs und Ruhephasen als Fixtures aufzeichnen,
- fünf parallele Sessions gegen Telegram-Backpressure testen.

Abnahme:

- Eine Claude- und eine Codex-Session laufen gleichzeitig.
- Die erste erstellte Session wird automatisch Default.
- Die zweite Session verändert diesen Default nicht.
- Replies auf Nachrichten beider Sessions landen zehnmal hintereinander
  nachweislich in der richtigen tmux-Session.
- Freie Nachricht geht an die gespeicherte Default-Session.
- Nach Router-Neustart bleiben tmux-Prozesse aktiv und Zuordnungen bestehen.
- Raw Stream und Screen Snapshot sind praktisch unterscheidbar.
- Lokal und über Telegram ist derselbe native CLI-Verlauf sichtbar.
- `pipe-pane` und Control Mode sind anhand von Messdaten bewertet; die
  Collector-Entscheidung behauptet keine semantischen Nachrichtengrenzen.
- In 100 absichtlich konkurrierenden lokalen/mobilen Zustellungen werden keine
  Zeichen interleavt.
- Nach 20 Core-Abbrüchen während verschiedener Write-Stufen entsteht weder
  automatische Doppelzustellung noch stiller Verlust; unklare Fälle werden
  sichtbar als `delivery_uncertain` markiert.
- Eine beendete Agenten-CLI lässt keine Messenger-Eingabe an eine Shell fallen.
- Reattach lehnt eine gleichnamige oder manipulierte tmux-Session sicher ab.

Ergebnis:

- Terminaltranskripte als Testfixtures,
- validierte tmux-Kommandos,
- Entscheidung über Implementierungssprache,
- konkretisierte Transport-, CLI-Definitions- und Tunnelverträge,
- Entscheidung über Outputquelle, Terminalemulator und feste Standardgröße,
- dokumentierte, agenten- und versionsbezogene Eingabematrix,
- dokumentierte Entscheidung „übernehmen, wiederverwenden oder neu bauen“ für
  relevante ccgram-Komponenten.

## Phase 1: Nutzbarer Linux-MVP

Ziel: Roundtable kann von einem einzelnen Besitzer auf Linux oder einem
Linux-VPS täglich genutzt werden.

Umfang:

- installierbarer Roundtable-Daemon,
- privater Telegram-Bot,
- sicheres Einbenutzer-Pairing,
- SQLite-Migrationen,
- Projekt-Allowlist,
- host-native Nutzung derselben lokalen Codebasis und Worktrees,
- Claude-Code-CLI-Definition,
- Codex-CLI-Definition,
- Agent und Modell pro Session,
- tmux-Runtime,
- direkte Agentenprozesse ohne Shell-Fallback,
- Roundtable-UUIDs als tmux User Options und geprüfter Reattach,
- feste virtuelle Terminalgröße,
- mehrere parallele Sessions,
- geführter Session-Start,
- Session-Liste und Detailansicht,
- Reply-Routing,
- erste Session als initialer Default,
- manuell änderbare Default-Session,
- explizite Sessionauswahl ohne Default,
- Outputaufbereitung ohne inhaltliches Umschreiben,
- optionaler CLI-Snapshot,
- Text, Enter, Escape und `Ctrl+C`,
- persistente Session-Input-Queue mit Zustellungszustandsautomat,
- sichtbarer Zustand `delivery_uncertain` ohne automatischen Retry,
- Schreibsperre bei einem attachten interaktiven tmux-Client,
- Session starten, unterbrechen und stoppen,
- Abonnieren und Stummschalten,
- einfache Statusmeldungen,
- Rückfragen und Approval-Prompts als normale inhaltstreue Ausgabe,
- freie Antworten darauf über Reply-Routing 1:1 zurücktunneln,
- persistente Nachrichten- und Sessionzuordnungen,
- strukturierte Logs und `roundtable doctor`.

Abnahme:

- Mindestens fünf parallele Sessions funktionieren über mehrere Projekte.
- Claude und Codex können gleichzeitig bedient werden.
- Jede Session ist eine echte attachbare tmux-Session mit nativer CLI.
- Jede Reply wird über persistiertes Mapping geroutet.
- Ohne Reply wird ausschließlich die konfigurierte Default-Session verwendet.
- Telegram-Eingaben sind im lokal attachten CLI-Verlauf sichtbar.
- Lokale Eingaben und Telegram-Eingaben nutzen denselben Agentenkontext.
- Lokale und mobile Eingaben können nicht zeichenweise interleaven.
- Doppelte Telegram-Updates erzeugen keine doppelten Terminaleingaben.
- Ein Absturz während der Zustellung führt nie zu einem blinden Retry.
- Neustart des Core beendet tmux-Sessions nicht.
- Nicht autorisierte Telegram-Nutzer sehen keine Metadaten.
- Projekte außerhalb der Allowlist können nicht gestartet werden.
- Direktes lokales Arbeiten und Roundtable verwenden nachweislich dieselben
  Projektdateien und Git-Änderungen.
- Lange Ausgaben bleiben vollständig abrufbar.

## Phase 2: Robuste Inbox und Interaktionen

Ziel: Hohe Zuverlässigkeit bei langen Sessions, TUIs und instabilen
Netzverbindungen.

Umfang:

- Transactional Outbox,
- erweiterte Queue-Recovery- und Dead-Letter-Werkzeuge,
- Output-Cursor und Backlog-Recovery,
- per-Chat Rate Limiting und Backpressure,
- stabile Fortschrittsnachrichten,
- alle Benachrichtigungsmodi,
- Interaction Manager,
- Prompt-Fingerprints,
- idempotente Approval-Buttons,
- vollständige Grundtasten,
- Verlauf und Raw-Output-Dateien,
- Datei- und Diff-Versand,
- Secret-Filter,
- Retention und Bereinigung,
- Wiederholungsstrategien und Dead-Letter-Ansicht,
- bestehende tmux-Sessions kontrolliert übernehmen,
- lokales Attach komfortabel anzeigen.

Abnahme:

- Ein Telegram-Ausfall verliert keine stabile Agentennachricht.
- Veraltete Approval-Buttons senden keine Eingabe.
- Schnelle Terminalausgabe erzeugt keine unkontrollierte Nachrichtenflut.
- Raw Output kann mit Prüfsumme vollständig nachvollzogen werden.
- Datenbank- oder Queue-Fehler führen nicht zu stiller Zustellung.
- Outputstau stoppt die CLI nicht und bleibt vollständig lokal abrufbar.

## Phase 3: Git- und Projektkomfort

Ziel: Paralleles Arbeiten an realen Repositories wird ergonomisch und sicher.

Umfang:

- automatische Worktree-Erstellung,
- Branch-Namensregeln,
- bestehende Worktrees auswählen,
- Worktree-Status anzeigen,
- `dirty`, `ahead`, `behind` und Konfliktzustände anzeigen,
- sichere Archivierung,
- Git-Diff als Artefakt,
- Projektvorlagen,
- Agentprofile,
- Standardmodelle und Standardberechtigungen,
- projektbezogene Umgebungsreferenzen,
- GitHub-/GitLab-Verknüpfung als optionale Adapter.

Abnahme:

- Zwei Agenten können dasselbe Repository in getrennten Worktrees bearbeiten.
- Roundtable zeigt jederzeit den exakten Pfad und Branch.
- Stoppen einer Session löscht keinen Worktree automatisch.
- Worktree-Entfernung ist explizit, geprüft und auditierbar.
- Roundtable führt in dieser Phase keine automatischen Merges durch.

## Phase 4: macOS

Ziel: Gleichwertiger lokaler Betrieb auf macOS.

Umfang:

- signierbares Binary,
- `launchd`-Integration,
- Keychain-Secrets,
- tmux-Installation und Prüfung,
- Pfad- und Berechtigungstests,
- Update- und Diagnoseablauf,
- Installer beziehungsweise Homebrew-Paket.

Abnahme:

- Funktionsumfang des Linux-MVP läuft auf unterstützten macOS-Versionen.
- Dienst übersteht Logout/Reboot gemäß dokumentierter Betriebsart.
- Agentensessions bleiben bei Core-Neustart erhalten.
- Installation erfordert keine Arbeit am Roundtable-Quellcode.

## Phase 5: Windows

Ziel: Zuerst ein klar unterstützter WSL/tmux-Modus, danach native
Windows-Nutzung mit derselben Tunnel-Semantik.

Umfang:

- WSL-Distribution erkennen und auswählen,
- Roundtable, tmux und Agenten-CLIs innerhalb von WSL prüfen,
- Windows-zu-WSL-Setup und Attach-Anleitung,
- eindeutige WSL-Projektpfade,
- anschließend ConPTY Session Host,
- Named-Pipe-Protokoll,
- persistente Output-Logs,
- Core-Wiederverbindung,
- Windows-Dienst oder Benutzer-Autostart,
- Credential Manager/DPAPI,
- Windows-Pfadmodell,
- Claude/Codex-Erkennung unter Windows,
- signierter Installer.

Abnahme:

- Claude und Codex können zunächst in parallelen WSL/tmux-Sessions laufen.
- Telegram-Eingaben sind beim Attach in WSL im nativen CLI-Verlauf sichtbar.
- Claude und Codex können nativ in parallelen ConPTY-Sessions laufen.
- Neustart des Core beendet separate Session Hosts nicht.
- Texte und Tasten werden ohne Shell-Interpretation übertragen.
- WSL- und native Pfade werden in der UI eindeutig unterschieden.
- Installation und Deinstallation hinterlassen keine unkontrollierten Dienste.

## Phase 6: Weitere Eingabe- und Transportarten

Ziel: Roundtable wird zu einer echten kanalunabhängigen Kommunikationsschicht.

Umfang:

- WhatsApp-Transport,
- Dateiempfang,
- Bilder und Screenshots an Agenten,
- Sprachnachrichten,
- lokale oder konfigurierbare Transkription,
- Transport-Capability-Modell,
- kanalübergreifende Default- und Reply-Semantik,
- konsistente Identitätsverknüpfung.

Abnahme:

- Dieselbe Session kann über Telegram und einen zweiten Transport erreicht
  werden.
- Reply-Routing bleibt pro Transport eindeutig.
- Transportlimits verändern nicht den Inhalt.
- Dateien werden vor Zustellung autorisiert und sicher gespeichert.

## Phase 7: Multi-Agent-Workflows

Ziel: Kontrollierte Zusammenarbeit mehrerer Sessions.

Umfang:

- Übergabe einer Nachricht oder eines Artefakts zwischen Sessions,
- plan -> implement -> review -> fix,
- konfigurierbare Bestätigung pro Übergabeschritt,
- Workflowzustände,
- Abbruch und Fortsetzung,
- gemeinsame Abschlusszusammenfassung auf explizite Anforderung,
- wiederkehrende und geplante Aufgaben,
- Review-Pipelines.

Abnahme:

- Jede Agent-zu-Agent-Nachricht besitzt Quelle, Ziel und Auditspur.
- Kein Agent darf ohne konfigurierte Regel beliebige Sessions bedienen.
- Benutzer kann vor jedem Übergabeschritt eingreifen.
- Workflowfehler stoppen nicht unabhängige Sessions.

## Phase 8: Mehrere Nodes, Web und Teams

Ziel: Roundtable skaliert über einen einzelnen Rechner und Benutzer hinaus.

Umfang:

- mehrere lokale oder entfernte Nodes,
- sichere Node-Identitäten,
- Web-Dashboard,
- mehrere Benutzer,
- Rollen und Projektberechtigungen,
- Session-Sharing,
- Kosten- und Tokenübersichten,
- Statistiken,
- Monitoring- und Deployment-Integrationen,
- mobile Pull-Request-Freigaben.

Abnahme:

- Sessionzugriff ist pro Benutzer und Projekt begrenzt.
- Node-Ausfall ist klar vom Agentenausfall unterscheidbar.
- Telegram und Web verwenden denselben Core und dieselben Invarianten.
- Auditdaten weisen Benutzer, Transport, Node und Session aus.

## Querschnittsarbeit

Jede Phase enthält:

- automatisierte Unit- und Integrationstests,
- Tests mit echten oder aufgezeichneten Agenten-TUIs,
- Datenbankmigrationen,
- Sicherheitsprüfung,
- dokumentierte Upgrade- und Recovery-Wege,
- plattformspezifische CI,
- Versions- und Capability-Matrix für Claude Code und Codex,
- verständliche Fehlermeldungen,
- aktualisierte Dokumentation.

## Teststrategie

### Unit-Tests

- Routingpriorität,
- Default-Session-Invarianten,
- Idempotenz,
- Statusnormalisierung,
- Berechtigungsentscheidungen,
- Outputsegmentierung,
- ANSI-Verarbeitung,
- Secret-Redaktion,
- Interaction-Zustandsautomat.

### Contract-Tests

Jeder Transport Adapter, jede CLI-Definition und jedes Session Backend muss die
zugehörige Contract-Suite bestehen.

### Transkript-Tests

Aufgezeichnete Ausgaben verschiedener Claude- und Codex-Versionen prüfen:

- Ready-Erkennung,
- Fragen,
- Approval-Optionen,
- Spinner,
- Vollbildänderungen,
- Abschluss,
- Fehler.

### Integrationstests

- echter lokaler Telegram-Testbot in separater Umgebung,
- echte tmux-Sessions,
- Neustart und Wiederverbindung,
- parallele Eingaben,
- Netzwerkunterbrechung,
- volle oder gesperrte Datenbank,
- veraltete Callbacks.

### End-to-End-Kernszenario

1. Claude-Session A starten.
2. Prüfen, dass A automatisch Default ist.
3. Codex-Session B starten und prüfen, dass A Default bleibt.
4. Agent A und B erzeugen je eine Nachricht.
5. Auf A antworten und Zustellung in die tmux-Session A prüfen.
6. Auf B antworten und Zustellung in die tmux-Session B prüfen.
7. Freie Nachricht senden und Zustellung in die Default-Session A prüfen.
8. Session B manuell als Default setzen.
9. Freie Nachricht senden und Zustellung in die Session B prüfen.
10. Beide Sessions lokal attachen und denselben CLI-Verlauf prüfen.
11. Core neu starten.
12. Reply- und Default-Routing erneut prüfen.
13. Eine veraltete Freigabe ablehnen, ohne Eingabe zu senden.

Dieses Szenario ist die dauerhafte Definition des zentralen Produktversprechens.

## Releasekriterien

Ein Release ist nicht bereit, wenn:

- eine Reply unter realistischen Bedingungen an die falsche Session gelangen
  kann,
- doppelte Updates doppelte Eingaben erzeugen,
- eine Messenger-Eingabe nicht im nativen CLI-Verlauf erscheint,
- lokales Attach einen anderen Agentenkontext zeigt,
- der Installer Secrets unsicher speichert,
- laufende Sessions beim normalen Core-Update unnötig beendet werden,
- eine Plattform nur durch nicht dokumentierte manuelle Entwicklungsschritte
  funktioniert,
- neue Agentenversionen ohne sicheren Fallback riskante Buttons erzeugen.
