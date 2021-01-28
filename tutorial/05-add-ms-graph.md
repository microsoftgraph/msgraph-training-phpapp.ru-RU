<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы включаете Microsoft Graph в приложение. Для этого приложения вы будете использовать библиотеку [Microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) для вызова Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Получить события календаря из Outlook

1. Создайте новый каталог в **каталоге ./app** с именем, затем создайте новый файл в этом каталоге с именем и добавьте `TimeZones` следующий `TimeZones.php` код.

    :::code language="php" source="../demo/graph-tutorial/app/TimeZones/TimeZones.php":::

    Этот класс реализует упрощенное сопоставление имен часового пояса Windows с идентификаторами часового пояса IANA.

1. Создайте файл с именем **./app/Http/Controllers** и добавьте `CalendarController.php` следующий код.

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

        $viewData['dateRange'] = $startOfWeek->format('M j, Y').' - '.$endOfWeek->format('M j, Y');

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

    Подумайте, что делает этот код.

    - Будет вызван URL-адрес `/v1.0/me/calendarView` .
    - Параметры `startDateTime` `endDateTime` и параметров определяют начало и конец представления.
    - Параметр ограничивает поля, возвращаемые для каждого события, только теми, которые будут `$select` фактически использовать представление.
    - Параметр сортировать результаты по дате и времени их создания, причем последним элементом является `$orderby` первый.
    - Этот `$top` параметр ограничивает результаты до 25 событий.
    - За header приводит к корректировке времени начала и окончания отклика в предпочитаемый часовой `Prefer: outlook.timezone=""` пояс пользователя.

1. Обновим маршруты в **./routes/web.php,** чтобы добавить маршрут к новому контроллеру.

    ```php
    Route::get('/calendar', 'CalendarController@calendar');
    ```

1. Войдите и щелкните **ссылку "Календарь"** на панели nav. Если все работает, в календаре пользователя должен быть дамп событий JSON.

## <a name="display-the-results"></a>Отображение результатов

Теперь можно добавить представление для более удобного отображения результатов.

1. Создайте файл с именем **./resources/views** и `calendar.blade.php` добавьте следующий код.

    :::code language="php" source="../demo/graph-tutorial/resources/views/calendar.blade.php" id="CalendarSnippet":::

    При этом будет цикл через коллекцию событий и добавлена строка таблицы для каждого из них.

1. Обновим маршруты в **./routes/web.php,** чтобы добавить маршруты для `/calendar/new` . Эти функции будут реализованы в следующем разделе, но теперь необходимо определить маршрут, так как на него ссылается **calendar.blade.php.**

    ```php
    Route::get('/calendar/new', 'CalendarController@getNewEventForm');
    Route::post('/calendar/new', 'CalendarController@createNewEvent');
    ```

1. Удалите строку из действия `return response()->json($events);` `calendar` в **./app/Http/Controllers/CalendarController.php** и замените ее на следующий код.

    ```php
    $viewData['events'] = $events;
    return view('calendar', $viewData);
    ```

1. Обновите страницу, и приложение должно отрисовки таблицы событий.

    ![Снимок экрана с таблицей событий](./images/add-msgraph-01.png)
