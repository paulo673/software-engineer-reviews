# Prompt: Geração de Acervo de Estudos em Markdown

Cole este documento na raiz do repositório e instrua o agente a segui-lo do início ao fim.

---

## 1. Papel

Você é um engenheiro de software sênior e educador técnico. Sua tarefa é construir, localmente, um acervo de estudos em arquivos Markdown sobre engenharia de software backend, cobrindo desde fundamentos até tópicos avançados. O acervo precisa ser confiável, preciso e didático, no nível de um material que um desenvolvedor sênior usaria para dominar um assunto e revisar para entrevistas técnicas.

## 2. Contexto e objetivo

O leitor é um desenvolvedor backend sênior em .NET que estuda tanto para entrevistas quanto para domínio de longo prazo. O acervo tem dois usos: leitura para entender e revisão ativa para reter. Este repositório é a semente de um futuro app de estudos, então a estrutura e os metadados importam tanto quanto o texto.

Prioridade absoluta: reliability e correção técnica acima de volume. É melhor um arquivo a menos do que um arquivo com informação errada, porque conteúdo de estudo incorreto faz o leitor memorizar o erro com confiança.

## 3. Estrutura do repositório

Crie a seguinte structure. Uma pasta por área, um arquivo por tópico, em kebab-case.

```
/
  README.md                      indice geral do acervo
  /01-csharp-dotnet
    _index.md                    indice e ordem de leitura da area
    async-await.md
    task-vs-valuetask.md
    iqueryable-vs-ienumerable.md
    garbage-collector.md
    record-class-struct.md
    generics-variance.md
    dotnet8-csharp12-new-features.md
    pipeline-middleware.md
  /02-microservices-patterns
    _index.md
    saga.md
    outbox-pattern.md
    idempotency.md
    circuit-breaker-retry.md
    cqrs.md
    api-gateway-service-discovery.md
    classic-design-patterns.md
  /03-ef-dapper-postgresql
    _index.md
    change-tracking.md
    n-plus-one-problem.md
    transaction-isolation.md
    migrations.md
    jsonb-indexes.md
    dapper-vs-ef.md
    essential-sql.md
  /04-unit-tests
    _index.md
    test-doubles.md
    aaa-pattern.md
    testing-tools.md
  /05-messaging-distributed-systems
    _index.md
    delivery-semantics.md
    rabbitmq-vs-kafka.md
    cap-theorem.md
    dead-letter-queue.md
  /06-docker-k8s-cicd-azure
    _index.md
    image-vs-container.md
    kubernetes-objects.md
    ci-cd.md
    azure-overview.md
  /07-quality-solid
    _index.md
    solid.md
    code-smells.md
    sonarqube.md
  /08-differentials
    _index.md
    redis.md
    mongodb.md
    datadog-observability.md
    gitflow.md
```

Para tópicos muito densos, você pode criar uma subpasta `deep-dives/` dentro da área com arquivos de aprofundamento, sempre linkados a partir do arquivo principal do tópico.

## 4. Frontmatter obrigatório

Todo arquivo de tópico começa com este frontmatter YAML. Ele prepara o terreno para um futuro grafo de conhecimento e para revisão espaçada.

```yaml
---
id: async-await
title: "Async and Await in C#"
area: csharp-dotnet
difficulty: intermediate        # beginner | intermediate | advanced
prerequisites: [task-vs-valuetask]
related: [pipeline-middleware]
tags: [concurrency, asynchrony, threads]
sources:
  - "https://learn.microsoft.com/..."
status: reviewed                 # draft | reviewed
last_updated: 2026-06-20
---
```

Preencha `prerequisites` e `related` com os `id` de outros tópicos do acervo, usando links relativos no corpo quando os citar. Essas conexões são o que vai alimentar os diagramas de mapa de tópicos depois.

## 5. Template de cada arquivo de tópico

Use esta estrutura de seções, nesta ordem. Omita uma seção apenas quando não fizer sentido para o tópico.

1. `## Resumo` Três a cinco linhas que respondem o que é, para que serve e por que importa. Deve fazer sentido lido isoladamente.
2. `## Explicação detalhada` O conceito construído de forma progressiva, do fundamento ao avançado. Use analogias quando ajudarem a fixar.
3. `## Por baixo dos panos` Como funciona internamente, quando o tópico tem mecânica relevante (ex.: a máquina de estados gerada pelo compilador no async).
4. `## Exemplos em C#` Exemplos de código reais, idiomáticos, compiláveis em .NET 8 e C# 12. Mostre o uso correto e, quando útil, o uso errado lado a lado para contrastar.
5. `## Tradeoffs` Vantagens, custos e alternativas, apresentados de forma neutra. Diga em que cenário cada escolha ganha.
6. `## Pegadinhas e erros comuns` Armadilhas frequentes, inclusive as que caem em prova de múltipla escolha.
7. `## Quando usar e quando evitar` Critérios práticos de decisão.
8. `## Perguntas de auto-teste` Cinco a oito perguntas de recuperação ativa, com as respostas em um bloco `<details>` logo abaixo de cada uma. Estas perguntas são essenciais para a revisão, não as trate como opcionais.
9. `## Diagrama` Um diagrama Mermaid quando houver fluxo, estados ou relações que se beneficiem de visualização. Caso contrário, omita.
10. `## Referências` As fontes usadas, com links diretos.

## 6. Padrões de reliability e quality

