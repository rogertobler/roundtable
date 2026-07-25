# Agenten und Terminal-Runtimes

## Zentrale Regel

Der Agent ist eine Eigenschaft der Session.

```text
Session A -> Claude Code -> Modell Opus
Session B -> Codex       -> konfiguriertes GPT-Modell
Session C -> Claude Code -> Modell Sonnet
```

Alle Sessions können gleichzeitig laufen. Der Telegram-Transport behandelt sie
gleich; Unterschiede werden ausschließlich durch Sessiondaten, Agent-Adapter
und Runtime-Fähigkeiten abgebildet.

## Trennung von Agent und Runtime

Agent und Runtime lösen verschiedene Probleme:

- Der **Agent-Adapter** weiß, wie Claude Code oder Codex gestartet und erkannt
  wird.
- Das **Runtime-Backend** weiß, wie ein interaktiver Terminalprozess auf einem
  Betriebssystem gehalten und bedient wird.

Claude Code kann daher in tmux, einem nativen Session Host oder später in einem
Container laufen, ohne dass das Telegram-Routing geändert wird.

## AgentAdapter-Vertrag

```text
AgentAdapter
  type() -> AgentType
  displayName() -> string
  detectInstallation(context) -> InstallationInfo
  inspectCapabilities(context) -> AgentCapabilities
  validateConfig(sessionSpec) -> ValidationResult
  buildLaunchSpec(sessionSpec) -> ProcessLaunchSpec
  buildResumeSpec(session, previousRuntime) -> ProcessLaunchSpec
  classifyOutput(outputContext) -> AgentEvent[]
  detectInteraction(screen, recentOutput) -> InteractionCandidate?
  validateInteraction(candidate, currentScreen) -> bool
  commands(session) -> AgentCommand[]
  normalizeStatus(agentEvidence) -> SessionInteractionStatus
```

## AgentCapabilities

Ein Adapter meldet Fähigkeiten, statt dass der Core Agentennamen abfragt:

```text
AgentCapabilities
  supportsModelSelection
  knownModels
  acceptsInitialPrompt
  supportsResume
  supportsNativeHooks
  supportsApprovalDetection
  supportsStructuredEvents
  supportsCostCommand
  supportsCompactCommand
  supportsClearCommand
  supportsFileInput
  supportsImageInput
```

Unbekannte oder versionsabhängige Fähigkeiten bleiben deaktiviert, bis sie
erkannt oder konfiguriert wurden.

## Claude-Code-Adapter

Geplanter Umfang:

- lokale Installation und Version erkennen,
- Modellparameter pro Session setzen,
- projektbezogene Startargumente erzeugen,
- bestehende Claude-Sitzungen soweit unterstützt fortsetzen,
- bekannte Bereitschafts-, Arbeits- und Fragezustände erkennen,
- Approval-Prompts erkennen und Optionen auf Eingaben abbilden,
- optionale native Hooks nutzen, ohne sie zur einzigen Datenquelle zu machen,
- agentenspezifische Kommandos wie `/compact`, `/clear` und `/cost` anbieten,
- Änderungen zwischen Claude-Versionen über Capability Detection abfangen.

Der Adapter darf sichtbare Claude-Texte nicht neu formulieren. Er darf
Metadaten ergänzen und bekannte Terminaloptionen als Telegram-Buttons
darstellen.

## Codex-Adapter

Geplanter Umfang:

- Installation und Version erkennen,
- Modell und weitere erlaubte Sessionoptionen setzen,
- Projekt- und Sandboxkonfiguration pro Session berücksichtigen,
- Resume-Funktionen verwenden, soweit von der installierten Version angeboten,
- Fragen, Freigaben, Abschluss und Fehler erkennen,
- Approval-Auswahlen auf konkrete Terminaleingaben abbilden,
- agentenspezifische Befehle oder Modi als deklarierte Fähigkeiten anbieten.

Auch hier darf Roundtable nicht voraussetzen, dass alle Versionen dieselben
Prompts oder Tasten verwenden.

## Generic-Terminal-Adapter

Ein generischer Adapter ermöglicht beliebige interaktive Programme:

- konfigurierbarer Startbefehl,
- Arbeitsverzeichnis,
- erlaubte Umgebungsvariablen,
- Terminalausgabe,
- Texteingabe und Grundtasten,
- manuelle Statusführung.

Ohne spezialisierten Adapter gibt es keine verlässliche automatische
Approval-Erkennung. Der Benutzer kann aber Terminalansicht und direkte Eingabe
verwenden.

## Adapterregistrierung

Adapter werden über eine Registry geladen:

```text
AgentRegistry
  claude-code -> ClaudeCodeAdapter
  codex       -> CodexAdapter
  generic     -> GenericTerminalAdapter
```

Spätere externe Plugins benötigen:

