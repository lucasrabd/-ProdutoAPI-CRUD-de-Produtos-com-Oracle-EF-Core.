# 📦 ProdutoAPI

API REST em ASP.NET Core 8 para gerenciamento de produtos (CRUD), com Entity Framework Core sobre banco Oracle. Projeto acadêmico — Sprint 4.

---

## 📦 Stack

![.NET](https://img.shields.io/badge/.NET-8.0-512BD4?style=flat&logo=dotnet&logoColor=white)
![EF Core](https://img.shields.io/badge/Entity%20Framework%20Core-Oracle-F80000?style=flat)
![Swagger](https://img.shields.io/badge/Swagger-OpenAPI-85EA2D?style=flat&logo=swagger&logoColor=black)
![xUnit](https://img.shields.io/badge/xUnit-Moq-5C2D91?style=flat)

---

## 🏗️ Arquitetura

```
Controllers/   → ProdutosController: endpoints REST de produtos
Repositories/  → ProdutoRepository: acesso a dados via EF Core (AppDbContext)
Data/          → AppDbContext: DbContext do EF Core configurado para Oracle
Models/        → Produto: entidade (Id, Nome, Preco, Categoria)
Services/      → FirebaseAuthService (registro/login via Firebase Auth) e ConfiguracaoManager (singleton, ver nota abaixo)
```

---

## 📁 Estrutura do Projeto

```
ProdutoAPI-Sprint-4-master/
├── Controllers/
│   └── ProdutosController.cs      # Endpoints CRUD de produtos
├── Repositories/
│   ├── IProdutoRepository.cs        # Contrato
│   └── ProdutoRepository.cs           # Implementação com EF Core
├── Data/
│   └── AppDbContext.cs                  # DbContext Oracle
├── Models/
│   └── Produto.cs                         # Entidade Produto
├── Services/
│   ├── FirebaseAuthService.cs               # RegisterUser / LoginUser via Firebase Identity Toolkit
│   └── ConfiguracaoManager.cs                 # Singleton de configuração (stub, não usado no restante do código)
├── Properties/
│   └── launchSettings.json                      # Perfis de execução (portas HTTP/HTTPS)
├── appsettings.json                                # Configuração + connection string do Oracle
├── appsettings.Development.json                      # Overrides de desenvolvimento
├── UnitTest1.cs                                         # Testes do FirebaseAuthService (xUnit + Moq)
├── ProdutoAPI.http                                        # Requisições de exemplo (REST Client)
├── ProdutoAPI.csproj                                        # Dependências do projeto
└── ProdutoAPI.sln                                             # Solution do Visual Studio
```

---

## 🔌 Endpoints

Todos em `api/Produtos`:

| Método | Rota | Descrição |
|---|---|---|
| GET | `/api/Produtos` | Lista todos os produtos |
| GET | `/api/Produtos/{id}` | Retorna um produto específico (404 se não existir) |
| POST | `/api/Produtos` | Cria um novo produto |
| PUT | `/api/Produtos/{id}` | Atualiza um produto existente |
| DELETE | `/api/Produtos/{id}` | Remove um produto |

### Modelo `Produto`

```json
{
  "id": 0,
  "nome": "string",
  "preco": 0.0,
  "categoria": "string"
}
```

> **Autenticação Firebase ainda não está exposta via API**: `FirebaseAuthService` implementa `RegisterUser`/`LoginUser` chamando o Identity Toolkit do Firebase, e está registrado no DI (`Program.cs`), mas não existe nenhum `AuthController` que exponha essas rotas — hoje só é chamável internamente pelo código C#. Além disso, `_firebaseApiKey` está com o valor placeholder `"YOUR_FIREBASE_API_KEY"`; é preciso configurar uma chave real do Firebase antes de usar.

---

## 🚀 Como rodar

### Pré-requisitos
- .NET SDK 8.0+
- Acesso a uma instância Oracle com a tabela de produtos (o EF Core cria o schema a partir do `AppDbContext`, mas migrations não estão incluídas no projeto — rode `dotnet ef migrations add Initial` e `dotnet ef database update` se for a primeira vez)
- Visual Studio ou VS Code

### Passos

1. Clone o repositório e acesse a pasta do projeto.
2. Configure a connection string do Oracle em `appsettings.json`:

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Data Source=(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=SEU_HOST)(PORT=1521)))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=ORCL)));User Id=SEU_USUARIO;Password=SUA_SENHA;"
  }
}
```

> Use [User Secrets](https://learn.microsoft.com/aspnet/core/security/app-secrets) (`dotnet user-secrets`) em vez de gravar usuário/senha reais nesse arquivo versionado.

3. Restaure, compile e execute:

```bash
dotnet restore
dotnet build
dotnet run
```

4. Acesse o Swagger:

```
http://localhost:5264/swagger
```

(porta do perfil `http` em `launchSettings.json`; HTTPS roda em `7254`)

---

## 🧪 Testes

```bash
dotnet test
```

`UnitTest1.cs` testa `FirebaseAuthService.RegisterUser`/`LoginUser` usando `Mock<HttpClient>`. Vale observar que mockar `HttpClient` diretamente com Moq não intercepta as chamadas reais — os métodos usados (`PostAsJsonAsync`) são extension methods que acabam chamando `SendAsync` na instância real por baixo dos panos, então esses testes tendem a disparar uma requisição HTTP de verdade em vez de usar um mock efetivo. O padrão mais confiável seria mockar `HttpMessageHandler` e injetar via `new HttpClient(handlerMock.Object)`.

---

## 📦 Dependências (`ProdutoAPI.csproj`)

| Pacote | Versão | Uso |
|---|---|---|
| `Oracle.EntityFrameworkCore` | 8.23.50 | Provider do EF Core para Oracle |
| `Oracle.ManagedDataAccess.Core` | 23.5.1 | Driver de conexão com Oracle |
| `Newtonsoft.Json` | 13.0.3 | Parse de JSON no `FirebaseAuthService` |
| `Swashbuckle.AspNetCore` | 6.4.0 | Geração da UI do Swagger |
| `xunit` / `Moq` | 2.9.2 / 4.20.72 | Testes unitários |

---

## 📄 Licença

Não especificada.
