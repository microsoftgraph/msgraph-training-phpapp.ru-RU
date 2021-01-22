<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="1cb78-101">В этом упражнении вы включаете Microsoft Graph в приложение.</span><span class="sxs-lookup"><span data-stu-id="1cb78-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="1cb78-102">Для этого приложения вы будете использовать библиотеку [Microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) для вызова Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="1cb78-102">For this application, you will use the [microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="1cb78-103">Получить события календаря из Outlook</span><span class="sxs-lookup"><span data-stu-id="1cb78-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="1cb78-104">Создайте новый каталог в **каталоге ./app** с именем , а затем создайте новый файл с именем и `TimeZones` добавьте следующий `TimeZones.php` код.</span><span class="sxs-lookup"><span data-stu-id="1cb78-104">Create a new directory in the **./app** directory named `TimeZones`, then create a new file in that directory named `TimeZones.php`, and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TimeZones/TimeZones.php":::

    <span data-ttu-id="1cb78-105">Этот класс реализует упрощенное сопоставление имен часового пояса Windows с идентификаторами часового пояса IANA.</span><span class="sxs-lookup"><span data-stu-id="1cb78-105">This class implements a simplistic mapping of Windows time zone names to IANA time zone identifiers.</span></span>

1. <span data-ttu-id="1cb78-106">Создайте файл с именем **./app/Http/Controllers** и добавьте `CalendarController.php` следующий код.</span><span class="sxs-lookup"><span data-stu-id="1cb78-106">Create a new file in the **./app/Http/Controllers** directory named `CalendarController.php`, and add the following code.</span></span>

    ```php
    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Microsoft\Graph\Graph;
    use Microsoft\Graph\Model;
    use App\TokenStore\TokenCache;
    use App\TimeZones\TimeZones;

    class CalendarController extends Controller
    {
      public function calendar()
      {
        $viewData = $this->loadViewData();

        $graph = $this->getGraph();

        // Get user's timezone
        $timezone = TimeZones::getTzFromWindows($viewData['userTimeZone']);

        // Get start and end of week
        $startOfWeek = new \DateTimeImmutable('sunday -1 week', $timezone);
        $endOfWeek = new \DateTimeImmutable('sunday', $timezone);

        $queryParams = array(
          'startDateTime' => $startOfWeek->format(\DateTimeInterface::ISO8601),
          'endDateTime' => $endOfWeek->format(\DateTimeInterface::ISO8601),
          // Only request the properties used by the app
          '$select' => 'subject,organizer,start,end',
          // Sort them by start time
          '$orderby' => 'start/dateTime',
          // Limit results to 25
          '$top' => 25
        );

        // Append query parameters to the '/me/calendarView' url
        $getEventsUrl = '/me/calendarView?'.http_build_query($queryParams);

        $events = $graph->createRequest('GET', $getEventsUrl)
          // Add the user's timezone to the Prefer header
          ->addHeaders(array(
            'Prefer' => 'outlook.timezone="'.$viewData['userTimeZone'].'"'
          ))
          ->setReturnType(Model\Event::class)
          ->execute();

        return response()->json($events);
      }

      private function getGraph(): Graph
      {
        // Get the access token from the cache
        $tokenCache = new TokenCache();
        $accessToken = $tokenCache->getAccessToken();

        // Create a Graph client
        $graph = new Graph();
        $graph->setAccessToken($accessToken);
        return $graph;
      }
    }
    ```

    <span data-ttu-id="1cb78-107">Подумайте, что делает этот код.</span><span class="sxs-lookup"><span data-stu-id="1cb78-107">Consider what this code is doing.</span></span>

    - <span data-ttu-id="1cb78-108">Будет вызван URL-адрес `/v1.0/me/calendarView` .</span><span class="sxs-lookup"><span data-stu-id="1cb78-108">The URL that will be called is `/v1.0/me/calendarView`.</span></span>
    - <span data-ttu-id="1cb78-109">Параметры `startDateTime` `endDateTime` и параметров определяют начало и конец представления.</span><span class="sxs-lookup"><span data-stu-id="1cb78-109">The `startDateTime` and `endDateTime` parameters define the start and end of the view.</span></span>
    - <span data-ttu-id="1cb78-110">Параметр ограничивает поля, возвращаемые для каждого события, только теми, которые будут `$select` фактически использовать представление.</span><span class="sxs-lookup"><span data-stu-id="1cb78-110">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
    - <span data-ttu-id="1cb78-111">Параметр сортировать результаты по дате и времени их создания, причем последним элементом является `$orderby` первый.</span><span class="sxs-lookup"><span data-stu-id="1cb78-111">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>
    - <span data-ttu-id="1cb78-112">Этот `$top` параметр ограничивает результаты до 25 событий.</span><span class="sxs-lookup"><span data-stu-id="1cb78-112">The `$top` parameter limits the results to 25 events.</span></span>
    - <span data-ttu-id="1cb78-113">За header приводит к корректировке времени начала и окончания отклика в предпочитаемый часовой `Prefer: outlook.timezone=""` пояс пользователя.</span><span class="sxs-lookup"><span data-stu-id="1cb78-113">The `Prefer: outlook.timezone=""` header causes the start and end times in the response to be adjusted to the user's preferred time zone.</span></span>

