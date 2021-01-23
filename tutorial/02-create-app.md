---
ms.openlocfilehash: 5b1a776c28b6f9218c713dde68f45e571ebfd999
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942152"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="a9f80-101">Comece criando um ASP.NET Aplicativo Web Principal.</span><span class="sxs-lookup"><span data-stu-id="a9f80-101">Start by creating an ASP.NET Core web app.</span></span>

1. <span data-ttu-id="a9f80-102">Abra sua CLI (interface de linha de comando) em um diretório onde você deseja criar o projeto.</span><span class="sxs-lookup"><span data-stu-id="a9f80-102">Open your command-line interface (CLI) in a directory where you want to create the project.</span></span> <span data-ttu-id="a9f80-103">Execute o seguinte comando.</span><span class="sxs-lookup"><span data-stu-id="a9f80-103">Run the following command.</span></span>

    ```Shell
    dotnet new mvc -o GraphTutorial
    ```

1. <span data-ttu-id="a9f80-104">Depois que o projeto for criado, verifique se ele funciona alterando o diretório atual para o diretório **GraphTutorial** e executando o seguinte comando em sua CLI.</span><span class="sxs-lookup"><span data-stu-id="a9f80-104">Once the project is created, verify that it works by changing the current directory to the **GraphTutorial** directory and running the following command in your CLI.</span></span>

    ```Shell
    dotnet run
    ```

1. <span data-ttu-id="a9f80-105">Abra seu navegador e navegue `https://localhost:5001` até.</span><span class="sxs-lookup"><span data-stu-id="a9f80-105">Open your browser and browse to `https://localhost:5001`.</span></span> <span data-ttu-id="a9f80-106">Se tudo estiver funcionando, você deverá ver uma página principal ASP.NET padrão.</span><span class="sxs-lookup"><span data-stu-id="a9f80-106">If everything is working, you should see a default ASP.NET Core page.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="a9f80-107">Se você receber um aviso de que o certificado do **localhost** não é confiável, poderá usar a CLI do .NET Core para instalar e confiar no certificado de desenvolvimento.</span><span class="sxs-lookup"><span data-stu-id="a9f80-107">If you receive a warning that the certificate for **localhost** is un-trusted you can use the .NET Core CLI to install and trust the development certificate.</span></span> <span data-ttu-id="a9f80-108">Consulte [Enforce HTTPS in ASP.NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0) para obter instruções para sistemas operacionais específicos.</span><span class="sxs-lookup"><span data-stu-id="a9f80-108">See [Enforce HTTPS in ASP.NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0) for instructions for specific operating systems.</span></span>

## <a name="add-nuget-packages"></a><span data-ttu-id="a9f80-109">Adicionar pacotes NuGet</span><span class="sxs-lookup"><span data-stu-id="a9f80-109">Add NuGet packages</span></span>

<span data-ttu-id="a9f80-110">Antes de continuar, instale alguns pacotes NuGet adicionais que você usará mais tarde.</span><span class="sxs-lookup"><span data-stu-id="a9f80-110">Before moving on, install some additional NuGet packages that you will use later.</span></span>

