# Fgc.MessageContracts

Class library .NET 8 com os contratos de eventos (records) compartilhados entre os microsserviços da plataforma FCG, publicada como **pacote NuGet local** (não vai para o nuget.org). Cada serviço consumidor referencia o pacote via um `nuget.config` próprio apontando para uma pasta `./LocalPackages`.

---

## Eventos definidos

Todos em `Fgc.MessageContracts/Events/UserCreatedEvent.cs` (apesar do nome do arquivo, o arquivo contém os três eventos):

```csharp
namespace Fgc.MessageContracts.Events;

public record UserCreatedEvent(Guid Id, string Name, string Email, DateTime CreatedAt);

public record OrderPlacedEvent(Guid OrderId, Guid UserId, Guid GameId, decimal Price);

public record PaymentProcessedEvent(
    Guid OrderedId,   // atenção: campo é "OrderedId", não "OrderId"
    Guid UserId,
    Guid GameId,
    decimal Price,
    string Status,
    DateTime ProcessedAt);
```

`fgc-notifications-lambda` **não** usa este pacote — mantém cópias locais equivalentes desses eventos em `Application/Events/` (decisão própria daquele repositório, não uma limitação deste pacote).

---

## Versão do projeto vs. pacote publicado

O `.csproj` está em `<Version>1.0.3</Version>`, mas o único `.nupkg` commitado em `LocalPackages/` deste repositório é `Fgc.MessageContracts.1.0.0.nupkg` — ou seja, mais antigo que a versão atual do código-fonte (não inclui `OrderPlacedEvent`/`PaymentProcessedEvent`, adicionados depois).

Os serviços consumidores (`fgc-users-api`, `fgc-catalog-api`, `fgc-payments-api`) referenciam as versões `1.0.1` e/ou `1.0.3` a partir de suas **próprias** pastas `LocalPackages/` (cada um mantém seu próprio `.nupkg`), e não a partir da pasta `LocalPackages/` deste repositório. Antes de assumir que um evento está disponível em um serviço consumidor, confira a versão exata em `<PackageReference Include="Fgc.MessageContracts" Version="..."/>` do `.csproj` daquele serviço.

---

## Como gerar e distribuir uma nova versão

1. Atualize `<Version>` em `Fgc.MessageContracts/Fgc.MessageContracts.csproj` (SemVer).
2. Gere o pacote:
   ```bash
   dotnet pack Fgc.MessageContracts/Fgc.MessageContracts.csproj -c Release -o ./LocalPackages
   ```
3. Copie o `.nupkg` gerado (`Fgc.MessageContracts.<versão>.nupkg`) para a pasta `LocalPackages/` de cada serviço consumidor que deva atualizar a referência.
4. Nos serviços consumidores, atualize `<PackageReference Include="Fgc.MessageContracts" Version="<versão>"/>` e rode `dotnet restore`.

Não há workflow de CI/build automatizado neste repositório — o processo acima é manual.

---

## Estrutura

```text
fgc-message-contracts/
└── Fgc.MessageContracts/
    ├── Fgc.MessageContracts.csproj
    └── Events/
        └── UserCreatedEvent.cs   # UserCreatedEvent, OrderPlacedEvent, PaymentProcessedEvent
LocalPackages/
└── Fgc.MessageContracts.1.0.0.nupkg
```
