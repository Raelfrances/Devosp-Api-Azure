# Deploy de uma API no Azure

Este guia passo a passo descreve como fazer o deploy de uma API no Azure usando Azure DevOps, Docker e Visual Studio Code.

# Estrutura do Projeto 
```
apiTempo/
|-- Controllers/
|   |-- WeatherForecastController.cs
|-- Models/
|   |-- WeatherForecast.cs
|-- Properties/
|   |-- launchSettings.json
|-- apiTempo.csproj
|-- Dockerfile
|-- Program.cs
|-- Startup.cs
```

## Ferramentas Utilizadas

- **.NET 8.x SDK**: Para desenvolver e executar a aplicação.
- **Docker**: Para containerização da aplicação.
- **Azure DevOps**: Para CI/CD (Integração Contínua/Entrega Contínua).
- **Visual Studio Code**: Como ambiente de desenvolvimento.
- **Swagger**: Para documentação e teste da API.
- **Azure Container Registry (ACR)**: Para armazenar as imagens Docker.
- **Azure Web App**: Para hospedar a aplicação.

##3. Código da API de Previsão do Tempo
```
using Microsoft.AspNetCore.Mvc;
namespace apiTempo.Models
{
    public class WeatherForecast
    {
        public DateTime Date { get; set; }
        public int TemperatureC { get; set; }
        public string Summary { get; set; }
    }
}
namespace apiTempo.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class WeatherForecastController : ControllerBase
    {
        private static readonly string[] Summaries = new[]
        {
            "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
        };

        [HttpGet]
        public IEnumerable<WeatherForecast> Get()
        {
            var rng = new Random();
            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = DateTime.Now.AddDays(index),
                TemperatureC = rng.Next(-20, 55),
                Summary = Summaries[rng.Next(Summaries.Length)]
            })
            .ToArray();
        }
    }
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

app.UseAuthorization();

app.MapControllers();

app.Run();

```

## 1. Configurando o nome do projeto no Azure Web API

Configure o nome do projeto no Azure Web API conforme necessário.

## 2. Configurando Dockerfile Ativo

Nomeie o projeto como `apiTempo` e configure o Dockerfile.

```
# Use uma imagem do SDK do .NET como a imagem base
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
EXPOSE 80

# Use uma imagem do SDK do .NET para construir a aplicação
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY ["apiTempo/apiTempo.csproj", "apiTempo/"]
RUN dotnet restore "apiTempo/apiTempo.csproj"
COPY . .
WORKDIR "/src/apiTempo"
RUN dotnet build "apiTempo.csproj" -c Release -o /app/build

# Publishe a aplicação
FROM build AS publish
RUN dotnet publish "apiTempo.csproj" -c Release -o /app/publish

# Configure a imagem final
FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "apiTempo.dll"]
```

## 3. Configurando Azure DevOps

Crie um projeto no Azure DevOps com o nome `api-Devopss-Azure`.

## 4. Criar Organização no Visual Studio Code Azure DevOps

Crie uma organização no Visual Studio Code Azure DevOps e crie um repositório com o nome `devapi`.

## 5. Clonar Repositório no Prompt de Comando

Clone o repositório criado no CMD:

```bash
git clone https://dev.azure.com/seu-usuario/devapi
```
## 6.  Startar o Container e Buildar o Dockerfile
Start o container e builde o Dockerfile conforme necessário.

## 7. Deixar o Workdir Só com o SRC
Deixe o workdir apenas com a pasta src.

##  8. Criar Recursos na Azure
Crie um Resource Group no Azure com o nome Devops-api-project.

## 9. Criar um ACR
Crie um Azure Container Registry (ACR) com o nome acr-Devops-Project e deixe no nível Basic.

##  10. Criar Web App
Crie um Web App no Azure com o nome Devops-api-app.

## 11. Criar Pipeline no Azure DevOps
No Azure DevOps, crie um pipeline com o seguinte conteúdo:
```
# Starter pipeline
trigger:
- main

pool:
  vmImage: ubuntu-latest

variables:
  solution: './apiTempo/*.sln'
  buildPlataform: 'anyCpu'
  buildConfiguration: 'release'

steps:
- task: UseDotNet@2
  displayName: 'Install .Net sdk'
  inputs:
    packageType: 'sdk'
    version: '8.x'

- script: |
    dotnet restore $(solution)
  displayName: 'Restore solution'

- script: dotnet build $(solution) --configuration $(buildConfiguration)
  displayName: 'Build solution'
  
- script: dotnet test $(solution) --configuration $(buildConfiguration) --nobuild --collect:"XPlat Code Coverage"
  displayName: 'Test solution'

# Adicione aqui os comandos para publicar seus artefatos
# Exemplo:
# - task: PublishPipelineArtifact
#   inputs:
#     artifact: '$(Build.ArtifactStagingDirectory)'
#   displayName: 'Publish artifacts'

- task: Docker@2
  inputs:
    containerRegistry: 'acrApiDevops'
    repository: 'devapi'
    command: 'buildAndPush'
    Dockerfile: './apiTempo/Dockerfile'
```

## 12. Criar Service Registry
Configure o registro do serviço Docker da aplicação com conectividade com o ACR.

## 13. Criar Webhook
Crie um webhook e pegue o link no container.

## 14. Testar API no Swagger

## 15. Ativar Trigger no YAML
Ative o trigger (gatilho) no YAML para qualquer mudança.
