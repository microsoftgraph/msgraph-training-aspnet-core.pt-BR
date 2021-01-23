---
ms.openlocfilehash: a024fb533c552563da6c9179301e16a2e1d09d5f
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942159"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="1144b-101">Neste exercício, você estenderá o aplicativo do exercício anterior para dar suporte à autenticação com o Azure AD.</span><span class="sxs-lookup"><span data-stu-id="1144b-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="1144b-102">Isso é necessário para obter o token de acesso OAuth necessário para chamar a API do Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="1144b-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph API.</span></span> <span data-ttu-id="1144b-103">Nesta etapa, você configurará a [biblioteca Microsoft.Identity.Web.](https://www.nuget.org/packages/Microsoft.Identity.Web/)</span><span class="sxs-lookup"><span data-stu-id="1144b-103">In this step you will configure the [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) library.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="1144b-104">Para evitar armazenar a ID do aplicativo e o segredo na fonte, você usará o Gerenciador de Segredos [do .NET](/aspnet/core/security/app-secrets) para armazenar esses valores.</span><span class="sxs-lookup"><span data-stu-id="1144b-104">To avoid storing the application ID and secret in source, you will use the [.NET Secret Manager](/aspnet/core/security/app-secrets) to store these values.</span></span> <span data-ttu-id="1144b-105">O Gerenciador de Segredos é apenas para fins de desenvolvimento, os aplicativos de produção devem usar um gerenciador de segredos confiáveis para armazenar segredos.</span><span class="sxs-lookup"><span data-stu-id="1144b-105">The Secret Manager is for development purposes only, production apps should use a trusted secret manager for storing secrets.</span></span>

1. <span data-ttu-id="1144b-106">Abra **./appsettings.jse** substitua seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="1144b-106">Open **./appsettings.json** and replace its contents with the following.</span></span>

    :::code language="json" source="../demo/GraphTutorial/appsettings.json" highlight="2-6":::

1. <span data-ttu-id="1144b-107">Abra sua CLI no diretório onde **GraphTutorial.csproj** está localizado e execute os seguintes comandos, substituindo a ID do aplicativo pelo portal do Azure e pelo segredo do `YOUR_APP_ID` `YOUR_APP_SECRET` aplicativo.</span><span class="sxs-lookup"><span data-stu-id="1144b-107">Open your CLI in the directory where **GraphTutorial.csproj** is located, and run the following commands, substituting `YOUR_APP_ID` with your application ID from the Azure portal, and `YOUR_APP_SECRET` with your application secret.</span></span>

    ```Shell
    dotnet user-secrets init
    dotnet user-secrets set "AzureAd:ClientId" "YOUR_APP_ID"
    dotnet user-secrets set "AzureAd:ClientSecret" "YOUR_APP_SECRET"
    ```

## <a name="implement-sign-in"></a><span data-ttu-id="1144b-108">Implementar a login</span><span class="sxs-lookup"><span data-stu-id="1144b-108">Implement sign-in</span></span>

<span data-ttu-id="1144b-109">Comece adicionando os serviços da Plataforma de Identidade da Microsoft ao aplicativo.</span><span class="sxs-lookup"><span data-stu-id="1144b-109">Start by adding the Microsoft Identity platform services to the application.</span></span>

1. <span data-ttu-id="1144b-110">Crie um novo arquivo **chamado GraphConstants.cs** no diretório **./Graph** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="1144b-110">Create a new file named **GraphConstants.cs** in the **./Graph** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphConstants.cs" id="GraphConstantsSnippet":::

1. <span data-ttu-id="1144b-111">Abra o **arquivo ./Startup.cs** e adicione as instruções a `using` seguir na parte superior do arquivo.</span><span class="sxs-lookup"><span data-stu-id="1144b-111">Open the **./Startup.cs** file and add the following `using` statements to the top of the file.</span></span>

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

1. <span data-ttu-id="1144b-112">Substitua a função `ConfigureServices` existente pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="1144b-112">Replace the existing `ConfigureServices` function with the following.</span></span>

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

1. <span data-ttu-id="1144b-113">Na `Configure` função, adicione a seguinte linha acima da `app.UseAuthorization();` linha.</span><span class="sxs-lookup"><span data-stu-id="1144b-113">In the `Configure` function, add the following line above the `app.UseAuthorization();` line.</span></span>

    ```csharp
    app.UseAuthentication();
    ```

