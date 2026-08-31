---
layout: post
title: "Modelagem de domínio quando parte dele não é seu: os desafios de integrações externas"
date: 2026-08-31
categories: [C#, .NET, Domain-Driven Design, Arquitetura]
permalink: /2026/08/31/modelagem-de-dominio-quando-parte-dele-nao-e-seu/
---

Quando assumimos o desenvolvimento de um sistema que precisa se integrar com um software externo, é comum pensarmos primeiro nos detalhes técnicos da comunicação: autenticação, protocolo, formato de payload, tratamento de erros. O que raramente discutimos com a mesma atenção é uma pergunta mais incômoda.

> **De quem é o domínio dessa informação?**

Em muitos sistemas que desenvolvemos, parte do nosso domínio simplesmente não nasce dentro da nossa aplicação. Ele já existe, com suas próprias regras, seu próprio ciclo de vida e sua própria modelagem, dentro de outro sistema. E isso muda completamente a forma como devemos pensar a arquitetura.

## O cenário: um módulo de vendas integrado ao SAP

Em um projeto que passei, fiz o desenvolvimento de um módulo de vendas dentro de um monolito que precisava se integrar ao SAP. Tínhamos, dentro da nossa aplicação, um domínio bem definido para usuários e controle de acesso por módulos. Até aí, tudo dentro do nosso contexto, com nossas regras, nossa linguagem, nosso desenho.

O problema começou quando entramos nas informações centrais do processo de vendas: pedidos, esboços de documentos e itens de pedido. Essas informações não pertenciam ao nosso domínio, pertenciam ao domínio do SAP, materializadas em tabelas como:

- `ORDR`: cabeçalho de pedidos
- `ODRF`: esboços de documentos
- `RDR1`: linhas de um pedido

A decisão tomada na época foi trazer esses dados para dentro da nossa camada de domínio via **scaffold**, em um `DbContext` separado, apontando diretamente para o banco do SAP.

## O problema: um domínio anêmico e ilegível

O scaffold resolveu o problema de curto prazo: conseguimos ler e persistir dados rapidamente. Mas gerou dois problemas sérios que só ficaram evidentes com o tempo.

### 1. Domínio anêmico

As classes geradas eram, na prática, DTOs disfarçados de entidades. Nenhuma regra de negócio, nenhum comportamento, apenas propriedades públicas mapeando colunas:

```csharp
// Classe gerada via scaffold, sem nenhuma regra de domínio
public class ORDR
{
    public int DocEntry { get; set; }
    public string DocNum { get; set; }
    public string CardCode { get; set; }
    public DateTime? DocDate { get; set; }
    public string DocStatus { get; set; }
    public double? DocTotal { get; set; }
    // ... dezenas de outras propriedades
}
```

Qualquer regra que precisássemos aplicar, como "um pedido só pode ser cancelado se estiver em aberto", por exemplo, acabava espalhada em services, sem nenhum lugar natural para viver. Isso é o oposto do que buscamos ao modelar um domínio rico.

### 2. Nomenclatura ilegível

Como o scaffold gera as propriedades a partir dos nomes originais das colunas, o código passou a carregar nomenclaturas que só faziam sentido para quem conhecia a documentação do SAP:

```csharp
// O que é "U_TipoFrete"? O que significa "TransportationCode"?
// A resposta só estava na documentação do SAP, não no código.
var pedido = context.ORDR
    .Where(o => o.DocStatus == "O" && o.U_TipoFrete == "1")
    .FirstOrDefault();
```

O código deixou de comunicar intenção. Para entender uma consulta simples, era preciso abrir a documentação do SAP ao lado do editor. Isso é uma dívida técnica silenciosa: não trava a aplicação, mas trava o desenvolvedor.

## O que eu faria diferente hoje

Vale separar dois pontos aqui, porque nem tudo estava errado na decisão original.

**O que eu manteria:** no cenário em questão, não havia como trazer esse contexto de leitura para dentro da nossa aplicação de outra forma. O SAP disponibiliza uma integração REST oficial (a **Service Layer**), mas ela não cobria as consultas que precisávamos fazer: eram filtros e cruzamentos mais específicos, fora do que a documentação da Service Layer oferecia de forma nativa. Isso nos deixou sem alternativa além de consultar o banco diretamente, já que o SAP era a fonte da verdade dessas informações. Essa parte, hoje, eu manteria como estava.

**O que eu mudaria:** a forma como esses dados chegavam até o domínio. Em vez de expor as entidades geradas por scaffold diretamente na camada de domínio, eu criaria uma **entidade própria do core da aplicação**, com nomes e semântica que fazem sentido para o nosso negócio, e usaria o mapeamento do próprio ORM para traduzir entre a entidade e a tabela externa.

No caso do EF Core, isso é feito de forma simples e elegante via **Fluent API**.

### Passo 1: Entidade pertencente ao nosso domínio

```csharp
// Entidade do nosso core, com nomes e comportamento que fazem sentido pra nós
public class SalesOrder
{
    public int Id { get; private set; }
    public string Number { get; private set; }
    public string CustomerCode { get; private set; }
    public DateTime? IssueDate { get; private set; }
    public SalesOrderStatus Status { get; private set; }
    public decimal TotalAmount { get; private set; }

    // Regra de domínio vivendo onde deveria viver
    public bool CanBeCancelled()
    {
        return Status == SalesOrderStatus.Open;
    }
}

public enum SalesOrderStatus
{
    Open,
    Closed,
    Cancelled
}
```

### Passo 2: Mapeamento via Fluent API

```csharp
public class SalesOrderMap : IEntityTypeConfiguration<SalesOrder>
{
    public void Configure(EntityTypeBuilder<SalesOrder> builder)
    {
        builder.ToTable("ORDR"); // tabela real do SAP

        builder.HasKey(p => p.Id);

        builder.Property(p => p.Id)
            .HasColumnName("DocEntry");

        builder.Property(p => p.Number)
            .HasColumnName("DocNum");

        builder.Property(p => p.CustomerCode)
            .HasColumnName("CardCode");

        builder.Property(p => p.IssueDate)
            .HasColumnName("DocDate");

        builder.Property(p => p.TotalAmount)
            .HasColumnName("DocTotal");

        // Conversão de valores "místicos" do SAP para algo que faz sentido no domínio
        builder.Property(p => p.Status)
            .HasColumnName("DocStatus")
            .HasConversion(
                status => status == SalesOrderStatus.Open ? "O" : "C",
                sapValue => sapValue == "O" ? SalesOrderStatus.Open : SalesOrderStatus.Closed
            );
    }
}
```

### Passo 3: Consulta legível

```csharp
var openOrders = context.SalesOrders
    .Where(p => p.Status == SalesOrderStatus.Open)
    .ToList();
```

O resultado é uma diferença enorme de legibilidade. Ninguém precisa saber que `DocStatus = "O"` significa "aberto": essa tradução acontece uma única vez, no mapeamento, e nunca mais precisa ser lembrada pelo resto da aplicação.

### Passo 4: Consultas complexas em um repositório de leitura, sobre uma view

Outra mudança que eu faria hoje: as consultas mais complexas (aquelas com filtros e cruzamentos específicos que nem a Service Layer nem um mapeamento simples de entidade resolvem bem) eu não deixaria soltas em LINQ espalhado pela aplicação. Eu colocaria essa lógica em uma **view construída previamente no próprio banco do SAP**, e consumiria essa view através de um repositório dedicado, de somente leitura.

Isso traz dois ganhos diretos: a performance melhora, porque o processamento pesado roda a nível de banco de dados, já otimizado para leitura; e a lógica da consulta fica centralizada em um único lugar, a view, em vez de reimplementada (e reinterpretada) toda vez que alguém precisa de uma variação da mesma informação.

```csharp
// Modelo de leitura, mapeado para uma view (sem tracking, sem regras de escrita)
public class OpenSalesOrderSummary
{
    public string OrderNumber { get; set; }
    public string CustomerCode { get; set; }
    public decimal TotalAmount { get; set; }
    public int OpenItemsCount { get; set; }
}

public interface ISalesOrderReadRepository
{
    Task<IReadOnlyList<OpenSalesOrderSummary>> GetOpenOrdersSummaryAsync();
}

public class SalesOrderReadRepository : ISalesOrderReadRepository
{
    private readonly SapReadDbContext _context;

    public SalesOrderReadRepository(SapReadDbContext context)
    {
        _context = context;
    }

    public async Task<IReadOnlyList<OpenSalesOrderSummary>> GetOpenOrdersSummaryAsync()
    {
        // vw_OpenSalesOrdersSummary é uma view criada no banco do SAP,
        // já com os filtros e joins mais complexos resolvidos a nível de banco
        return await _context.Set<OpenSalesOrderSummary>()
            .FromSqlRaw("SELECT * FROM vw_OpenSalesOrdersSummary")
            .AsNoTracking()
            .ToListAsync();
    }
}
```

Com isso, a entidade de domínio (`SalesOrder`) continua responsável pelo que é, de fato, domínio (regras e comportamento), enquanto o repositório de leitura assume as consultas mais elaboradas, sem misturar as duas responsabilidades.

### Um ponto que ainda não tenho certeza

Tem uma outra mudança que fico em dúvida se faria hoje, e prefiro ser honesto sobre isso em vez de fingir uma certeza que não tenho: às vezes penso que um repositório deveria lidar *apenas* com lógica de banco de dados; em outros momentos, penso que ele pode (e talvez deva) lidar com tudo que diz respeito à leitura e persistência de um agregado, independente do meio.

A questão surge porque, para persistência, escrever diretamente via SQL contra as tabelas do SAP não é uma opção segura: perderíamos as garantias, validações e regras internas que o próprio SAP aplica ao processar um pedido através da Service Layer. Ou seja, se a leitura pode (e às vezes precisa) ir direto ao banco, a escrita normalmente não pode.

Uma possibilidade seria ter um repositório dedicado exclusivamente à persistência, totalmente independente do repositório de leitura mostrado acima, sem métodos de consulta, sem `Get`, nada além de operações de escrita. Por dentro, esse repositório não geraria `INSERT`/`UPDATE` algum: ele chamaria a Service Layer via HTTP.

```csharp
public interface ISalesOrderCommandRepository
{
    Task SaveAsync(SalesOrder order); // por dentro, chama a Service Layer via HTTP
}

public class SalesOrderCommandRepository : ISalesOrderCommandRepository
{
    private readonly SapServiceLayerClient _serviceLayer;

    public SalesOrderCommandRepository(SapServiceLayerClient serviceLayer)
    {
        _serviceLayer = serviceLayer;
    }

    public async Task SaveAsync(SalesOrder order)
    {
        // Delega a persistência para a Service Layer,
        // preservando as regras e validações do próprio SAP
        await _serviceLayer.PostAsync("Orders", order.ToSapPayload());
    }
}
```

Acredito que tenha essa dúvida justamente porque minha perspectiva sobre o que um repository deve ser muda continuamente: ainda não decidi se separar a persistência assim, em um componente próprio, um repositório, que fala com a Service Layer via HTTP em vez de tocar o banco diretamente, é a forma mais correta de pensar isso, ou se existe uma abordagem melhor que eu ainda não enxerguei. É um ponto que ainda estou pensando, e que provavelmente vira assunto para outro post.

## A lição por trás do exemplo

O scaffold não é o vilão. Ele é uma ferramenta ótima para prototipagem rápida ou para cenários onde realmente não importa a expressividade do domínio. O problema é usá-lo como modelagem definitiva de domínio, sem uma camada de tradução entre "o que o banco externo chama as coisas" e "o que o nosso negócio chama as coisas".

Essa mesma armadilha, aliás, não é exclusiva de scaffold: qualquer consulta SQL pura escrita diretamente contra nomes originais de um banco de terceiros carrega o mesmo risco, uma dívida técnica que só o desenvolvedor enxerga, guardada na cabeça de quem escreveu o código ou na documentação de outro sistema.

No fim, o aprendizado é simples de enunciar e difícil de lembrar no calor do prazo: **o domínio é seu, mesmo quando o dado não é**. Vale o esforço de traduzir cedo.

Nos vemos no próximo post.