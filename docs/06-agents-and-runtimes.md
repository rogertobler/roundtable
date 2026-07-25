# Agenten-CLIs und Session-Tunnel

## Zentrale Regel

Der Agent ist eine Eigenschaft der Session, und jede Session ist ein Tunnel zu
einer echten nativen CLI.

```text
Session A -> tmux -> Claude Code -> Modell Opus
Session B -> tmux -> Codex       -> konfiguriertes Modell
Session C -> tmux -> Claude Code -> Modell Sonnet
```

Alle Sessions können gleichzeitig laufen. Der Benutzer bedient sie über einen
gemeinsamen Messenger-Chat. Eine Reply wählt die zugehörige Session; eine freie
Nachricht geht an den Default.

## Kein eigener Agenten-Client

Roundtable:

- startet die lokal installierte Agenten-CLI,
- liest deren echte Terminalausgabe,
- schreibt Benutzertext in dieselbe CLI,
- hält Routing- und Auditmetadaten,
- entscheidet keine Agentenantwort.

Roundtable:

- baut keinen separaten API-Chat auf,
- synchronisiert keine zwei Gesprächskontexte,
- ersetzt nicht die native Sessionhistorie,
- sendet Nachrichten nicht an tmux vorbei.

Die CLI bleibt Quelle des laufenden Agentenkontexts. Roundtable ist der Tunnel.

## Voraussetzungen pro Agent

Bevor eine Session gestartet werden kann, muss in derselben Umgebung verfügbar
sein:

- ausführbare CLI,
- gültige lokale Authentifizierung,
- unterstützte oder benutzerdefinierte Modellkonfiguration,
- Zugriff auf das freigegebene Projekt,
- notwendige lokale Tools,
- tmux beziehungsweise das gewählte Session Backend.

Beispiele:

```text
claude --version
codex --version
tmux -V
```

Roundtable übernimmt bestehende CLI-Anmeldungen und Konfigurationen. Tokens
werden nicht in die Roundtable-Projektkonfiguration kopiert.

## Agent Definition

Eine Agent Definition hält nur das Wissen, das zum Betrieb der CLI im Tunnel
nötig ist:

```text
AgentDefinition
  type()
  displayName()
  executableName()
  detectInstallation()
  detectVersion()
  detectAuthentication()
  modelOptions()
  validateConfig()
  buildLaunchCommand()
  buildResumeCommand()
  readinessHints()
  outputHints()
  knownInteractions()
  nativeCommands()
```

Der Startbefehl wird strukturiert modelliert:

```text
LaunchCommand
  executable
  arguments[]
  workingDirectory
  environmentReferences[]
  terminalColumns
  terminalRows
```

Roundtable baut keinen frei interpolierten Shellstring aus Messengerdaten.

## Capability-Modell

CLI-Versionen unterscheiden sich. Eine Agent Definition meldet erkannte
Fähigkeiten:

```text
AgentCapabilities
  supportsModelSelection
  supportsResume
  supportsNativeSlashCommands
  supportsFileInput
  supportsImageInput
  hasOptionalLifecycleHooks
  hasKnownApprovalLayout
  hasKnownTurnBoundaryHints
```

Capabilities verbessern Bedienung und Erkennung. Keine Capability darf den
Tunnelgrundsatz verändern.

## Claude-Code-Definition

Geplanter Umfang:

- Installation und Version erkennen,
- lokale Claude-Authentifizierung prüfen, ohne Secrets auszugeben,
- Modellargument pro Session setzen,
- Projektverzeichnis und erlaubte Startargumente verwenden,
- native Resume-Funktion einsetzen, wenn eine CLI-Session neu gestartet werden
  muss,
- sichtbare Bereitschaft und bekannte Interaktionen erkennen,
- bekannte Tastenauswahlen für Approvals beschreiben,
- native Befehle wie `/compact`, `/clear` und `/cost` anbieten,
- optionale Hooks für Turn-Ende, Status und Approval-Hinweise verwenden.

Normale Telegram-Nachrichten werden als Eingabe in die Claude-CLI geschrieben.
Auch wenn ein Hook existiert, stammt die an Telegram gesendete Antwort aus
derselben tmux-Ausgabe.

## Codex-Definition

Geplanter Umfang:

- Installation und Version erkennen,
- lokale Codex-Authentifizierung prüfen,
- Modell- und erlaubte Startargumente pro Session setzen,
- Sandbox- und Approvalmodus sichtbar konfigurieren,
- native Resume-Funktion einsetzen,
- Bereitschaft, Fragen, Ende und bekannte Approvals im CLI-Bild erkennen,
- native Kommandos und Grundtasten anbieten,
- optionale Ereignisquellen nur als Erkennungshilfe verwenden.

Roundtable verwendet für den Kernfluss nicht `codex exec` und führt keinen
separaten App-Server-Dialog. Telegram-Eingaben landen in der echten
interaktiven Codex-CLI.

## Weitere Agenten

Ein neuer Agent benötigt mindestens:

