# Host-native Workspace-Parität

Datum: 25. Juli 2026

## Ausgangslage

Die Architektur war bereits local-first und sah tmux sowie lokal installierte
Agenten-CLIs vor. Die Betriebsdokumentation erwähnte Docker dennoch als
möglichen VPS-Installationsweg, ohne deutlich genug festzuhalten, dass der
zentrale Roundtable-Workflow dieselbe Codebasis und Entwicklungsumgebung wie
die direkte Arbeit am Rechner verwenden muss.

## Entscheidung

- Roundtable startet Agenten standardmäßig host-nativ.
- Roundtable-Sessions und direkte lokale Arbeit verwenden denselben
  freigegebenen Projektpfad oder Git-Worktree.
- Lokal installierte Agenten-CLIs, Authentifizierung, Git-Konfiguration und
  Entwicklungswerkzeuge werden weiterverwendet.
- Docker ist kein Roundtable-Installations- oder Runtime-Modell.
- Isolation verwendet agenteneigene Sandboxfunktionen, Betriebssystemrechte
  und getrennte Worktrees.

## Begründung

Der Benutzer soll ohne Kopieren oder Synchronisieren zwischen Roundtable und
direkter Rechnerarbeit wechseln können. Änderungen eines Agenten müssen sofort
im lokalen Editor und Git-Worktree sichtbar sein; lokale Änderungen müssen
umgekehrt derselben Agentensession zur Verfügung stehen.

Ein Pflichtcontainer würde Projekt-Mounts, Agenten-Credentials, SSH, Git,
lokale Entwicklungswerkzeuge und direktes tmux-Attach unnötig komplizieren und
damit das zentrale Produktversprechen schwächen.

## Auswirkungen

- Installer und `roundtable doctor` prüfen die Hostumgebung.
- Projekt-Allowlist verweist auf reale lokale Pfade.
- Dokumentierte Sicherheitsprofile dürfen die host-native Standardsemantik
  nicht still verändern.
- VM oder WSL bleiben mögliche Hostumgebungen, aber Roundtable selbst fügt
  keine Container-Runtime hinzu.

## Betroffene Dokumente

- `README.md`
- `docs/README.md`
- `docs/01-product-vision.md`
- `docs/02-functional-requirements.md`
- `docs/03-telegram-ux.md`
- `docs/04-architecture.md`
- `docs/06-agents-and-runtimes.md`
- `docs/07-security-and-operations.md`
- `docs/08-roadmap.md`
- `docs/09-decisions-and-open-questions.md`
- `docs/11-ai-review-brief.md`
- `docs/12-ai-second-opinion-request.md`
- `docs/sessions/2026-07-25-001-product-definition-and-architecture.md`
