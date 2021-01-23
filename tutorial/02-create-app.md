---
ms.openlocfilehash: 5b1a776c28b6f9218c713dde68f45e571ebfd999
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942152"
---
<!-- markdownlint-disable MD002 MD041 -->

Comece criando um ASP.NET Aplicativo Web Principal.

1. Abra sua CLI (interface de linha de comando) em um diretório onde você deseja criar o projeto. Execute o seguinte comando.

    ```Shell
    dotnet new mvc -o GraphTutorial
    ```

1. Depois que o projeto for criado, verifique se ele funciona alterando o diretório atual para o diretório **GraphTutorial** e executando o seguinte comando em sua CLI.

    ```Shell
    dotnet run
    ```

1. Abra seu navegador e navegue `https://localhost:5001` até. Se tudo estiver funcionando, você deverá ver uma página principal ASP.NET padrão.

> [!IMPORTANT]
> Se você receber um aviso de que o certificado do **localhost** não é confiável, poderá usar a CLI do .NET Core para instalar e confiar no certificado de desenvolvimento. Consulte [Enforce HTTPS in ASP.NET Core](/aspnet/core/security/enforcing-ssl?view=aspnetcore-5.0) para obter instruções para sistemas operacionais específicos.

## <a name="add-nuget-packages"></a>Adicionar pacotes NuGet

Antes de continuar, instale alguns pacotes NuGet adicionais que você usará mais tarde.

- [Microsoft.Identity.Web](https://www.nuget.org/packages/Microsoft.Identity.Web/) para solicitar e gerenciar tokens de acesso.
- [Microsoft.Identity.Web.MicrosoftGraph](https://www.nuget.org/packages/Microsoft.Identity.Web.MicrosoftGraph/) para adicionar o SDK do Microsoft Graph por meio de injeção de dependência.
- [Microsoft.Identity.Web.UI](https://www.nuget.org/packages/Microsoft.Identity.Web.UI/) para interface do usuário de login e saída.
- [TimeZoneConverter para](https://github.com/mj1856/TimeZoneConverter) lidar com identificadores de fuso horário entre plataformas.

1. Execute os seguintes comandos em sua CLI para instalar as dependências.

    ```Shell
    dotnet add package Microsoft.Identity.Web --version 1.5.1
    dotnet add package Microsoft.Identity.Web.MicrosoftGraph --version 1.5.1
    dotnet add package Microsoft.Identity.Web.UI --version 1.5.1
    dotnet add package TimeZoneConverter
    ```

## <a name="design-the-app"></a>Projetar o aplicativo

Nesta seção, você criará a estrutura básica da interface do usuário do aplicativo.

### <a name="implement-alert-extension-methods"></a>Implementar métodos de extensão de alerta

Nesta seção, você criará métodos de extensão para o tipo retornado `IActionResult` pelas exibições do controlador. Essa extensão permitirá passar mensagens temporárias de erro ou de sucesso para o modo de exibição.

> [!TIP]
> Você pode usar qualquer editor de texto para editar os arquivos de origem deste tutorial. No entanto, [o Visual Studio Code](https://code.visualstudio.com/) fornece recursos adicionais, como depuração e Intellisense.

1. Crie um novo diretório no diretório **GraphTutorial** chamado **Alertas.**

1. Crie um novo arquivo **WithAlertResult.cs** no diretório **./Alerts** e adicione o código a seguir.

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/WithAlertResult.cs" id="WithAlertResultSnippet":::

1. Crie um novo arquivo **AlertExtensions.cs** no diretório **./Alerts** e adicione o código a seguir.

    :::code language="csharp" source="../demo/GraphTutorial/Alerts/AlertExtensions.cs" id="AlertExtensionsSnippet":::

### <a name="implement-user-data-extension-methods"></a>Implementar métodos de extensão de dados do usuário

Nesta seção, você criará métodos de extensão para o `ClaimsPrincipal` objeto gerado pela Plataforma de Identidade da Microsoft. Isso permitirá que você estenda a identidade do usuário existente com dados do Microsoft Graph.

> [!NOTE]
> Este código é apenas um espaço reservado por enquanto, você o concluirá em uma seção posterior.

1. Crie um novo diretório no diretório **GraphTutorial** chamado **Graph**.

1. Crie um novo arquivo chamado **GraphClaimsPrincipalExtensions.cs** e adicione o código a seguir.

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

### <a name="create-views"></a>Criar exibições

Nesta seção, você implementará os exibições de Views para o aplicativo.

1. Adicione um novo arquivo **chamado _LoginPartial.cshtml** no diretório **./Views/Shared** e adicione o código a seguir.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_LoginPartial.cshtml" id="LoginPartialSnippet":::

1. Adicione um novo arquivo **chamado _AlertPartial.cshtml** no diretório **./Views/Shared** e adicione o código a seguir.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_AlertPartial.cshtml" id="AlertPartialSnippet":::

1. Abra o **arquivo ./Views/Shared/_Layout.cshtml** e substitua todo o conteúdo pelo código a seguir para atualizar o layout global do aplicativo.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Shared/_Layout.cshtml" id="LayoutSnippet":::

1. Abra **./wwwroot/css/site.css** e adicione o seguinte código na parte inferior do arquivo.

    :::code language="css" source="../demo/GraphTutorial/wwwroot/css/site.css" id="CssSnippet":::

1. Abra o **arquivo ./Views/Home/index.cshtml** e substitua seu conteúdo pelo seguinte.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Home/Index.cshtml" id="HomeIndexSnippet":::

1. Crie um novo diretório no **diretório ./wwwroot** chamado **img.** Adicione um arquivo de imagem de sua escolha **no-profile-photo.png** neste diretório. Essa imagem será usada como a foto do usuário quando o usuário não tiver foto no Microsoft Graph.

    > [!TIP]
    > Você pode baixar a imagem usada nessas capturas de tela do [GitHub.](https://github.com/microsoftgraph/msgraph-training-aspnet-core/blob/master/demo/GraphTutorial/wwwroot/img/no-profile-photo.png)

1. Salve todas as suas alterações e reinicie o servidor ( `dotnet run` ). Agora, o aplicativo deve parecer muito diferente.

    ![Uma captura de tela da home page reprojetada](./images/create-app-01.png)
