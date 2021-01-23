---
ms.openlocfilehash: a024fb533c552563da6c9179301e16a2e1d09d5f
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942159"
---
<!-- markdownlint-disable MD002 MD041 -->

Neste exercício, você estenderá o aplicativo do exercício anterior para dar suporte à autenticação com o Azure AD. Isso é necessário para obter o token de acesso OAuth necessário para chamar a API do Microsoft Graph. Nesta etapa, você configurará a [biblioteca Microsoft.Identity.Web.](https://www.nuget.org/packages/Microsoft.Identity.Web/)

> [!IMPORTANT]
> Para evitar armazenar a ID do aplicativo e o segredo na fonte, você usará o Gerenciador de Segredos [do .NET](/aspnet/core/security/app-secrets) para armazenar esses valores. O Gerenciador de Segredos é apenas para fins de desenvolvimento, os aplicativos de produção devem usar um gerenciador de segredos confiáveis para armazenar segredos.

1. Abra **./appsettings.jse** substitua seu conteúdo pelo seguinte.

    :::code language="json" source="../demo/GraphTutorial/appsettings.json" highlight="2-6":::

1. Abra sua CLI no diretório onde **GraphTutorial.csproj** está localizado e execute os seguintes comandos, substituindo a ID do aplicativo pelo portal do Azure e pelo segredo do `YOUR_APP_ID` `YOUR_APP_SECRET` aplicativo.

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a>Implementar a login

Comece adicionando os serviços da Plataforma de Identidade da Microsoft ao aplicativo.

1. Crie um novo arquivo **chamado GraphConstants.cs** no diretório **./Graph** e adicione o código a seguir.

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphConstants.cs" id="GraphConstantsSnippet":::

1. Abra o **arquivo ./Startup.cs** e adicione as instruções a `using` seguir na parte superior do arquivo.

    ```csharp
    using Microsoft.AspNetCore.Authentication.OpenIdConnect;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Mvc.Authorization;
    using Microsoft.Identity.Web;
    using Microsoft.Identity.Web.UI;
    using Microsoft.IdentityModel.Protocols.OpenIdConnect;
    using Microsoft.Graph;
    using System.Net;
    using System.Net.Http.Headers;
    ```

1. Substitua a função `ConfigureServices` existente pelo seguinte.

    ```csharp
    public void ConfigureServices(IServiceCollection services)
    {
        services
            // Use OpenId authentication
            .AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
            // Specify this is a web app and needs auth code flow
            .AddMicrosoftIdentityWebApp(Configuration)
            // Add ability to call web API (Graph)
            // and get access tokens
            .EnableTokenAcquisitionToCallDownstreamApi(options => {
                Configuration.Bind("AzureAd", options);
            }, GraphConstants.Scopes)
            // Use in-memory token cache
            // See https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization
            .AddInMemoryTokenCaches();

        // Require authentication
        services.AddControllersWithViews(options =>
        {
            var policy = new AuthorizationPolicyBuilder()
                .RequireAuthenticatedUser()
                .Build();
            options.Filters.Add(new AuthorizeFilter(policy));
        })
        // Add the Microsoft Identity UI pages for signin/out
        .AddMicrosoftIdentityUI();
    }
    ```

1. Na `Configure` função, adicione a seguinte linha acima da `app.UseAuthorization();` linha.

    ```csharp
    app.UseAuthentication();
    ```

1. Abra **./Controllers/HomeController.cs** e substitua seu conteúdo pelo seguinte.

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.AspNetCore.Authorization;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Web;
    using System.Diagnostics;
    using System.Threading.Tasks;

    namespace GraphTutorial.Controllers
    {
        public class HomeController : Controller
        {
            ITokenAcquisition _tokenAcquisition;
            private readonly ILogger<HomeController> _logger;

            // Get the ITokenAcquisition interface via
            // dependency injection
            public HomeController(
                ITokenAcquisition tokenAcquisition,
                ILogger<HomeController> logger)
            {
                _tokenAcquisition = tokenAcquisition;
                _logger = logger;
            }

            public async Task<IActionResult> Index()
            {
                // TEMPORARY
                // Get the token and display it
                try
                {
                    string token = await _tokenAcquisition
                        .GetAccessTokenForUserAsync(GraphConstants.Scopes);
                    return View().WithInfo("Token acquired", token);
                }
                catch (MicrosoftIdentityWebChallengeUserException)
                {
                    return Challenge();
                }
            }

            public IActionResult Privacy()
            {
                return View();
            }

            [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
            public IActionResult Error()
            {
                return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier });
            }

            [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
            [AllowAnonymous]
            public IActionResult ErrorWithMessage(string message, string debug)
            {
                return View("Index").WithError(message, debug);
            }
        }
    }
    ```

1. Salve suas alterações e inicie o projeto. Faça logon com sua conta da Microsoft.

1. Examine a solicitação de consentimento. A lista de permissões corresponde à lista de escopos de permissões configurados em **./Graph/GraphConstants.cs**.

    - **Mantenha acesso aos dados aos** que você deu a eles acesso: ( ) Essa permissão é solicitada pela MSAL para recuperar `offline_access` tokens de atualização.
    - **Entre e leia seu perfil:** ( ) Essa permissão permite que o aplicativo receba o perfil do usuário conectado e a foto `User.Read` do perfil.
    - **Leia as configurações da** caixa de correio: ( ) Essa permissão permite que o aplicativo leia as configurações da caixa de correio do usuário, incluindo fuso horário e `MailboxSettings.Read` formato de hora.
    - Tenha acesso total aos seus **calendários:** ( ) Essa permissão permite que o aplicativo leia eventos no calendário do usuário, adicione novos eventos e modifique `Calendars.ReadWrite` os existentes.

    ![Uma captura de tela do prompt de consentimento da Plataforma de Identidade da Microsoft](./images/add-aad-auth-03.png)

    Para obter mais informações sobre consentimento, consulte Noções básicas sobre experiências de consentimento [do aplicativo Azure AD.](/azure/active-directory/develop/application-consent-experience)

1. Consentir com as permissões solicitadas. O navegador redireciona para o aplicativo, mostrando o token.

### <a name="get-user-details"></a>Obter detalhes do usuário

Depois que o usuário está conectado, você pode obter as informações dele no Microsoft Graph.

1. Abra **./Graph/GraphClaimsPrincipalExtensions.cs** e substitua todo o conteúdo pelo seguinte.

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphClaimsPrincipalExtensions.cs" id="GraphClaimsExtensionsSnippet":::

1. Abra **./Startup.cs** e substitua a linha `.AddMicrosoftIdentityWebApp(Configuration)` existente pelo código a seguir.

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddSignInSnippet":::

    Considere o que esse código faz.

    - Ele adiciona um manipulador de eventos para o `OnTokenValidated` evento.
        - Ele usa a `ITokenAcquisition` interface para obter um token de acesso.
        - Ele chama o Microsoft Graph para obter o perfil e a foto do usuário.
        - Ele adiciona as informações do Graph à identidade do usuário.

1. Adicione a seguinte chamada de função após `EnableTokenAcquisitionToCallDownstreamApi` a chamada e antes da `AddInMemoryTokenCaches` chamada.

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddGraphClientSnippet":::

    Isso disponibiliza um **GraphServiceClient** autenticado para controladores por meio de injeção de dependência.

1. Abra **./Controllers/HomeController.cs** e substitua `Index` a função pelo seguinte.

    ```csharp
    public IActionResult Index()
    {
        return View();
    }
    ```

1. Remova todas as referências `ITokenAcquisition` à classe **HomeController.**

1. Salve suas alterações, inicie o aplicativo e vá pelo processo de login. Você deve voltar à home page, mas a interface do usuário deve mudar para indicar que você está entrar.

    ![Uma captura de tela da home page após entrar](./images/add-aad-auth-01.png)

1. Clique no avatar do usuário no canto superior direito para acessar o link **Sair.** Clicar em **Sair** redefine a sessão e o retorna para a home page.

    ![Uma captura de tela do menu suspenso com o link Sair](./images/add-aad-auth-02.png)

> [!TIP]
> Se você não vir seu nome de usuário na home page e o nome e email do avatar de uso não estiver no nome e no email depois de fazer essas alterações, saia e entre novamente.

## <a name="storing-and-refreshing-tokens"></a>Armazenar e atualizar tokens

Neste ponto, seu aplicativo tem um token de acesso, que é enviado no `Authorization` header de chamadas de API. Esse é o token que permite que o aplicativo acesse o Microsoft Graph em nome do usuário.

No entanto, esse token tem curta duração. O token expira uma hora após sua emissão. É aqui que o token de atualização se torna útil. O token de atualização permite que o aplicativo solicite um novo token de acesso sem exigir que o usuário entre novamente.

Como o aplicativo está usando a biblioteca Microsoft.Identity.Web, você não precisa implementar nenhum armazenamento de token ou lógica de atualização.

O aplicativo usa o cache de token na memória, o que é suficiente para aplicativos que não precisam persistir tokens quando o aplicativo é reiniciado. Em vez disso, os aplicativos de produção podem usar [as opções de cache distribuído](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) na biblioteca Microsoft.Identity.Web.

O `GetAccessTokenForUserAsync` método lida com a expiração do token e a atualização para você. Primeiro, ele verifica o token armazenado em cache e, se não tiver expirado, ele o retornará. Se ele tiver expirado, ele usará o token de atualização em cache para obter um novo.

O **GraphServiceClient que** os controladores receberem por injeção de dependência será pré-configurado com um provedor de autenticação que usa `GetAccessTokenForUserAsync` para você.