- ausführbaren Startbefehl,
- Arbeitsverzeichnis,
- Modellparameter, falls vorhanden,
- Bereitschaftshinweis oder konservatives Timeout,
- Output Capture über das Session Backend,
- literal Texteingabe und Grundtasten.

Ohne spezialisierte Erkennung funktioniert er als generische CLI-Session.
Status und Approval-Buttons sind dann Best Effort, der Reply-Tunnel bleibt
vollständig funktionsfähig.

Geplante Kandidaten:

- Gemini CLI,
- Aider,
- OpenCode,
- Pi,
- beliebige interaktive Shellprogramme.

## tmux als erstes Session Backend

tmux erfüllt die zentralen Anforderungen:

- echter dauerhafter Agentenprozess,
- unabhängig vom Roundtable-Core,
- lokales Attach,
- literal Eingaben und Tasten,
- aktueller Bildschirm,
- Scrollback,
- fortlaufendes Output Capture,
- mehrere parallele Sessions.

Roundtable verwendet einen eigenen Namensraum und eine unveränderliche
technische Runtime-ID. Der sichtbare Sessionname ist nicht die einzige
Identität.

Beispiel:

```text
Roundtable Session ID: 01J...
Anzeigename: Backend Review
tmux Runtime ID: roundtable-01J...
Agent: codex
Projekt: DumbleScore
```

## Sessionstart

Der Startablauf:

1. Benutzer wählt Agent, Modell, Projekt und Sessionname.
2. Roundtable prüft Allowlist, Arbeitsverzeichnis, tmux und CLI.
3. Roundtable erzeugt eine technische Runtime-ID.
4. Roundtable legt eine detached tmux-Session an.
5. Roundtable konfiguriert eine ausreichend große History.
6. Roundtable aktiviert den fortlaufenden Outputpfad.
7. Roundtable startet die CLI im Projektverzeichnis.
8. Roundtable prüft Prozess und Startbereitschaft.
9. Roundtable speichert die Sessionzuordnung.
10. Ist es die erste Session ohne vorhandenen Default, wird sie Default.
11. Eine optionale Startaufgabe wird literal in die CLI geschrieben.

Schlägt ein Schritt fehl, wird keine fälschlich laufende logische Session
angezeigt.

## Eingaben

Roundtable modelliert Eingaben getrennt:

```text
WriteLiteral("Prüfe zuerst die Tests")
SendKey(Enter)
SendKey(Escape)
SendKey(CtrlC)
SendKeys([ArrowDown, ArrowDown, Enter])
```

Für normale Messengernachrichten:

1. Text als Daten in einen tmux-Buffer laden.
2. Buffer in das Ziel-Pane einfügen.
3. Enter separat senden.

Dadurch werden Shell-Metazeichen nicht von Roundtable ausgewertet.

Mehrzeilige Nachrichten benötigen eine definierte Paste-Strategie, die mit den
unterstützten Agenten-CLIs getestet wird.

## Ausgaben

### Raw Stream

`tmux pipe-pane` schreibt den fortlaufenden Terminaldatenstrom in einen
kontrollierten lokalen Outputpfad. Roundtable verfolgt einen Byte- oder
Sequenzcursor.

### Screen State

`tmux capture-pane` liefert den aktuellen sichtbaren Zustand einschließlich
eines begrenzten Scrollbacks. Er dient:

- der optionalen CLI-Snapshot-Funktion,
- der Erkennung überschreibender TUI-Zeilen,
- der Prüfung von Approvals,
- der Wiederverbindung,
- der Diagnose.

### Native Agentenhistorie

Die Agenten-CLI selbst hält ihren Gesprächskontext. tmux zeigt den laufenden
nativen Verlauf. Roundtable hält zusätzlich Raw Logs, damit ältere Ausgabe auch
nach Erreichen der tmux-History-Grenze abrufbar bleibt.

Der Roundtable-Verlauf ist eine Routing- und Auditkopie, nicht der
maßgebliche Agentenkontext.

## Stabilisierung für Messenger

Ein Terminaldatenstrom enthält:

- ANSI-Farben,
- Cursorbewegungen,
- Spinner,
- Fortschrittsupdates,
- Vollbildneuzeichnungen,
- Eingabe-Echos.

Roundtable darf diese technischen Artefakte normalisieren. Es darf jedoch keine
Sätze umformulieren oder Inhalte durch ein LLM zusammenfassen.

Der Processor benötigt pro Session:

- Raw-Cursor,
- letzten Screen State,
- zuletzt gesendeten stabilen Bereich,
- Ruhefenster,
- optionale agentenspezifische Turn-Hinweise.

## Optionale Hooks

Hooks sind Sensoren, keine Transportstrecke.

Sie können signalisieren:

- Sessionstart,
- Turn-Ende,
- Rückfrage,
- Approval,
- Toolfehler,
- Agentenende.

Roundtable nutzt den Hinweis, um die zugehörige tmux-Ausgabe schneller oder
präziser zu lesen. Der Hooktext wird nicht als eigener paralleler
Agentenverlauf behandelt.

Wenn Hooks fehlen oder inkompatibel sind:

