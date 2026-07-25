# Roundtable

Roundtable ist eine lokale, plattformübergreifende Steuerungs- und
Kommunikationsebene für mehrere gleichzeitig laufende Terminal-Agenten.

Telegram ist der erste Kommunikationskanal. Claude Code und OpenAI Codex sind
die ersten unterstützten Agenten. Jede Session besitzt ihren eigenen Agenten,
ihr eigenes Modell, Projekt und Terminal.

```text
Reply auf eine Agentennachricht -> Ursprungssession
Freie Nachricht                -> Default-Session
```

Dadurch können beispielsweise eine Claude-Session und eine Codex-Session
gleichzeitig über denselben privaten Telegram-Chat bedient werden, ohne
Nachrichten manuell einer Session zuordnen zu müssen.

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
- [Agenten und Terminal-Runtimes](docs/06-agents-and-runtimes.md)
- [Roadmap](docs/08-roadmap.md)
- [Entscheidungen und offene Fragen](docs/09-decisions-and-open-questions.md)

## Zentrale Prinzipien

- Agent und Modell werden pro Session festgelegt.
- Claude-, Codex- und weitere Agentensessions können parallel laufen.
- Replies werden über persistente Nachrichten-IDs zur Ursprungssession
  geroutet.
- Freie Nachrichten gehen ausschließlich an eine explizite Default-Session.
- Roundtable schreibt Nachrichten und Freigaben nicht um.
- Linux, macOS und Windows sind Zielplattformen.
- Der Grundbetrieb bleibt local-first und benötigt keinen Roundtable-Cloud-
  Dienst.

## Lizenz

Die Projektlizenz ist noch nicht festgelegt.