- <span data-ttu-id="a9f80-111">[Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) para solicitar e gerenciar tokens de acesso.</span><span class="sxs-lookup"><span data-stu-id="a9f80-111">[Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) for requesting and managing access tokens.</span></span>
- <span data-ttu-id="a9f80-112">[Microsoft.Identity.Web.MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) para adicionar o SDK do Microsoft Graph por meio de injeção de dependência.</span><span class="sxs-lookup"><span data-stu-id="a9f80-112">[Microsoft.Identity.Web.MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) for adding the Microsoft Graph SDK via dependency injection.</span></span>
- <span data-ttu-id="a9f80-113">[Microsoft.Identity.Web.UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/) para interface do usuário de login e saída.</span><span class="sxs-lookup"><span data-stu-id="a9f80-113">[Microsoft.Identity.Web.UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/) for sign-in and sign-out UI.</span></span>
- <span data-ttu-id="a9f80-114">[TimeZoneConverter para](https://github.com/mj1856/TimeZoneConverter) lidar com identificadores de fuso horário entre plataformas.</span><span class="sxs-lookup"><span data-stu-id="a9f80-114">[TimeZoneConverter](https://github.com/mj1856/TimeZoneConverter) for handling time zoned identifiers cross-platform.</span></span>

1. <span data-ttu-id="a9f80-115">Execute os seguintes comandos em sua CLI para instalar as dependências.</span><span class="sxs-lookup"><span data-stu-id="a9f80-115">Run the following commands in your CLI to install the dependencies.</span></span>

    ```Shell
    dotnet add package Microsoft.Identity.Web --version 1.5.1
    dotnet add package Microsoft.Identity.Web.MicrosoftGraph --version 1.5.1
    dotnet add package Microsoft.Identity.Web.UI --version 1.5.1
    dotnet add package TimeZoneConverter
    ```

## <a name="design-the-app"></a><span data-ttu-id="a9f80-116">Projetar o aplicativo</span><span class="sxs-lookup"><span data-stu-id="a9f80-116">Design the app</span></span>

<span data-ttu-id="a9f80-117">Nesta seção, você criará a estrutura básica da interface do usuário do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="a9f80-117">In this section you will create the basic UI structure of the application.</span></span>

### <a name="implement-alert-extension-methods"></a><span data-ttu-id="a9f80-118">Implementar métodos de extensão de alerta</span><span class="sxs-lookup"><span data-stu-id="a9f80-118">Implement alert extension methods</span></span>

<span data-ttu-id="a9f80-119">Nesta seção, você criará métodos de extensão para o tipo retornado `IActionResult` pelas exibições do controlador.</span><span class="sxs-lookup"><span data-stu-id="a9f80-119">In this section you will create extension methods for the `IActionResult` type returned by controller views.</span></span> <span data-ttu-id="a9f80-120">Essa extensão permitirá passar mensagens temporárias de erro ou de sucesso para o modo de exibição.</span><span class="sxs-lookup"><span data-stu-id="a9f80-120">This extension will enable passing temporary error or success messages to the view.</span></span>

> [!TIP]
> <span data-ttu-id="a9f80-121">Você pode usar qualquer editor de texto para editar os arquivos de origem deste tutorial.</span><span class="sxs-lookup"><span data-stu-id="a9f80-121">You can use any text editor to edit the source files for this tutorial.</span></span> <span data-ttu-id="a9f80-122">No entanto, [o Visual Studio Code](https://code.visualstudio.com/) fornece recursos adicionais, como depuração e Intellisense.</span><span class="sxs-lookup"><span data-stu-id="a9f80-122">However, [Visual Studio Code](https://code.visualstudio.com/) provides additional features, such as debugging and Intellisense.</span></span>

1. <span data-ttu-id="a9f80-123">Crie um novo diretório no diretório **GraphTutorial** chamado **Alertas.**</span><span class="sxs-lookup"><span data-stu-id="a9f80-123">Create a new directory in the **GraphTutorial** directory named **Alerts**.</span></span>

1. <span data-ttu-id="a9f80-124">Crie um novo arquivo **WithAlertResult.cs** no diretório **./Alerts** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="a9f80-124">Create a new file named **WithAlertResult.cs** in the **./Alerts** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/WithAlertResult.cs" id="WithAlertResultSnippet":::

1. <span data-ttu-id="a9f80-125">Crie um novo arquivo **AlertExtensions.cs** no diretório **./Alerts** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="a9f80-125">Create a new file named **AlertExtensions.cs** in the **./Alerts** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/AlertExtensions.cs" id="AlertExtensionsSnippet":::

### <a name="implement-user-data-extension-methods"></a><span data-ttu-id="a9f80-126">Implementar métodos de extensão de dados do usuário</span><span class="sxs-lookup"><span data-stu-id="a9f80-126">Implement user data extension methods</span></span>

<span data-ttu-id="a9f80-127">Nesta seção, você criará métodos de extensão para o `ClaimsPrincipal` objeto gerado pela Plataforma de Identidade da Microsoft.</span><span class="sxs-lookup"><span data-stu-id="a9f80-127">In this section you will create extension methods for the `ClaimsPrincipal` object generated by the Microsoft Identity platform.</span></span> <span data-ttu-id="a9f80-128">Isso permitirá que você estenda a identidade do usuário existente com dados do Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="a9f80-128">This will allow you to extend the existing user identity with data from Microsoft Graph.</span></span>

> [!NOTE]
> <span data-ttu-id="a9f80-129">Este código é apenas um espaço reservado por enquanto, você o concluirá em uma seção posterior.</span><span class="sxs-lookup"><span data-stu-id="a9f80-129">This code is just a placeholder for now, you will complete it in a later section.</span></span>

1. <span data-ttu-id="a9f80-130">Crie um novo diretório no diretório **GraphTutorial** chamado **Graph**.</span><span class="sxs-lookup"><span data-stu-id="a9f80-130">Create a new directory in the **GraphTutorial** directory named **Graph**.</span></span>

1. <span data-ttu-id="a9f80-131">Crie um novo arquivo chamado **GraphClaimsPrincipalExtensions.cs** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="a9f80-131">Create a new file named **GraphClaimsPrincipalExtensions.cs** and add the following code.</span></span>

    ```csharp
    using System.Security.Claims;

    namespace GraphTutorial
    {
        public static class GraphClaimTypes {
            public const string DisplayName ="graph_name";
            public const string Email = "graph_email";
            public const string Photo = "graph_photo";
            public const string TimeZone = "graph_timezone";
            public const string DateTimeFormat = "graph_datetimeformat";
        }

        // Helper methods to access Graph user data stored in
        // the claims principal
        public static class GraphClaimsPrincipalExtensions
        {
            public static string GetUserGraphDisplayName(this ClaimsPrincipal claimsPrincipal)
            {
                return "Adele Vance";
            }

            public static string GetUserGraphEmail(this ClaimsPrincipal claimsPrincipal)
            {
                return "adelev@contoso.com";
            }

            public static string GetUserGraphPhoto(this ClaimsPrincipal claimsPrincipal)
            {
                return "/img/no-profile-photo.png";
            }
        }
    }
    ```

### <a name="create-views"></a><span data-ttu-id="a9f80-132">Criar exibições</span><span class="sxs-lookup"><span data-stu-id="a9f80-132">Create views</span></span>

<span data-ttu-id="a9f80-133">Nesta seção, você implementará os exibições de Views para o aplicativo.</span><span class="sxs-lookup"><span data-stu-id="a9f80-133">In this section you will implement the Razor views for the application.</span></span>

1. <span data-ttu-id="a9f80-134">Adicione um novo arquivo **chamado _LoginPartial.cshtml** no diretório **./Views/Shared** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="a9f80-134">Add a new file named **_LoginPartial.cshtml** in the **./Views/Shared** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_LoginPartial.cshtml" id="LoginPartialSnippet":::

1. <span data-ttu-id="a9f80-135">Adicione um novo arquivo **chamado _AlertPartial.cshtml** no diretório **./Views/Shared** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="a9f80-135">Add a new file named **_AlertPartial.cshtml** in the **./Views/Shared** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_AlertPartial.cshtml" id="AlertPartialSnippet":::

1. <span data-ttu-id="a9f80-136">Abra o **arquivo ./Views/Shared/_Layout.cshtml** e substitua todo o conteúdo pelo código a seguir para atualizar o layout global do aplicativo.</span><span class="sxs-lookup"><span data-stu-id="a9f80-136">Open the **./Views/Shared/_Layout.cshtml** file, and replace its entire contents with the following code to update the global layout of the app.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_Layout.cshtml" id="LayoutSnippet":::

1. <span data-ttu-id="a9f80-137">Abra **./wwwroot/css/site.css** e adicione o seguinte código na parte inferior do arquivo.</span><span class="sxs-lookup"><span data-stu-id="a9f80-137">Open **./wwwroot/css/site.css** and add the following code at the bottom of the file.</span></span>

    :::code language="css" source="../demo/GraphTutorial/wwwroot/css/site.css" id="CssSnippet":::

1. <span data-ttu-id="a9f80-138">Abra o **arquivo ./Views/Home/index.cshtml** e substitua seu conteúdo pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="a9f80-138">Open the **./Views/Home/index.cshtml** file and replace its contents with the following.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Home/Index.cshtml" id="HomeIndexSnippet":::

1. <span data-ttu-id="a9f80-139">Crie um novo diretório no **diretório ./wwwroot** chamado **img.**</span><span class="sxs-lookup"><span data-stu-id="a9f80-139">Create a new directory in the **./wwwroot** directory named **img**.</span></span> <span data-ttu-id="a9f80-140">Adicione um arquivo de imagem de sua escolha **no-profile-photo.png** neste diretório.</span><span class="sxs-lookup"><span data-stu-id="a9f80-140">Add an image file of your choosing named **no-profile-photo.png** in this directory.</span></span> <span data-ttu-id="a9f80-141">Essa imagem será usada como a foto do usuário quando o usuário não tiver foto no Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="a9f80-141">This image will be used as the user's photo when the user has no photo in Microsoft Graph.</span></span>

    > [!TIP]
    > <span data-ttu-id="a9f80-142">Você pode baixar a imagem usada nessas capturas de tela do [GitHub.](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png)</span><span class="sxs-lookup"><span data-stu-id="a9f80-142">You can download the image used in these screenshots from [GitHub](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png).</span></span>

1. <span data-ttu-id="a9f80-143">Salve todas as suas alterações e reinicie o servidor ( `dotnet run` ).</span><span class="sxs-lookup"><span data-stu-id="a9f80-143">Save all of your changes and restart the server (`dotnet run`).</span></span> <span data-ttu-id="a9f80-144">Agora, o aplicativo deve parecer muito diferente.</span><span class="sxs-lookup"><span data-stu-id="a9f80-144">Now, the app should look very different.</span></span>

    ![Uma captura de tela da home page reprojetada](./images/create-app-01.png)