- normale Ein- und Ausgabe funktioniert weiter,
- Status kann weniger präzise sein,
- Approval-Buttons können entfallen,
- der Benutzer kann weiterhin per Reply oder CLI-Snapshot antworten.

## Approvals

Approvals werden aus der echten CLI-Session weitergeleitet:

1. Output oder optionaler Hook weist auf einen Prompt hin.
2. Roundtable liest den aktuellen Screen State.
3. Der unveränderte Prompt wird in den Messenger gesendet.
4. Reply-Routing bindet die Antwort an dieselbe Session.
5. Freitext wird literal in die CLI geschrieben.
6. Ein Button sendet ausschließlich eine bekannte Tastenfolge.
7. Vor Buttons wird geprüft, ob der Prompt noch aktuell ist.

Roundtable entscheidet nicht, ob eine Aktion erlaubt werden soll.

## Lokale und mobile Bedienung gleichzeitig

Der Benutzer darf dieselbe tmux-Session lokal attachen.

Folgen:

- lokale Eingaben erreichen unmittelbar den Agenten,
- Telegram-Eingaben erreichen denselben Prozess,
- Roundtable serialisiert nur seine eigenen Eingaben,
- lokale Eingaben können offene Telegram-Buttons veralten lassen,
- der Screen State wird vor einer Buttonaktion erneut geprüft,
- Antworten auf lokale Eingaben erscheinen ebenfalls im beobachteten Output.

Eine Session darf niemals dupliziert werden, nur weil der Benutzer sie lokal
öffnet.

## Wiederverbindung

Nach einem Roundtable-Neustart:

1. gespeicherte Session- und Runtime-IDs laden,
2. Existenz der tmux-Session prüfen,
3. primären Prozess und Arbeitsverzeichnis validieren,
4. Output Capture prüfen oder erneuern,
5. ab gespeichertem Cursor weiterlesen,
6. Status neu bestimmen.

tmux bleibt währenddessen aktiv.

Eine gleichnamige fremde tmux-Session darf nicht automatisch adoptiert werden.

## Bestehende Sessions übernehmen

Roundtable kann tmux-Sessions auflisten und für eine kontrollierte Übernahme
anzeigen:

- technische tmux-ID,
- Session-/Fenstername,
- aktueller Vordergrundprozess,
- Arbeitsverzeichnis,
- Bildschirmvorschau.

Der Benutzer bestätigt Agent, Projekt und Roundtable-Name. Historische Ausgabe
vor Aktivierung des Output Capture kann unvollständig sein.

## Stoppen und Fortsetzen

Stoppen ist abgestuft:

1. Agentennativer Exit, wenn sicher verfügbar.
2. `Ctrl+C` für eine laufende Aktion.
3. kontrolliertes Ende des CLI-Prozesses.
4. tmux-Session beenden nach Bestätigung.

Roundtable löscht Worktrees oder Logs nicht automatisch.

Wurde nur der Agentenprozess beendet, kann die Session mit dessen nativer
Resume-Funktion in derselben oder einer neuen Runtime-Generation fortgesetzt
werden. Dies wird sichtbar protokolliert.

## Linux

Linux ist die erste Zielplattform:

- tmux direkt,
- Roundtable als `systemd --user` Dienst,
- lokale Unix-Sockets,
- Agenten-CLIs im selben Benutzerkontext.

## macOS

macOS verwendet ebenfalls tmux:

- vorhandenes tmux nutzen oder als Voraussetzung installieren,
- Roundtable als `launchd` User Agent,
- Keychain für Roundtable-Secrets,
- Agenten-CLI und tmux unter demselben Benutzer.

## Windows

### Erste Unterstützung: WSL und tmux

Roundtable, tmux, Projekte und Agenten-CLIs laufen gemeinsam in WSL. Der
Installer beziehungsweise Setup-Assistent erklärt:

- welche WSL-Distribution verwendet wird,
- wo das Projekt liegt,
- dass die CLI innerhalb von WSL installiert und angemeldet sein muss,
- wie die tmux-Session aus Windows geöffnet wird.

### Native Unterstützung: ConPTY Session Host

Später ersetzt ein eigener Session Host tmux:

- hält ConPTY und Agentenprozess dauerhaft,
- liefert Raw Stream und Screen State,
- nimmt literal Text und Tasten über Named Pipe entgegen,
- bietet `roundtable attach <session>` als lokale native Ansicht,
- bleibt bei Core-Neustart aktiv.

Das Backend muss denselben Agentenverlauf lokal und im Messenger sichtbar
machen. Native Windows-Unterstützung ist kein App-Server-Sonderweg.

## Agent-zu-Agent-Übergaben

Spätere Workflows senden Nachrichten durch denselben Tunnel:

```text
Claude-Session erzeugt ein Review-Artefakt
  -> Roundtable adressiert eine Nachricht an Codex-Session
  -> Nachricht wird in Codex-CLI geschrieben
  -> Codex-Ausgabe erscheint in gemeinsamer Inbox
```

Quelle, Ziel und Inhalt werden protokolliert. Es entsteht kein unsichtbarer
Agentenkanal außerhalb der echten CLI-Sessions.