- versionierte Adapter-API,
- isolierte Konfiguration,
- Capability Declaration,
- sichere Prozessgrenzen,
- Signatur- oder Vertrauensmodell.

Eine dynamische Plugin-API ist keine Voraussetzung für die erste Version. Die
internen Adaptergrenzen sollen sie jedoch nicht verhindern.

## Startbefehl und Shell-Sicherheit

Der Adapter liefert keinen unkontrollierten zusammengesetzten Shell-String,
sondern:

```text
ProcessLaunchSpec
  executable
  arguments[]
  workingDirectory
  environmentReferences[]
  terminalSize
  startupTimeout
```

Runtime-Backends starten Executable und Argumente strukturiert. Eine Shell wird
nur verwendet, wenn ein ausdrücklich konfiguriertes Profil dies verlangt.

Im Auditlog wird eine maskierte, menschenlesbare Darstellung gespeichert.
Secretwerte werden niemals Teil dieser Darstellung.

## Runtime-Backend-Vertrag

Ein Runtime-Backend muss mindestens können:

- Verfügbarkeit prüfen,
- Session erzeugen,
- existierende verwaltete Sessions wiederfinden,
- Text schreiben,
- Tasten senden,
- Bildschirm erfassen,
- inkrementelle Ausgabe liefern,
- Prozessstatus prüfen,
- unterbrechen und stoppen,
- Metadaten für Wiederverbindung speichern.

Optionale Fähigkeiten:

- lokale Attach-Anweisung,
- mehrere Terminalclients erkennen,
- Prozess nach Router-Neustart weiterhalten,
- Fenstergröße ändern,
- Signalweiterleitung,
- Dateiupload in Sessionkontext,
- Remote-Node-Verbindung.

## Linux

### Erste Implementierung: tmux

tmux bietet:

- dauerhafte Sessions unabhängig vom Router,
- lokales Attach,
- `send-keys` für Eingaben,
- `capture-pane` für Bildschirm-Snapshots,
- `pipe-pane` oder Logdateien für fortlaufende Ausgabe,
- stabile Identifikation über eigene Namen und Metadaten.

Roundtable verwendet einen eigenen tmux-Namensraum und speichert die echte
tmux-ID. Anzeigenamen werden nicht direkt als einzige Runtime-ID verwendet.

Geplante Operationen:

```text
create        -> neue tmux-Session mit kontrollierter Shell/Agent
writeText     -> literal senden, keine Shell-Auswertung
sendKeys      -> definierte tmux key names
captureScreen -> capture-pane inklusive gewünschter History
streamOutput  -> pipe-pane Log plus persistierter Offset
interrupt     -> C-c an aktives Pane
stop          -> kontrolliertes Agentenende, dann tmux kill-session
attachHint    -> tmux attach-session -t <runtime-id>
```

Risiken:

- `capture-pane` allein verliert flüchtige Ausgaben,
- `pipe-pane` muss vor relevanter Ausgabe aktiv sein,
- Vollbild-TUIs erzeugen viele Cursoroperationen,
- lokales und Telegram-basiertes Schreiben kann sich überschneiden,
- Shell-Quoting darf Benutzereingaben nicht verändern.

Deshalb werden Raw Stream und Screen Snapshot getrennt behandelt.

### Später: eigener Unix Session Host

Ein eigener PTY-Host kann langfristig plattformübergreifend konsistentere
Semantik bieten. tmux bleibt dennoch ein wertvolles Backend für Benutzer, die
direktes Attach und maximale Prozesspersistenz wünschen.

## macOS

tmux ist technisch geeignet, wird aber nicht standardmäßig mit macOS
ausgeliefert. Der Installer kann:

- eine vorhandene tmux-Installation verwenden,
- tmux als klar benannte Voraussetzung installieren,
- oder später den eigenen PTY Session Host nutzen.

Der Roundtable-Daemon läuft als `launchd`-Benutzerdienst. Projektzugriffe müssen
macOS-Dateisystem- und Datenschutzberechtigungen berücksichtigen.

## Windows

### Native Unterstützung

Windows verwendet ConPTY für moderne interaktive Konsolenprogramme. Da ein
ConPTY-Kontext an seinen Hostprozess gebunden ist, sollte jede dauerhafte
Session von einem separaten `roundtable-session-host` gehalten werden.

Der Session Host:

- besitzt den ConPTY-Handle,
- startet den Agentenprozess,
- schreibt Raw Output in ein lokales Append-only-Log,
- bietet eine lokale Named Pipe für Eingaben und Status,
- bleibt bei einem Neustart des zentralen Routers am Leben,
- authentifiziert wiederverbindende Roundtable-Prozesse,
- beendet sich nur nach Sessionregel oder kontrollierter Stop-Aktion.

Damit ist die Lebensdauer einer Session nicht untrennbar mit der
Telegram-Verbindung gekoppelt.

### WSL-Unterstützung

