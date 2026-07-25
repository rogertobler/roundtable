# Roundtable

Roundtable ist ein lokaler, plattformübergreifender Multi-Agent-Chat-Router.
Ein gemeinsamer Messenger-Chat steuert mehrere gleichzeitig laufende
AI-Agenten in echten CLI-Sessions.

Telegram ist der erste Kommunikationskanal. Claude Code und OpenAI Codex sind
die ersten unterstützten Agenten. Jede Session besitzt ihren eigenen Agenten,
ihr eigenes Modell, Projekt und eine dauerhafte tmux-Session.

```text
Reply auf eine Agentennachricht -> Ursprungssession
Freie Nachricht                -> Default-Session
```

Dadurch können beispielsweise eine Claude-Session und eine Codex-Session
gleichzeitig über denselben privaten Telegram-Chat bedient werden, ohne
Nachrichten manuell einer Session zuordnen zu müssen.

Roundtable führt keinen eigenen Agentendialog. Es liest Ausgaben aus der echten
CLI-Session und schreibt Benutzerantworten in genau diese Session zurück. Wer
die tmux-Session lokal öffnet, sieht denselben nativen Chatverlauf und kann dort
nahtlos weiterschreiben. Lokales Terminal und Messenger schreiben dabei
kontrolliert abwechselnd, nie gleichzeitig in dasselbe Pane.

## Status

Roundtable befindet sich in der Planungs- und Architekturphase. Die
Produktvision, Anforderungen, Bedienabläufe, technische Architektur,
Plattformstrategie und Roadmap sind ausführlich dokumentiert.

## Dokumentation

Der Einstieg befindet sich unter [docs/README.md](docs/README.md).

Besonders relevant:

- [Produktvision](docs/01-product-vision.md)
- [Funktionale Anforderungen](docs/02-functional-requirements.md)
- [Telegram-Bedienkonzept](docs/03-telegram-ux.md)
- [Technische Architektur](docs/04-architecture.md)
- [Agenten-CLIs und Session-Tunnel](docs/06-agents-and-runtimes.md)
- [Roadmap](docs/08-roadmap.md)
- [Entscheidungen und offene Fragen](docs/09-decisions-and-open-questions.md)
- [Entwicklungs-Sessions](docs/sessions/README.md)

## Zentrale Prinzipien

- Agent und Modell werden pro Session festgelegt.
- Claude-, Codex- und weitere Agentensessions können parallel laufen.
- Der Messenger-Chat ist die Roundtable-Benutzeroberfläche.
- Die native CLI-Session ist die einzige laufende Agentenkonversation.
- Roundtable und die Agenten arbeiten host-nativ auf derselben Codebasis, mit
  der der Benutzer auch direkt am Rechner arbeitet.
- Roundtable ist ein inhaltstreuer Tunnel zwischen Chat und CLI-Session.
- Replies werden über persistente Nachrichten-IDs zur Ursprungssession
  geroutet.
- Freie Nachrichten gehen ausschließlich an eine explizite Default-Session.
- Die erste erfolgreich erstellte Session wird initial Default. Spätere
  Sessionstarts ändern diesen Default nicht; nur der Benutzer kann ihn
  wechseln.
- Roundtable schreibt Nachrichten und Freigaben nicht um.
- Ein attachter lokaler tmux-Client sperrt mobile Writes, bis der Benutzer
  lokal detached oder eine kontrollierte mobile Übernahme bestätigt.
- Eine nach einem Teilabsturz unklare Pane-Zustellung wird nicht automatisch
  wiederholt.
- Agenten-APIs, Hooks und strukturierte Events dürfen höchstens optionale
  Erkennungshilfen sein und bilden keinen zweiten Nachrichtenpfad.
- Linux, macOS und Windows sind Zielplattformen.
- Der Grundbetrieb bleibt local-first und benötigt keinen Roundtable-Cloud-
  Dienst.
- Docker ist kein Roundtable-Installations- oder Runtime-Modell.

## Lizenz

Die Projektlizenz ist noch nicht festgelegt.
