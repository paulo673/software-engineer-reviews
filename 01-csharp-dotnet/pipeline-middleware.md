---
id: pipeline-middleware
title: "Pipeline e Middleware no ASP.NET Core"
area: csharp-dotnet
difficulty: intermediario
prerequisites: [async-await]
related: [async-await]
tags: [aspnetcore, middleware, pipeline, http, request]
sources:
  - "https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/"
  - "https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/write"
status: revisado
last_updated: 2026-06-20
---

## Resumo

No ASP.NET Core, cada requisição HTTP passa por um pipeline de middlewares: componentes encadeados que podem inspecionar, modificar ou curto-circuitar a requisição e a resposta. Cada middleware decide se chama o próximo (`next`) ou encerra ali. A ordem de registro define a ordem de execução, e essa ordem é decisiva para correção (autenticação antes de autorização, tratamento de exceção no topo).

## Explicação detalhada

O pipeline é construído na inicialização, registrando middlewares na ordem desejada. Em tempo de requisição, cada middleware recebe o `HttpContext` e um delegate `next` que representa o resto do pipeline. O padrão é: fazer algo antes, chamar `await next(context)`, fazer algo depois. Isso forma uma estrutura aninhada, como camadas de uma cebola: a requisição desce pelas camadas até um terminal e a resposta sobe de volta na ordem inversa.

Um middleware pode **curto-circuitar**: não chamar `next` e produzir a resposta diretamente. É o que faz, por exemplo, um middleware de arquivos estáticos que encontra o arquivo, ou um de autorização que rejeita com 401/403.

A ordem importa porque cada middleware depende do estado preparado pelos anteriores. Recomendação típica de ordem: tratamento de exceções primeiro (para capturar tudo abaixo), depois HSTS/HTTPS redirection, arquivos estáticos, roteamento, CORS, autenticação, autorização e por fim os endpoints. Autenticação tem que vir antes de autorização, pois autorização decide com base na identidade que a autenticação estabeleceu.

`Use` adiciona um middleware que pode chamar o próximo. `Run` adiciona um terminal que nunca chama o próximo. `Map` ramifica o pipeline por caminho.

## Por baixo dos panos

Cada middleware é, no fundo, uma função `RequestDelegate`, ou seja, `Func<HttpContext, Task>`. O pipeline é montado compondo essas funções: o `app.Use(...)` recebe o `next` e devolve um novo `RequestDelegate` que o envolve. No final, o conjunto vira uma única cadeia de delegates aninhados.

Há duas formas de escrever middleware: o estilo inline com `app.Use(async (context, next) => { ... })`, e o estilo de classe convencional, com um construtor que recebe o `RequestDelegate next` e um método `InvokeAsync(HttpContext context)`. O middleware de classe é instanciado uma vez (singleton de fato) e o `InvokeAsync` é chamado por requisição, então dependências com escopo de requisição devem ser injetadas como parâmetro de `InvokeAsync`, não no construtor.

Como `next` é aguardado com `await`, todo o pipeline é assíncrono de ponta a ponta (ver [async e await](async-await.md)), o que permite I/O sem bloquear threads.

## Exemplos em C#

Middleware inline que mede o tempo de cada requisição:

```csharp
app.Use(async (context, next) =>
{
    var stopwatch = Stopwatch.StartNew();
    await next(context);
    stopwatch.Stop();
    context.Response.Headers["X-Response-Time-ms"] =
        stopwatch.ElapsedMilliseconds.ToString();
});
```

Middleware de classe convencional:

```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        _logger.LogInformation("Início {Method} {Path}", context.Request.Method, context.Request.Path);
        await _next(context);
        _logger.LogInformation("Fim {StatusCode}", context.Response.StatusCode);
    }
}
```

Ordem correta no `Program.cs`:

```csharp
var app = builder.Build();

app.UseExceptionHandler("/error");
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

Curto-circuito que rejeita sem chamar `next`:

```csharp
app.Use(async (context, next) =>
{
    if (!context.Request.Headers.ContainsKey("X-Api-Key"))
    {
        context.Response.StatusCode = StatusCodes.Status401Unauthorized;
        return;
    }
    await next(context);
});
```

## Tradeoffs

- Middlewares dão um ponto central e ordenado para preocupações transversais (logging, autenticação, exceções, compressão), separando-as da lógica de negócio. O custo é que a ordem vira contrato implícito e fácil de quebrar.
- Lógica em middleware roda para todas as requisições que passam por ele; lógica em filtros ou no endpoint é mais localizada. Escolha o nível certo: middleware para o que é global, filtros para o que é específico de MVC.
- Curto-circuitar cedo economiza trabalho, mas pode pular middlewares que algo posterior esperava ter rodado.

## Pegadinhas e erros comuns

- Ordem errada: `UseAuthorization` antes de `UseAuthentication`, ou tratamento de exceção registrado tarde demais para capturar erros de middlewares acima.
- Esquecer de chamar `await next(context)` quando não se queria curto-circuitar: a requisição "morre" silenciosamente.
- Escrever na resposta depois de `await next` quando o middleware seguinte já enviou a resposta (headers já enviados): lança exceção.
- Injetar serviço com escopo de requisição (scoped) no construtor de um middleware de classe: o middleware é singleton, o que captura indevidamente o scoped. Receba-o como parâmetro de `InvokeAsync`.
- Confundir `Use` (encadeia) com `Run` (terminal): um `Run` no meio impede o resto do pipeline.

## Quando usar e quando evitar

Use middleware para preocupações verdadeiramente transversais e globais: tratamento de exceções, logging/correlação, autenticação, CORS, compressão, rate limiting. Coloque cada um na posição correta da ordem. Evite colocar lógica de negócio em middleware, e evite middleware quando um filtro de ação ou um serviço injetado no endpoint resolveria de forma mais localizada e testável.

## Perguntas de auto-teste

1. O que define a ordem de execução dos middlewares?
<details><summary>Resposta</summary>A ordem em que são registrados no pipeline. A requisição desce na ordem de registro e a resposta sobe na ordem inversa.</details>

2. Por que `UseAuthentication` deve vir antes de `UseAuthorization`?
<details><summary>Resposta</summary>Porque a autorização decide o acesso com base na identidade estabelecida pela autenticação. Sem autenticar antes, não há identidade para autorizar.</details>

3. O que significa curto-circuitar o pipeline?
<details><summary>Resposta</summary>Um middleware produzir a resposta sem chamar next, encerrando o pipeline ali e impedindo a execução dos middlewares seguintes.</details>

4. Por que não injetar um serviço scoped no construtor de um middleware de classe?
<details><summary>Resposta</summary>Porque o middleware é instanciado uma vez (singleton). Injetar um scoped no construtor o prenderia ao ciclo do singleton. O correto é recebê-lo como parâmetro de InvokeAsync, resolvido por requisição.</details>

5. Qual a diferença entre `Use`, `Run` e `Map`?
<details><summary>Resposta</summary>Use adiciona middleware que pode chamar o próximo; Run adiciona um terminal que não chama o próximo; Map ramifica o pipeline conforme o caminho da requisição.</details>

6. Por que o tratamento de exceções costuma ser o primeiro middleware?
<details><summary>Resposta</summary>Para envolver todo o restante do pipeline e capturar exceções lançadas por qualquer middleware ou endpoint abaixo dele.</details>

## Diagrama

```mermaid
flowchart LR
    R[Requisição] --> EX[Exception handler]
    EX --> AUTHN[Authentication]
    AUTHN --> AUTHZ[Authorization]
    AUTHZ --> EP[Endpoint]
    EP --> AUTHZ2[volta]
    AUTHZ2 --> AUTHN2[volta]
    AUTHN2 --> EX2[volta]
    EX2 --> RESP[Resposta]
```

## Referências

- [ASP.NET Core Middleware](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/)
- [Write custom ASP.NET Core middleware](https://learn.microsoft.com/en-us/aspnet/core/fundamentals/middleware/write)