1. <span data-ttu-id="1144b-114">Abra **./Controllers/HomeController.cs** e substitua seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="1144b-114">Open **./Controllers/HomeController.cs** and replace its contents with the following.</span></span>

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

1. <span data-ttu-id="1144b-115">Salve suas alterações e inicie o projeto.</span><span class="sxs-lookup"><span data-stu-id="1144b-115">Save your changes and start the project.</span></span> <span data-ttu-id="1144b-116">Faça logon com sua conta da Microsoft.</span><span class="sxs-lookup"><span data-stu-id="1144b-116">Login with your Microsoft account.</span></span>

1. <span data-ttu-id="1144b-117">Examine a solicitação de consentimento.</span><span class="sxs-lookup"><span data-stu-id="1144b-117">Examine the consent prompt.</span></span> <span data-ttu-id="1144b-118">A lista de permissões corresponde à lista de escopos de permissões configurados em **./Graph/GraphConstants.cs**.</span><span class="sxs-lookup"><span data-stu-id="1144b-118">The list of permissions correspond to list of permissions scopes configured in **./Graph/GraphConstants.cs**.</span></span>

    - <span data-ttu-id="1144b-119">**Mantenha acesso aos dados aos** que você deu a eles acesso: ( ) Essa permissão é solicitada pela MSAL para recuperar `offline_access` tokens de atualização.</span><span class="sxs-lookup"><span data-stu-id="1144b-119">**Maintain access to data you have given it access to:** (`offline_access`) This permission is requested by MSAL in order to retrieve refresh tokens.</span></span>
    - <span data-ttu-id="1144b-120">**Entre e leia seu perfil:** ( ) Essa permissão permite que o aplicativo receba o perfil do usuário conectado e a foto `User.Read` do perfil.</span><span class="sxs-lookup"><span data-stu-id="1144b-120">**Sign you in and read your profile:** (`User.Read`) This permission allows the app to get the logged-in user's profile and profile photo.</span></span>
    - <span data-ttu-id="1144b-121">**Leia as configurações da** caixa de correio: ( ) Essa permissão permite que o aplicativo leia as configurações da caixa de correio do usuário, incluindo fuso horário e `MailboxSettings.Read` formato de hora.</span><span class="sxs-lookup"><span data-stu-id="1144b-121">**Read your mailbox settings:** (`MailboxSettings.Read`) This permission allows the app to read the user's mailbox settings, including time zone and time format.</span></span>
    - <span data-ttu-id="1144b-122">Tenha acesso total aos seus **calendários:** ( ) Essa permissão permite que o aplicativo leia eventos no calendário do usuário, adicione novos eventos e modifique `Calendars.ReadWrite` os existentes.</span><span class="sxs-lookup"><span data-stu-id="1144b-122">**Have full access to your calendars:** (`Calendars.ReadWrite`) This permission allows the app to read events on the user's calendar, add new events, and modify existing ones.</span></span>

    ![Uma captura de tela do prompt de consentimento da Plataforma de Identidade da Microsoft](./images/add-aad-auth-03.png)

    <span data-ttu-id="1144b-124">Para obter mais informações sobre consentimento, consulte Noções básicas sobre experiências de consentimento [do aplicativo Azure AD.](/azure/active-directory/develop/application-consent-experience)</span><span class="sxs-lookup"><span data-stu-id="1144b-124">For more information regarding consent, see [Understanding Azure AD application consent experiences](/azure/active-directory/develop/application-consent-experience).</span></span>

1. <span data-ttu-id="1144b-125">Consentir com as permissões solicitadas.</span><span class="sxs-lookup"><span data-stu-id="1144b-125">Consent to the requested permissions.</span></span> <span data-ttu-id="1144b-126">O navegador redireciona para o aplicativo, mostrando o token.</span><span class="sxs-lookup"><span data-stu-id="1144b-126">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="1144b-127">Obter detalhes do usuário</span><span class="sxs-lookup"><span data-stu-id="1144b-127">Get user details</span></span>

<span data-ttu-id="1144b-128">Depois que o usuário está conectado, você pode obter as informações dele no Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="1144b-128">Once the user is logged in, you can get their information from Microsoft Graph.</span></span>

