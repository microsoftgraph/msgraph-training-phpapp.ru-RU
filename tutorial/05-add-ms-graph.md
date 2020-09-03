<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="69a6e-101">В этом упражнении вы добавите Microsoft Graph в приложение.</span><span class="sxs-lookup"><span data-stu-id="69a6e-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="69a6e-102">Для этого приложения вы будете использовать библиотеку [Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-php) , чтобы совершать вызовы в Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="69a6e-102">For this application, you will use the [microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="69a6e-103">Получение событий календаря из Outlook</span><span class="sxs-lookup"><span data-stu-id="69a6e-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="69a6e-104">Создайте каталог в каталоге **./АПП** с именем `TimeZones` , затем создайте в этом каталоге новый файл с именем `TimeZones.php` и добавьте приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="69a6e-104">Create a new directory in the **./app** directory named `TimeZones`, then create a new file in that directory named `TimeZones.php`, and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TimeZones/TimeZones.php":::

    <span data-ttu-id="69a6e-105">Этот класс реализует упрощенное сопоставление имен часовых поясов Windows с идентификаторами часовых поясов IANA.</span><span class="sxs-lookup"><span data-stu-id="69a6e-105">This class implements a simplistic mapping of Windows time zone names to IANA time zone identifiers.</span></span>

1. <span data-ttu-id="69a6e-106">Создайте новый файл в каталоге **./АПП/хттп/контроллерс** с именем `CalendarController.php` и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="69a6e-106">Create a new file in the **./app/Http/Controllers** directory named `CalendarController.php`, and add the following code.</span></span>

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

    <span data-ttu-id="69a6e-107">Рассмотрите, что делает этот код.</span><span class="sxs-lookup"><span data-stu-id="69a6e-107">Consider what this code is doing.</span></span>

    - <span data-ttu-id="69a6e-108">URL-адрес, который будет вызываться — это `/v1.0/me/calendarView` .</span><span class="sxs-lookup"><span data-stu-id="69a6e-108">The URL that will be called is `/v1.0/me/calendarView`.</span></span>
    - <span data-ttu-id="69a6e-109">`startDateTime`Параметры и `endDateTime` задают начало и конец представления.</span><span class="sxs-lookup"><span data-stu-id="69a6e-109">The `startDateTime` and `endDateTime` parameters define the start and end of the view.</span></span>
    - <span data-ttu-id="69a6e-110">`$select`Параметр позволяет ограничить поля, возвращаемые для каждого события, только теми, которые будут реально использоваться представлением.</span><span class="sxs-lookup"><span data-stu-id="69a6e-110">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
    - <span data-ttu-id="69a6e-111">`$orderby`Параметр сортирует результаты по дате и времени создания, начиная с самого последнего элемента.</span><span class="sxs-lookup"><span data-stu-id="69a6e-111">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>
    - <span data-ttu-id="69a6e-112">`$top`Параметр ограничит результаты до 25 событий.</span><span class="sxs-lookup"><span data-stu-id="69a6e-112">The `$top` parameter limits the results to 25 events.</span></span>
    - <span data-ttu-id="69a6e-113">`Prefer: outlook.timezone=""`Заголовок приводит к тому, что время начала и окончания в отклике изменяется на предпочтительный часовой пояс пользователя.</span><span class="sxs-lookup"><span data-stu-id="69a6e-113">The `Prefer: outlook.timezone=""` header causes the start and end times in the response to be adjusted to the user's preferred time zone.</span></span>

1. <span data-ttu-id="69a6e-114">Обновите маршруты в файле **./раутес/веб.ФП** , чтобы добавить маршрут к этому новому контроллеру.</span><span class="sxs-lookup"><span data-stu-id="69a6e-114">Update the routes in **./routes/web.php** to add a route to this new controller.</span></span>

    ```php
    Route::get('/calendar', 'CalendarController@calendar');
    ```

1. <span data-ttu-id="69a6e-115">Войдите и щелкните ссылку **Календарь** на панели навигации.</span><span class="sxs-lookup"><span data-stu-id="69a6e-115">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="69a6e-116">Если все работает, вы должны увидеть дамп событий JSON в календаре пользователя.</span><span class="sxs-lookup"><span data-stu-id="69a6e-116">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="69a6e-117">Отображение результатов</span><span class="sxs-lookup"><span data-stu-id="69a6e-117">Display the results</span></span>

<span data-ttu-id="69a6e-118">Теперь вы можете добавить представление для отображения результатов более удобным для пользователя способом.</span><span class="sxs-lookup"><span data-stu-id="69a6e-118">Now you can add a view to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="69a6e-119">Создайте новый файл в каталоге **./ресаурцес/виевс** `calendar.blade.php` и добавьте указанный ниже код.</span><span class="sxs-lookup"><span data-stu-id="69a6e-119">Create a new file in the **./resources/views** directory named `calendar.blade.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/calendar.blade.php" id="CalendarSnippet":::

    <span data-ttu-id="69a6e-120">Это приведет к перебору коллекции событий и добавлению строки таблицы для каждой из них.</span><span class="sxs-lookup"><span data-stu-id="69a6e-120">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="69a6e-121">Удалите `return response()->json($events);` строку из `calendar` действия в файле **./АПП/хттп/контроллерс/календарконтроллер.ФП**и замените ее на приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="69a6e-121">Remove the `return response()->json($events);` line from the `calendar` action in **./app/Http/Controllers/CalendarController.php**, and replace it with the following code.</span></span>

    ```php
    $viewData['events'] = $events;
    return view('calendar', $viewData);
    ```

1. <span data-ttu-id="69a6e-122">Обновите страницу, после чего приложение должно отобразить таблицу событий.</span><span class="sxs-lookup"><span data-stu-id="69a6e-122">Refresh the page and the app should now render a table of events.</span></span>

    ![Снимок экрана с таблицей событий](./images/add-msgraph-01.png)