1. <span data-ttu-id="1cb78-114">Обновим маршруты в **./routes/web.php,** чтобы добавить маршрут к новому контроллеру.</span><span class="sxs-lookup"><span data-stu-id="1cb78-114">Update the routes in **./routes/web.php** to add a route to this new controller.</span></span>

    ```php
    Route::get('/calendar', 'CalendarController@calendar');
    ```

1. <span data-ttu-id="1cb78-115">Войдите и щелкните **ссылку "Календарь"** на панели nav.</span><span class="sxs-lookup"><span data-stu-id="1cb78-115">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="1cb78-116">Если все работает, в календаре пользователя должен быть дамп событий JSON.</span><span class="sxs-lookup"><span data-stu-id="1cb78-116">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="1cb78-117">Отображение результатов</span><span class="sxs-lookup"><span data-stu-id="1cb78-117">Display the results</span></span>

<span data-ttu-id="1cb78-118">Теперь можно добавить представление для более удобного отображения результатов.</span><span class="sxs-lookup"><span data-stu-id="1cb78-118">Now you can add a view to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="1cb78-119">Создайте файл с именем **./resources/views** и `calendar.blade.php` добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="1cb78-119">Create a new file in the **./resources/views** directory named `calendar.blade.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/calendar.blade.php" id="CalendarSnippet":::

    <span data-ttu-id="1cb78-120">При этом будет цикл через коллекцию событий и добавлена строка таблицы для каждого из них.</span><span class="sxs-lookup"><span data-stu-id="1cb78-120">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="1cb78-121">Обновим маршруты в **./routes/web.php,** чтобы добавить маршруты для `/calendar/new` .</span><span class="sxs-lookup"><span data-stu-id="1cb78-121">Update the routes in **./routes/web.php** to add routes for `/calendar/new`.</span></span> <span data-ttu-id="1cb78-122">Эти функции будут реализованы в следующем разделе, но теперь необходимо определить маршрут, так как на него ссылается **calendar.blade.php.**</span><span class="sxs-lookup"><span data-stu-id="1cb78-122">You will implement these functions in the next section, but the route need to be defined now because **calendar.blade.php** references it.</span></span>

    ```php
    Route::get('/calendar/new', 'CalendarController@getNewEventForm');
    Route::post('/calendar/new', 'CalendarController@createNewEvent');
    ```

1. <span data-ttu-id="1cb78-123">Удалите строку из действия `return response()->json($events);` `calendar` в **./app/Http/Controllers/CalendarController.php** и замените ее на следующий код.</span><span class="sxs-lookup"><span data-stu-id="1cb78-123">Remove the `return response()->json($events);` line from the `calendar` action in **./app/Http/Controllers/CalendarController.php**, and replace it with the following code.</span></span>

    ```php
    $viewData['events'] = $events;
    return view('calendar', $viewData);
    ```

1. <span data-ttu-id="1cb78-124">Обновите страницу, и приложение должно отрисовки таблицы событий.</span><span class="sxs-lookup"><span data-stu-id="1cb78-124">Refresh the page and the app should now render a table of events.</span></span>

    ![Снимок экрана с таблицей событий](./images/add-msgraph-01.png)
