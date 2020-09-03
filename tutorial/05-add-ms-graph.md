<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы добавите Microsoft Graph в приложение. Для этого приложения вы будете использовать библиотеку [Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-php) , чтобы совершать вызовы в Microsoft Graph.

## <a name="get-calendar-events-from-outlook"></a>Получение событий календаря из Outlook

1. Создайте каталог в каталоге **./АПП** с именем `TimeZones` , затем создайте в этом каталоге новый файл с именем `TimeZones.php` и добавьте приведенный ниже код.

    :::code language="php" source="../demo/graph-tutorial/app/TimeZones/TimeZones.php":::

    Этот класс реализует упрощенное сопоставление имен часовых поясов Windows с идентификаторами часовых поясов IANA.

1. Создайте новый файл в каталоге **./АПП/хттп/контроллерс** с именем `CalendarController.php` и добавьте следующий код.

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

    Рассмотрите, что делает этот код.

    - URL-адрес, который будет вызываться — это `/v1.0/me/calendarView` .
    - `startDateTime`Параметры и `endDateTime` задают начало и конец представления.
    - `$select`Параметр позволяет ограничить поля, возвращаемые для каждого события, только теми, которые будут реально использоваться представлением.
    - `$orderby`Параметр сортирует результаты по дате и времени создания, начиная с самого последнего элемента.
    - `$top`Параметр ограничит результаты до 25 событий.
    - `Prefer: outlook.timezone=""`Заголовок приводит к тому, что время начала и окончания в отклике изменяется на предпочтительный часовой пояс пользователя.

1. Обновите маршруты в файле **./раутес/веб.ФП** , чтобы добавить маршрут к этому новому контроллеру.

    ```php
    Route::get('/calendar', 'CalendarController@calendar');
    ```

1. Войдите и щелкните ссылку **Календарь** на панели навигации. Если все работает, вы должны увидеть дамп событий JSON в календаре пользователя.

## <a name="display-the-results"></a>Отображение результатов

Теперь вы можете добавить представление для отображения результатов более удобным для пользователя способом.

1. Создайте новый файл в каталоге **./ресаурцес/виевс** `calendar.blade.php` и добавьте указанный ниже код.

    :::code language="php" source="../demo/graph-tutorial/resources/views/calendar.blade.php" id="CalendarSnippet":::

    Это приведет к перебору коллекции событий и добавлению строки таблицы для каждой из них.

1. Удалите `return response()->json($events);` строку из `calendar` действия в файле **./АПП/хттп/контроллерс/календарконтроллер.ФП**и замените ее на приведенный ниже код.

    ```php
    $viewData['events'] = $events;
    return view('calendar', $viewData);
    ```

1. Обновите страницу, после чего приложение должно отобразить таблицу событий.

    ![Снимок экрана с таблицей событий](./images/add-msgraph-01.png)
