# Acervo de Estudos: Engenharia de Software Backend

Acervo de estudos em Markdown sobre engenharia de software backend, do fundamento ao avançado, com foco em **.NET / C#**. O material serve a dois usos: leitura para entender e revisão ativa para reter, com perguntas de auto-teste em cada tópico.

A prioridade é **confiabilidade e correção técnica acima de volume**. Todo conteúdo é ancorado em fontes autoritativas (Microsoft Learn, documentação oficial de PostgreSQL, RabbitMQ, Kafka, Kubernetes, Redis, MongoDB) e o código é idiomático em .NET 8 e C# 12.

## Como usar

1. **Leia para entender.** Comece pelo `_index.md` de cada área, que traz a ordem de leitura sugerida (do mais simples ao mais avançado).
2. **Revise para reter.** Use a seção `## Perguntas de auto-teste` de cada arquivo como recuperação ativa: responda antes de abrir o bloco com a resposta.
3. **Navegue pelas conexões.** Cada tópico declara `prerequisites` e `related` no frontmatter e usa links relativos no corpo, formando um mapa de tópicos.

## Como o acervo é construído

Uma descrição de vaga (job description) é usada como entrada para identificar as áreas de conhecimento relevantes. Cada área vira uma pasta e cada tópico vira um arquivo Markdown. As vagas são insumo, não artefato: não ficam guardadas no repositório.

## Estrutura

Uma pasta por área, um arquivo por tópico, em kebab-case. Cada arquivo segue um template comum (resumo, explicação detalhada, por baixo dos panos, exemplos em C#, tradeoffs, pegadinhas, quando usar, auto-teste, diagrama, referências).

| Área | Pasta | Foco |
| --- | --- | --- |
| 01 | [`01-csharp-dotnet`](01-csharp-dotnet/_index.md) | Fundamentos da linguagem e runtime: threads/ThreadPool, concurrency/parallelism, async/await, Task vs ValueTask, IQueryable, GC, records, generics, .NET 8 / C# 12 new features, pipeline and middleware |
| 02 | [`02-microservices-patterns`](02-microservices-patterns/_index.md) | Microservices patterns: saga, outbox, idempotency, circuit breaker, CQRS, API gateway, classic design patterns |
| 03 | [`03-ef-dapper-postgresql`](03-ef-dapper-postgresql/_index.md) | Acesso a dados: change tracking, N+1, transaction isolation, migrations, indexes e JSONB, Dapper vs EF, Essential SQL |
| 04 | [`04-unit-tests`](04-unit-tests/_index.md) | Unit tests: test doubles, AAA pattern e testing tools |
| 05 | [`05-messaging-distributed-systems`](05-messaging-distributed-systems/_index.md) | Messaging e distributed systems: delivery semantics, RabbitMQ vs Kafka, CAP theorem, dead letter queue |
| 06 | [`06-docker-k8s-cicd-azure`](06-docker-k8s-cicd-azure/_index.md) | Infra e delivery: image vs container, Kubernetes objects, CI/CD, Azure overview |
| 07 | [`07-quality-solid`](07-quality-solid/_index.md) | Code quality: SOLID, code smells, SonarQube |
| 08 | [`08-differentials`](08-differentials/_index.md) | Diferenciais: Redis, MongoDB, observability com Datadog, Gitflow |

## Ordem de estudo sugerida

Fundamentos antes de avançados. Uma trilha possível:

1. **01 csharp-dotnet** (base da linguagem e do runtime)
2. **07 quality-solid** (princípios que valem para todo o resto)
3. **03 ef-dapper-postgresql** (persistência)
4. **04 unit-tests** (garantia de comportamento)
5. **02 microservices-patterns** (arquitetura distribuída)
6. **05 messaging-distributed-systems** (comunicação assíncrona)
7. **06 docker-k8s-cicd-azure** (empacotamento e delivery)
8. **08 differentials** (tópicos que destacam o candidato)

## Convenções

- Português do Brasil, com termos técnicos consagrados mantidos em inglês (`deadlock`, `change tracking`, `deferred execution`).
- Código limpo e autoexplicativo, sem comentários dentro dos blocos, compilável em .NET 8 / C# 12.
- Frontmatter YAML em todo tópico, com `id`, `area`, `difficulty`, `prerequisites`, `related`, `tags`, `sources`, `status` e `last_updated`.
- Commits em Conventional Commits, em inglês (ex.: `docs: add review notes for async-await`).

Este repositório é a semente de um futuro app de estudos, por isso estrutura e metadados importam tanto quanto o texto.