1. <span data-ttu-id="1144b-129">Abra **./Graph/GraphClaimsPrincipalExtensions.cs** e substitua todo o conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="1144b-129">Open **./Graph/GraphClaimsPrincipalExtensions.cs** and replace its entire contents with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Graph/GraphClaimsPrincipalExtensions.cs" id="GraphClaimsExtensionsSnippet":::

1. <span data-ttu-id="1144b-130">Abra **./Startup.cs** e substitua a linha `.AddMicrosoftIdentityWebApp(Configuration)` existente pelo código a seguir.</span><span class="sxs-lookup"><span data-stu-id="1144b-130">Open **./Startup.cs** and replace the existing `.AddMicrosoftIdentityWebApp(Configuration)` line with the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddSignInSnippet":::

    <span data-ttu-id="1144b-131">Considere o que esse código faz.</span><span class="sxs-lookup"><span data-stu-id="1144b-131">Consider what this code does.</span></span>

    - <span data-ttu-id="1144b-132">Ele adiciona um manipulador de eventos para o `OnTokenValidated` evento.</span><span class="sxs-lookup"><span data-stu-id="1144b-132">It adds an event handler for the `OnTokenValidated` event.</span></span>
        - <span data-ttu-id="1144b-133">Ele usa a `ITokenAcquisition` interface para obter um token de acesso.</span><span class="sxs-lookup"><span data-stu-id="1144b-133">It uses the `ITokenAcquisition` interface to get an access token.</span></span>
        - <span data-ttu-id="1144b-134">Ele chama o Microsoft Graph para obter o perfil e a foto do usuário.</span><span class="sxs-lookup"><span data-stu-id="1144b-134">It calls Microsoft Graph to get the user's profile and photo.</span></span>
        - <span data-ttu-id="1144b-135">Ele adiciona as informações do Graph à identidade do usuário.</span><span class="sxs-lookup"><span data-stu-id="1144b-135">It adds the Graph information to the user's identity.</span></span>

1. <span data-ttu-id="1144b-136">Adicione a seguinte chamada de função após `EnableTokenAcquisitionToCallDownstreamApi` a chamada e antes da `AddInMemoryTokenCaches` chamada.</span><span class="sxs-lookup"><span data-stu-id="1144b-136">Add the following function call after the `EnableTokenAcquisitionToCallDownstreamApi` call and before the `AddInMemoryTokenCaches` call.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Startup.cs" id="AddGraphClientSnippet":::

    <span data-ttu-id="1144b-137">Isso disponibiliza um **GraphServiceClient** autenticado para controladores por meio de injeção de dependência.</span><span class="sxs-lookup"><span data-stu-id="1144b-137">This will make an authenticated **GraphServiceClient** available to controllers via dependency injection.</span></span>

1. <span data-ttu-id="1144b-138">Abra **./Controllers/HomeController.cs** e substitua `Index` a função pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="1144b-138">Open **./Controllers/HomeController.cs** and replace the `Index` function with the following.</span></span>

    ```csharp
    public IActionResult Index()
    {
        return View();
    }
    ```

1. <span data-ttu-id="1144b-139">Remova todas as referências `ITokenAcquisition` à classe **HomeController.**</span><span class="sxs-lookup"><span data-stu-id="1144b-139">Remove all references to `ITokenAcquisition` in the **HomeController** class.</span></span>

1. <span data-ttu-id="1144b-140">Salve suas alterações, inicie o aplicativo e vá pelo processo de login.</span><span class="sxs-lookup"><span data-stu-id="1144b-140">Save your changes, start the app, and go through the sign-in process.</span></span> <span data-ttu-id="1144b-141">Você deve voltar à home page, mas a interface do usuário deve mudar para indicar que você está entrar.</span><span class="sxs-lookup"><span data-stu-id="1144b-141">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Uma captura de tela da home page após entrar](./images/add-aad-auth-01.png)

1. <span data-ttu-id="1144b-143">Clique no avatar do usuário no canto superior direito para acessar o link **Sair.**</span><span class="sxs-lookup"><span data-stu-id="1144b-143">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="1144b-144">Clicar em **Sair** redefine a sessão e o retorna para a home page.</span><span class="sxs-lookup"><span data-stu-id="1144b-144">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Uma captura de tela do menu suspenso com o link Sair](./images/add-aad-auth-02.png)

