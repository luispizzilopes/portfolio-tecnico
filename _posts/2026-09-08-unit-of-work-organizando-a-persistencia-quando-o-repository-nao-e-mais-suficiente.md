---
layout: post
title: "Unit of Work: organizando a persistência quando o Repository não é mais suficiente"
date: 2026-09-08
categories: [C#, .NET, Design Patterns]
permalink: /2026/09/08/unit-of-work-organizando-a-persistencia-quando-o-repository-nao-e-mais-suficiente/
---

Quando falamos sobre persistência e leitura de dados no desenvolvimento de aplicações, é quase automático associar o tema ao design pattern **Repository**, seja aplicando o conceito dentro de um contexto de Domain Driven Design, seja apenas como uma camada de abstração sobre o banco de dados: um intermediário entre a aplicação e uma coleção de objetos, ocultando a complexidade da persistência.

Em sistemas que crescem em tamanho e complexidade, é comum surgirem diversos repositórios vinculados diretamente a entidades e agregados do domínio, além de repositórios somente leitura, criados para atender regras de negócio específicas definidas no escopo do projeto. Tudo isso precisa ser traduzido em código, e é aí que mora o problema: a quantidade de arquivos cresce, o controle se perde, e as dependências acabam espalhadas em diversas injeções de dependência e configurações de inversão de controle.

## O cenário: um domínio pequeno até deixar de ser

No começo de um projeto, essa estrutura parece perfeitamente sustentável. Poucos repositórios, poucas entidades, tudo visível em uma única tela do editor. É justamente por isso que o Unit of Work costuma ser subestimado logo de início: parece apenas mais uma camada de abstração, um esforço de implementação que não traz retorno imediato.

O problema é que esse raciocínio só se sustenta enquanto o projeto é pequeno. Conforme o domínio da aplicação cresce, cresce junto o número de repositórios, de agregados e de regras de negócio que dependem de mais de uma entidade sendo persistida ao mesmo tempo. É nesse momento que perguntas incômodas começam a aparecer.

Como garantir que a atualização de duas ou três entidades diferentes aconteça de forma atômica, sem deixar o banco em um estado inconsistente se uma das operações falhar no meio do caminho? Como evitar que cada serviço da aplicação precise injetar cinco, dez, quinze repositórios diferentes só para completar uma única operação de negócio? Como testar essa orquestração sem precisar montar um cenário de mock gigantesco, com um repositório fake para cada dependência?

## O problema: transação sem dono

Sem uma camada que coordene esses repositórios, a tendência natural é que cada desenvolvedor resolva esse problema à sua maneira: um `SaveChanges` aqui, um `try/catch` com rollback manual ali, uma transação aberta diretamente no serviço em outro lugar. O resultado é um código inconsistente, difícil de manter e, principalmente, difícil de confiar.

O Unit of Work atua como uma camada de coordenação entre os repositórios, sendo responsável por controlar o início, a confirmação (commit) e o desfazimento (rollback) de uma transação que pode envolver múltiplas entidades e múltiplos repositórios ao mesmo tempo. Em vez de cada repositório persistir suas próprias alterações de forma isolada, o Unit of Work centraliza esse controle, garantindo que um conjunto de operações seja tratado como uma única unidade atômica, daí o nome.

Isso significa que, se uma regra de negócio precisa atualizar um pedido, debitar um estoque e registrar um evento de auditoria, todas essas operações passam a fazer parte da mesma transação, coordenada por um único ponto do código. Se qualquer uma delas falhar, nenhuma é persistida.

## O que eu ganho além da transação

Embora a atomicidade seja o benefício mais evidente, o Unit of Work traz uma série de outras vantagens que se tornam cada vez mais valiosas conforme o projeto escala.

**Consistência transacional real.** Operações que envolvem múltiplos agregados deixam de depender de código de controle de transação espalhado pela aplicação. Toda a lógica de commit e rollback fica concentrada em um único lugar, reduzindo drasticamente o risco de inconsistência de dados.

**Redução da complexidade de injeção de dependência.** Em vez de um serviço depender diretamente de vários repositórios, ele passa a depender de uma única Unit of Work, que expõe os repositórios necessários. Isso simplifica a configuração de inversão de controle e deixa as assinaturas de construtores muito mais enxutas.

**Facilidade para testes.** Como toda a coordenação de persistência passa por um único ponto, fica muito mais simples criar um mock ou uma implementação em memória da Unit of Work para testes unitários, sem precisar simular o comportamento de dezenas de repositórios isolados.

**Desacoplamento entre regra de negócio e infraestrutura de persistência.** A camada de aplicação ou de domínio não precisa saber como e quando o commit acontece. Ela apenas define o que precisa ser feito, e a Unit of Work garante que isso será persistido de forma correta e consistente.

**Organização e rastreabilidade do código.** Com o tempo, fica muito mais fácil localizar onde as transações acontecem e entender o fluxo de persistência da aplicação, já que ele não está mais pulverizado em múltiplos pontos do código.

**Menor duplicação de código.** Lógicas de abertura de conexão, controle de transação e tratamento de exceções deixam de ser reescritas repositório por repositório, serviço por serviço.

## Um exemplo simples

Uma implementação básica de Unit of Work costuma expor os repositórios necessários e um método de confirmação das alterações.

```csharp
public interface IUnitOfWork : IDisposable
{
    IOrderRepository Orders { get; }
    IInventoryRepository Inventory { get; }
    IAuditLogRepository AuditLogs { get; }

    Task<int> CommitAsync();
}

public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;

    public IOrderRepository Orders { get; }
    public IInventoryRepository Inventory { get; }
    public IAuditLogRepository AuditLogs { get; }

    public UnitOfWork(AppDbContext context)
    {
        _context = context;
        Orders = new OrderRepository(_context);
        Inventory = new InventoryRepository(_context);
        AuditLogs = new AuditLogRepository(_context);
    }

    public async Task<int> CommitAsync()
        => await _context.SaveChangesAsync();

    public void Dispose() => _context.Dispose();
}
```

No serviço de aplicação, o uso fica bastante direto.

```csharp
public async Task CompleteOrderAsync(int orderId)
{
    var order = await _unitOfWork.Orders.GetByIdAsync(orderId);

    order.Complete();
    _unitOfWork.Inventory.Debit(order.Items);
    _unitOfWork.AuditLogs.Register("Order completed", orderId);

    await _unitOfWork.CommitAsync();
}
```

O serviço não sabe nada sobre transação, conexão ou banco de dados. Ele apenas orquestra a regra de negócio e delega a persistência à Unit of Work.

## Quando vale a pena adotar

O Unit of Work não é uma bala de prata nem deve ser aplicado indiscriminadamente. Em aplicações muito pequenas, com poucas entidades e sem operações que envolvam múltiplos agregados, a camada extra pode realmente representar apenas complexidade desnecessária.

Mas à medida que o domínio cresce, que novas regras de negócio passam a envolver mais de uma entidade por operação e que o número de repositórios se multiplica, o padrão deixa de ser opcional na prática. Ele se torna a diferença entre um código organizado e sustentável e um código onde ninguém sabe mais ao certo onde e como as transações são controladas.

## A lição por trás do exemplo

O Unit of Work não substitui o Repository, trabalha em conjunto com ele, adicionando a peça que faltava: coordenação. Ao centralizar o controle transacional, reduzir a complexidade de injeção de dependência e facilitar testes, ele contribui diretamente para a manutenibilidade e a escalabilidade da aplicação como um todo.

O padrão não resolve problemas de design ruins, nem substitui uma modelagem cuidadosa do domínio. Mas, quando o contexto pede coordenação entre múltiplos repositórios, ele é a abstração certa no lugar certo, e ignorá-lo costuma significar reinventá-lo de forma pior, espalhada e inconsistente pelo código.

Nos vemos no próximo post.
