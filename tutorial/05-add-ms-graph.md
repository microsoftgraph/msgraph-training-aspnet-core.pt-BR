---
ms.openlocfilehash: c954903f38d48bcca4c534f4d0cfbe1605cbf7f6
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942145"
---
<!-- markdownlint-disable MD002 MD041 -->

Nesta seção, você incorporará o Microsoft Graph ao aplicativo. Para este aplicativo, você usará a Biblioteca de Cliente do [Microsoft Graph para .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) para fazer chamadas para o Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Obter eventos de calendário do Outlook

Comece criando um novo controlador para exibições de calendário.

1. Adicione um novo arquivo **CalendarController.cs** no diretório **./Controllers** e adicione o código a seguir.

    ```csharp
    using GraphTutorial.Models;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Logging;
    using Microsoft.Identity.Web;
    using Microsoft.Graph;
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using TimeZoneConverter;

    namespace GraphTutorial.Controllers
    {
        public class CalendarController : Controller
        {
            private readonly GraphServiceClient _graphClient;
            private readonly ILogger<HomeController> _logger;

            public CalendarController(
                GraphServiceClient graphClient,
                ILogger<HomeController> logger)
            {
                _graphClient = graphClient;
                _logger = logger;
            }
        }
    }
    ```

1. Adicione as funções a seguir à `CalendarController` classe para obter a exibição de calendário do usuário.

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="GetCalendarViewSnippet":::

    Considere o que o código `GetUserWeekCalendar` em faz.

    - Ele usa o fuso horário do usuário para obter valores de data/hora de início e término UTC para a semana.
    - Ele consulta a exibição de [calendário](/graph/api/calendar-list-calendarview?view=graph-rest-1.0) do usuário para obter todos os eventos que se enquadram entre a data/hora de início e de término. Usar uma exibição de calendário em vez de [listar](/graph/api/user-list-events?view=graph-rest-1.0) eventos expande eventos recorrentes, retornando quaisquer ocorrências que ocorram na janela de tempo especificada.
    - Ele usa o `Prefer: outlook.timezone` header para obter resultados de volta no zona de tempo do usuário.
    - Ele usa `Select` para limitar os campos que voltarão apenas aos usados pelo aplicativo.
    - Ele usa `OrderBy` para classificar os resultados cronologicamente.
    - Ele usa um `PageIterator` para página através da coleção de [eventos](/graph/sdks/paging). Isso lida com o caso em que o usuário tem mais eventos em seu calendário do que o tamanho de página solicitado.

1. Adicione a função a seguir à `CalendarController` classe para implementar uma exibição temporária dos dados retornados.

    ```csharp
    // Minimum permission scope needed for this view
    [AuthorizeForScopes(Scopes = new[] { "Calendars.Read" })]
    public async Task<IActionResult> Index()
    {
        try
        {
            var userTimeZone = TZConvert.GetTimeZoneInfo(
                User.GetUserGraphTimeZone());
            var startOfWeek = CalendarController.GetUtcStartOfWeekInTimeZone(
                DateTime.Today, userTimeZone);

            var events = await GetUserWeekCalendar(startOfWeek);

            // Return a JSON dump of events
            return new ContentResult {
                Content = _graphClient.HttpProvider.Serializer.SerializeObject(events),
                ContentType = "application/json"
            };
        }
        catch (ServiceException ex)
        {
            if (ex.InnerException is MicrosoftIdentityWebChallengeUserException)
            {
                throw;
            }

            return new ContentResult {
                Content = $"Error getting calendar view: {ex.Message}",
                ContentType = "text/plain"
            };
        }
    }
    ```

1. Inicie o aplicativo, entre e clique **no** link Calendário na barra de inv. Se tudo funcionar, você deverá ver um despejo JSON de eventos no calendário do usuário.

## <a name="display-the-results"></a>Exibir os resultados

Agora você pode adicionar um modo de exibição para exibir os resultados de uma maneira mais amigável.

### <a name="create-view-models"></a>Criar modelos de exibição

1. Crie um novo arquivo **chamado CalendarViewEvent.cs** no diretório **./Models** e adicione o código a seguir.

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewEvent.cs" id="CalendarViewEventSnippet":::

1. Crie um novo arquivo **chamado DailyViewModel.cs** no diretório **./Models** e adicione o código a seguir.

    :::code language="csharp" source="../demo/GraphTutorial/Models/DailyViewModel.cs" id="DailyViewModelSnippet":::

1. Crie um novo arquivo **chamado CalendarViewModel.cs** no diretório **./Models** e adicione o código a seguir.

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewModel.cs" id="CalendarViewModelSnippet":::

### <a name="create-views"></a>Criar exibições

1. Crie um novo diretório chamado **Calendário** no **diretório ./Views.**

1. Crie um novo arquivo **chamado _DailyEventsPartial.cshtml** no diretório **./Views/Calendar** e adicione o código a seguir.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/_DailyEventsPartial.cshtml" id="DailyEventsPartialSnippet":::

1. Crie um novo arquivo chamado **Index.cshtml** no diretório **./Views/Calendar** e adicione o código a seguir.

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/Index.cshtml" id="CalendarIndexSnippet":::

### <a name="update-calendar-controller"></a>Atualizar o controlador de calendário

1. Abra **./Controllers/CalendarController.cs** e substitua a função `Index` existente pelo seguinte.

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="IndexSnippet":::

1. Inicie o aplicativo, entre e clique **no** link Calendário. O aplicativo agora deve renderizar uma tabela de eventos.

    ![Uma captura de tela da tabela de eventos](./images/add-msgraph-01.png)
