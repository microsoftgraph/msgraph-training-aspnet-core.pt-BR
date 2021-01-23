---
ms.openlocfilehash: c954903f38d48bcca4c534f4d0cfbe1605cbf7f6
ms.sourcegitcommit: 6341ad07cd5b03269e7fd20cd3212e48baee7c07
ms.translationtype: MT
ms.contentlocale: pt-BR
ms.lasthandoff: 01/23/2021
ms.locfileid: "49942145"
---
<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="9fe80-101">Nesta seção, você incorporará o Microsoft Graph ao aplicativo.</span><span class="sxs-lookup"><span data-stu-id="9fe80-101">In this section you will incorporate Microsoft Graph into the application.</span></span> <span data-ttu-id="9fe80-102">Para este aplicativo, você usará a Biblioteca de Cliente do [Microsoft Graph para .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) para fazer chamadas para o Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="9fe80-102">For this application, you will use the [Microsoft Graph Client Library for .NET](https://github.com/microsoftgraph/msgraph-sdk-dotnet) to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="9fe80-103">Obter eventos de calendário do Outlook</span><span class="sxs-lookup"><span data-stu-id="9fe80-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="9fe80-104">Comece criando um novo controlador para exibições de calendário.</span><span class="sxs-lookup"><span data-stu-id="9fe80-104">Start by creating a new controller for calendar views.</span></span>

1. <span data-ttu-id="9fe80-105">Adicione um novo arquivo **CalendarController.cs** no diretório **./Controllers** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="9fe80-105">Add a new file named **CalendarController.cs** in the **./Controllers** directory and add the following code.</span></span>

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

1. <span data-ttu-id="9fe80-106">Adicione as funções a seguir à `CalendarController` classe para obter a exibição de calendário do usuário.</span><span class="sxs-lookup"><span data-stu-id="9fe80-106">Add the following functions to the `CalendarController` class to get the user's calendar view.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="GetCalendarViewSnippet":::

    <span data-ttu-id="9fe80-107">Considere o que o código `GetUserWeekCalendar` em faz.</span><span class="sxs-lookup"><span data-stu-id="9fe80-107">Consider what the code in `GetUserWeekCalendar` does.</span></span>

    - <span data-ttu-id="9fe80-108">Ele usa o fuso horário do usuário para obter valores de data/hora de início e término UTC para a semana.</span><span class="sxs-lookup"><span data-stu-id="9fe80-108">It uses the user's time zone to get UTC start and end date/time values for the week.</span></span>
    - <span data-ttu-id="9fe80-109">Ele consulta a exibição de [calendário](/graph/api/calendar-list-calendarview?view=graph-rest-1.0) do usuário para obter todos os eventos que se enquadram entre a data/hora de início e de término.</span><span class="sxs-lookup"><span data-stu-id="9fe80-109">It queries the user's [calendar view](/graph/api/calendar-list-calendarview?view=graph-rest-1.0) to get all events that fall between the start and end date/times.</span></span> <span data-ttu-id="9fe80-110">Usar uma exibição de calendário em vez de [listar](/graph/api/user-list-events?view=graph-rest-1.0) eventos expande eventos recorrentes, retornando quaisquer ocorrências que ocorram na janela de tempo especificada.</span><span class="sxs-lookup"><span data-stu-id="9fe80-110">Using a calendar view instead of [listing events](/graph/api/user-list-events?view=graph-rest-1.0) expands recurring events, returning any occurrences that happen in the specified time window.</span></span>
    - <span data-ttu-id="9fe80-111">Ele usa o `Prefer: outlook.timezone` header para obter resultados de volta no zona de tempo do usuário.</span><span class="sxs-lookup"><span data-stu-id="9fe80-111">It uses the `Prefer: outlook.timezone` header to get results back in the user's timezone.</span></span>
    - <span data-ttu-id="9fe80-112">Ele usa `Select` para limitar os campos que voltarão apenas aos usados pelo aplicativo.</span><span class="sxs-lookup"><span data-stu-id="9fe80-112">It uses `Select` to limit the fields that come back to just those used by the app.</span></span>
    - <span data-ttu-id="9fe80-113">Ele usa `OrderBy` para classificar os resultados cronologicamente.</span><span class="sxs-lookup"><span data-stu-id="9fe80-113">It uses `OrderBy` to sort the results chronologically.</span></span>
    - <span data-ttu-id="9fe80-114">Ele usa um `PageIterator` para página através da coleção de [eventos](/graph/sdks/paging).</span><span class="sxs-lookup"><span data-stu-id="9fe80-114">It uses a `PageIterator` to [page through the events collection](/graph/sdks/paging).</span></span> <span data-ttu-id="9fe80-115">Isso lida com o caso em que o usuário tem mais eventos em seu calendário do que o tamanho de página solicitado.</span><span class="sxs-lookup"><span data-stu-id="9fe80-115">This handles the case where the user has more events on their calendar than the requested page size.</span></span>

1. <span data-ttu-id="9fe80-116">Adicione a função a seguir à `CalendarController` classe para implementar uma exibição temporária dos dados retornados.</span><span class="sxs-lookup"><span data-stu-id="9fe80-116">Add the following function to the `CalendarController` class to implement a temporary view of the returned data.</span></span>

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

1. <span data-ttu-id="9fe80-117">Inicie o aplicativo, entre e clique **no** link Calendário na barra de inv.</span><span class="sxs-lookup"><span data-stu-id="9fe80-117">Start the app, sign in, and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="9fe80-118">Se tudo funcionar, você deverá ver um despejo JSON de eventos no calendário do usuário.</span><span class="sxs-lookup"><span data-stu-id="9fe80-118">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="9fe80-119">Exibir os resultados</span><span class="sxs-lookup"><span data-stu-id="9fe80-119">Display the results</span></span>

<span data-ttu-id="9fe80-120">Agora você pode adicionar um modo de exibição para exibir os resultados de uma maneira mais amigável.</span><span class="sxs-lookup"><span data-stu-id="9fe80-120">Now you can add a view to display the results in a more user-friendly manner.</span></span>

### <a name="create-view-models"></a><span data-ttu-id="9fe80-121">Criar modelos de exibição</span><span class="sxs-lookup"><span data-stu-id="9fe80-121">Create view models</span></span>

1. <span data-ttu-id="9fe80-122">Crie um novo arquivo **chamado CalendarViewEvent.cs** no diretório **./Models** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="9fe80-122">Create a new file named **CalendarViewEvent.cs** in the **./Models** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewEvent.cs" id="CalendarViewEventSnippet":::

1. <span data-ttu-id="9fe80-123">Crie um novo arquivo **chamado DailyViewModel.cs** no diretório **./Models** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="9fe80-123">Create a new file named **DailyViewModel.cs** in the **./Models** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Models/DailyViewModel.cs" id="DailyViewModelSnippet":::

1. <span data-ttu-id="9fe80-124">Crie um novo arquivo **chamado CalendarViewModel.cs** no diretório **./Models** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="9fe80-124">Create a new file named **CalendarViewModel.cs** in the **./Models** directory and add the following code.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Models/CalendarViewModel.cs" id="CalendarViewModelSnippet":::

### <a name="create-views"></a><span data-ttu-id="9fe80-125">Criar exibições</span><span class="sxs-lookup"><span data-stu-id="9fe80-125">Create views</span></span>

1. <span data-ttu-id="9fe80-126">Crie um novo diretório chamado **Calendário** no **diretório ./Views.**</span><span class="sxs-lookup"><span data-stu-id="9fe80-126">Create a new directory named **Calendar** in the **./Views** directory.</span></span>

1. <span data-ttu-id="9fe80-127">Crie um novo arquivo **chamado _DailyEventsPartial.cshtml** no diretório **./Views/Calendar** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="9fe80-127">Create a new file named **_DailyEventsPartial.cshtml** in the **./Views/Calendar** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/_DailyEventsPartial.cshtml" id="DailyEventsPartialSnippet":::

1. <span data-ttu-id="9fe80-128">Crie um novo arquivo chamado **Index.cshtml** no diretório **./Views/Calendar** e adicione o código a seguir.</span><span class="sxs-lookup"><span data-stu-id="9fe80-128">Create a new file named **Index.cshtml** in the **./Views/Calendar** directory and add the following code.</span></span>

    :::code language="cshtml" source="../demo/GraphTutorial/Views/Calendar/Index.cshtml" id="CalendarIndexSnippet":::

### <a name="update-calendar-controller"></a><span data-ttu-id="9fe80-129">Atualizar o controlador de calendário</span><span class="sxs-lookup"><span data-stu-id="9fe80-129">Update calendar controller</span></span>

1. <span data-ttu-id="9fe80-130">Abra **./Controllers/CalendarController.cs** e substitua a função `Index` existente pelo seguinte.</span><span class="sxs-lookup"><span data-stu-id="9fe80-130">Open **./Controllers/CalendarController.cs** and replace the existing `Index` function with the following.</span></span>

    :::code language="csharp" source="../demo/GraphTutorial/Controllers/CalendarController.cs" id="IndexSnippet":::

1. <span data-ttu-id="9fe80-131">Inicie o aplicativo, entre e clique **no** link Calendário.</span><span class="sxs-lookup"><span data-stu-id="9fe80-131">Start the app, sign in, and click the **Calendar** link.</span></span> <span data-ttu-id="9fe80-132">O aplicativo agora deve renderizar uma tabela de eventos.</span><span class="sxs-lookup"><span data-stu-id="9fe80-132">The app should now render a table of events.</span></span>

    ![Uma captura de tela da tabela de eventos](./images/add-msgraph-01.png)