> [!TIP]
> <span data-ttu-id="1144b-146">Se você não vir seu nome de usuário na home page e o nome e email do avatar de uso não estiver no nome e no email depois de fazer essas alterações, saia e entre novamente.</span><span class="sxs-lookup"><span data-stu-id="1144b-146">If you do not see your user name on the home page and the use avatar dropdown is missing name and email after making these changes, sign out and sign back in.</span></span>

## <a name="storing-and-refreshing-tokens"></a><span data-ttu-id="1144b-147">Armazenar e atualizar tokens</span><span class="sxs-lookup"><span data-stu-id="1144b-147">Storing and refreshing tokens</span></span>

<span data-ttu-id="1144b-148">Neste ponto, seu aplicativo tem um token de acesso, que é enviado no `Authorization` header de chamadas de API.</span><span class="sxs-lookup"><span data-stu-id="1144b-148">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="1144b-149">Esse é o token que permite que o aplicativo acesse o Microsoft Graph em nome do usuário.</span><span class="sxs-lookup"><span data-stu-id="1144b-149">This is the token that allows the app to access Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="1144b-150">No entanto, esse token tem curta duração.</span><span class="sxs-lookup"><span data-stu-id="1144b-150">However, this token is short-lived.</span></span> <span data-ttu-id="1144b-151">O token expira uma hora após sua emissão.</span><span class="sxs-lookup"><span data-stu-id="1144b-151">The token expires an hour after it is issued.</span></span> <span data-ttu-id="1144b-152">É aqui que o token de atualização se torna útil.</span><span class="sxs-lookup"><span data-stu-id="1144b-152">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="1144b-153">O token de atualização permite que o aplicativo solicite um novo token de acesso sem exigir que o usuário entre novamente.</span><span class="sxs-lookup"><span data-stu-id="1144b-153">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span>

<span data-ttu-id="1144b-154">Como o aplicativo está usando a biblioteca Microsoft.Identity.Web, você não precisa implementar nenhum armazenamento de token ou lógica de atualização.</span><span class="sxs-lookup"><span data-stu-id="1144b-154">Because the app is using the Microsoft.Identity.Web library, you do not have to implement any token storage or refresh logic.</span></span>

<span data-ttu-id="1144b-155">O aplicativo usa o cache de token na memória, o que é suficiente para aplicativos que não precisam persistir tokens quando o aplicativo é reiniciado.</span><span class="sxs-lookup"><span data-stu-id="1144b-155">The app uses the in-memory token cache, which is sufficient for apps that do not need to persist tokens when the app restarts.</span></span> <span data-ttu-id="1144b-156">Em vez disso, os aplicativos de produção podem usar [as opções de cache distribuído](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) na biblioteca Microsoft.Identity.Web.</span><span class="sxs-lookup"><span data-stu-id="1144b-156">Production apps may instead use the [distributed cache options](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization) in the Microsoft.Identity.Web library.</span></span>

<span data-ttu-id="1144b-157">O `GetAccessTokenForUserAsync` método lida com a expiração do token e a atualização para você.</span><span class="sxs-lookup"><span data-stu-id="1144b-157">The `GetAccessTokenForUserAsync` method handles token expiration and refresh for you.</span></span> <span data-ttu-id="1144b-158">Primeiro, ele verifica o token armazenado em cache e, se não tiver expirado, ele o retornará.</span><span class="sxs-lookup"><span data-stu-id="1144b-158">It first checks the cached token, and if it is not expired, it returns it.</span></span> <span data-ttu-id="1144b-159">Se ele tiver expirado, ele usará o token de atualização em cache para obter um novo.</span><span class="sxs-lookup"><span data-stu-id="1144b-159">If it is expired, it uses the cached refresh token to obtain a new one.</span></span>

<span data-ttu-id="1144b-160">O **GraphServiceClient que** os controladores receberem por injeção de dependência será pré-configurado com um provedor de autenticação que usa `GetAccessTokenForUserAsync` para você.</span><span class="sxs-lookup"><span data-stu-id="1144b-160">The **GraphServiceClient** that controllers get via dependency injection will be pre-configured with an authentication provider that uses `GetAccessTokenForUserAsync` for you.</span></span>