Alternativ kann Roundtable Agenten innerhalb von WSL und tmux ausführen.
Roundtable muss deutlich zeigen:

- ob eine Session nativ oder in WSL läuft,
- welches Dateisystem und welcher Pfad verwendet wird,
- welcher Agent innerhalb dieser Umgebung installiert sein muss.

Windows-Pfade und WSL-Pfade dürfen nicht blind ineinander umgerechnet werden.

## Runtime-Auswahl

Ein Projekt oder Benutzer kann eine Priorität konfigurieren:

```text
Linux:  tmux -> unix-session-host
macOS:  tmux -> unix-session-host
Windows: conpty-session-host -> wsl-tmux
```

Die tatsächliche Auswahl wird vor dem Session-Start angezeigt und an der
Session gespeichert.

## Persistenz und Wiederverbindung

Nach einem Roundtable-Neustart:

1. Datenbank öffnen und unvollständige Transaktionen prüfen.
2. aktive Runtimeinstanzen laden.
3. Runtime-Backends nach ihren externen IDs fragen.
4. Identität des Runtimekontexts validieren.
5. Output-Cursor ab gespeicherter Position fortsetzen.
6. Status aktualisieren.
7. fehlende Runtimes als `disconnected`, nicht sofort als `stopped`, markieren.
8. Wiederherstellungsoptionen anbieten.

Eine neu entdeckte Runtime darf nur dann automatisch verbunden werden, wenn
eine kryptografische oder ausreichend starke lokale Zuordnung vorhanden ist.

## Import bestehender Sessions

Die Übernahme einer bereits laufenden Session ist backendabhängig:

### tmux

Roundtable kann Sessions auflisten und Bildschirm, Pane-Prozess sowie
Arbeitsverzeichnis anzeigen. Der Benutzer wählt:

- welche tmux-Session übernommen wird,
- welcher Agent darin läuft,
- welches Projekt zugeordnet wird,
- welcher Roundtable-Sessionname verwendet wird.

Historische Ausgabe vor Aktivierung von `pipe-pane` kann unvollständig sein.

### ConPTY

Beliebige fremde ConPTY-Sessions können normalerweise nicht sicher übernommen
werden. Import ist primär für bereits von einem Roundtable Session Host
verwaltete Sessions vorgesehen.

### Agent Resume

Falls die physische Runtime verloren ging, kann eine neue Runtime über die
agenteneigene Resume-Funktion an denselben Agentenkontext anknüpfen. Dies ist
eine neue Runtime-Generation und muss sichtbar vom bloßen Wiederverbinden
unterschieden werden.

## Approval-Erkennung

Es gibt drei Evidenzstufen:

1. **Strukturiertes Agentenereignis:** höchste Verlässlichkeit.
2. **Bekannter Prompt plus Screen-Fingerprint:** verwendbar für Buttons.
3. **Heuristische Textanalyse:** nur als Hinweis; direkte Texteingabe und
   Terminalansicht bevorzugen.

Ein Approval-Button ist nur aktiv, wenn die konkrete Terminalwirkung bekannt
ist. Bei Unsicherheit sendet Roundtable den Originalprompt als routbare
Nachricht ohne semantischen Button.

## Terminaleingaben

Eingaben werden als strukturierte Operationen modelliert:

```text
WriteText("prüfe zuerst die Tests")
SendKey(Enter)
SendKey(CtrlC)
SendKeys([ArrowDown, ArrowDown, Enter])
```

Text wird literal geschrieben. Er darf nicht als Shellbefehl auf der
Roundtable-Seite ausgewertet werden.

## Gleichzeitige lokale Bedienung

Roundtable verhindert nicht, dass ein Benutzer dieselbe tmux-Session lokal
öffnet. Es gelten folgende Regeln:

- Roundtable serialisiert seine eigenen Eingaben.
- Lokale Tastendrücke können nicht zuverlässig in dieselbe Queue gezwungen
  werden.
- Eine aktuelle Terminalansicht wird vor veralteten Approval-Aktionen geprüft.
- Die UI zeigt lokale Attach-Informationen, soweit erkennbar.
- Agenteninteraktionen können als `superseded` markiert werden, wenn sich der
  Bildschirm nach lokaler Eingabe geändert hat.

## Agent-zu-Agent-Workflows

Spätere Workflows verwenden normale Sessions und explizite Übergaben:

```text
Claude implementiert
  -> erzeugt Übergabeartefakt oder Diff
  -> Benutzer oder Regel bestätigt Übergabe
Codex reviewt
  -> Review wird als Nachricht/Artefakt an Claude-Session übergeben
Claude korrigiert
  -> gemeinsamer Abschluss
```

Agenten kommunizieren nicht über magische globale Kontexte. Jede Übergabe wird
als adressierte Nachricht mit Quelle, Ziel, Inhalt und Auditereignis
modelliert.