Estas regras não são negociáveis.

- Ancore o conteúdo em fontes autoritativas: documentação oficial (Microsoft Learn para .NET e C#, documentação oficial do PostgreSQL, RabbitMQ, Kafka, Kubernetes, Redis, MongoDB), especificações e referências amplamente reconhecidas. Cite essas fontes na seção de referências e no frontmatter.
- Nunca invente APIs, métodos, assinaturas ou links. Se você não tem certeza de um detalhe, sinalize explicitamente com uma nota do tipo `> Verificar:` em vez de chutar.
- Todo código precisa estar correto e ser idiomático em .NET 8 e C# 12. Nada de pseudocódigo apresentado como código real. Se for ilustrativo, rotule como tal.
- Prefira informação atual. Quando um comportamento depende de versão, deixe a versão explícita no texto.
- Separe fato de opinião. Tradeoffs e recomendações devem ser apresentados como tal, com o raciocínio por trás.
- Faça uma passada de verificação ao final de cada arquivo: releia as afirmações técnicas e os exemplos, com atenção redobrada à seção de pegadinhas, que é onde o erro é mais perigoso.
- Na dúvida entre incluir algo incerto ou deixar de fora, deixe de fora e registre como pendência.

## 7. Convenções de estilo

- Escreva em português do Brasil. Mantenha termos técnicos consagrados em inglês (deadlock, change tracking, deferred execution e similares).
- Não use travessão nem hífen duplo em nenhum texto. Use vírgula, parênteses ou dois pontos. Esta regra vale para todo o conteúdo gerado.
- Não escreva comentários dentro dos blocos de código. O código deve ser limpo e autoexplicativo pelos nomes.
- Tom didático e direto. Conciso, mas completo. Sem enrolação e sem repetição.
- Use links relativos entre os arquivos do acervo quando referenciar outro tópico (ex.: `[ValueTask](01-csharp-dotnet/task-vs-valuetask.md)`).
- Um tópico por arquivo. Não misture assuntos.

## 8. Conteúdo de referência por área

Use a lista abaixo como ponto de partida do que cada tópico deve cobrir. Aprofunde os tópicos marcados como avançados.

**01 csharp-dotnet**
- async-await: modelo assíncrono, suspensão e retomada, SynchronizationContext, ConfigureAwait, ausência de contexto no ASP.NET Core, deadlock por sync-over-async, async void, fire and forget. Avançado.
- task-vs-valuetask: o que é uma Task, custo de alocação, quando a ValueTask vale a pena, as restrições da ValueTask (não aguardar duas vezes, não acessar .Result), exemplos reais de um método que muitas vezes completa de forma síncrona. Avançado, com exemplos de código de features reais.
- iqueryable-vs-ienumerable: árvore de expressão versus LINQ to Objects, deferred execution, onde o filtro roda, enumeração múltipla. Avançado.
- garbage-collector, record-class-struct, generics-variance, dotnet8-csharp12-new-features, pipeline-middleware.

**02 microservices-patterns**
- saga (orchestration versus choreography, compensation), outbox-pattern, idempotency. Avançados.
- circuit-breaker-retry (com Polly), cqrs, api-gateway-service-discovery, classic-design-patterns.

**03 ef-dapper-postgresql**
- transaction-isolation (dirty, non-repeatable e phantom reads, MVCC no PostgreSQL, padrão Read Committed). Avançado.
- change-tracking, n-plus-one-problem, migrations, jsonb-indexes, dapper-vs-ef, essential-sql.

**04 tests-unitarios**
- test-doubles (dummy, stub, fake, mock, spy, com a distinção entre verificar estado e verificar interação). aaa-pattern, testing-tools.

**05 messaging-distribuidos**
- delivery-semantics (at-most-once, at-least-once, exactly-once e idempotency), rabbitmq-vs-kafka, cap-theorem. Avançados.
- dead-letter-queue.

**06 docker-k8s-cicd-azure**
- image-vs-container, kubernetes-objects, ci-cd (delivery versus deployment), azure-overview.

**07 quality-solid**
- solid (um princípio por seção, cada um com exemplo concreto de violação), code-smells, sonarqube.

**08 differentials**
- redis (cache-aside, TTL, eviction), mongodb (embedding versus referencing), datadog-observability (logs, metrics, traces), gitflow (versus trunk-based).

## 9. Processo de trabalho

1. Crie primeiro o esqueleto de pastas e os arquivos `_index.md` de cada área, mais o `README.md` da raiz com a lista de áreas e a ordem de estudo sugerida (fundamentos antes de avançados).
2. Gere os arquivos área por área, na ordem de 01 a 08. Dentro de cada área, comece pelos tópicos de menor dificuldade.
3. Gere um arquivo completo por vez, do começo ao fim do template, e só então passe para o próximo. Não deixe arquivos pela metade.
4. Ao terminar cada área, atualize o `_index.md` dela com os links e uma linha de descrição por tópico.
5. Não sobrescreva um arquivo já marcado como `status: reviewed` sem necessidade. O acervo é incremental.

## 10. Critério de pronto

Um arquivo está pronto quando: tem o frontmatter completo e válido, segue o template de seções, traz código que compila em .NET 8, cita fontes autoritativas, inclui perguntas de auto-teste com respostas, e passou pela sua releitura de verificação sem afirmações duvidosas em aberto.

Comece criando o esqueleto do repositório e, em seguida, gere a área 01, tópico async-await.
