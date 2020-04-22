<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="cf185-101">В этом упражнении вы добавите Microsoft Graph в приложение.</span><span class="sxs-lookup"><span data-stu-id="cf185-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="cf185-102">Для этого приложения вы будете использовать библиотеку [Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-php) , чтобы совершать вызовы в Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="cf185-102">For this application, you will use the [microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="cf185-103">Получение событий календаря из Outlook</span><span class="sxs-lookup"><span data-stu-id="cf185-103">Get calendar events from Outlook</span></span>

1. <span data-ttu-id="cf185-104">Создайте новый файл в каталоге **./АПП/хттп/контроллерс** с именем `CalendarController.php`и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="cf185-104">Create a new file in the **./app/Http/Controllers** directory named `CalendarController.php`, and add the following code.</span></span>

    ```php
    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;
    use Microsoft\Graph\Graph;
    use Microsoft\Graph\Model;
    use App\TokenStore\TokenCache;

    class CalendarController extends Controller
    {
      public function calendar()
      {
        $viewData = $this->loadViewData();

        // Get the access token from the cache
        $tokenCache = new TokenCache();
        $accessToken = $tokenCache->getAccessToken();

        // Create a Graph client
        $graph = new Graph();
        $graph->setAccessToken($accessToken);

        $queryParams = array(
          '$select' => 'subject,organizer,start,end',
          '$orderby' => 'createdDateTime DESC'
        );

        // Append query parameters to the '/me/events' url
        $getEventsUrl = '/me/events?'.http_build_query($queryParams);

        $events = $graph->createRequest('GET', $getEventsUrl)
          ->setReturnType(Model\Event::class)
          ->execute();

        return response()->json($events);
      }
    }
    ```

    <span data-ttu-id="cf185-105">Рассмотрите, что делает этот код.</span><span class="sxs-lookup"><span data-stu-id="cf185-105">Consider what this code is doing.</span></span>

    - <span data-ttu-id="cf185-106">URL-адрес, который будет вызываться — это `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="cf185-106">The URL that will be called is `/v1.0/me/events`.</span></span>
    - <span data-ttu-id="cf185-107">`$select` Параметр позволяет ограничить поля, возвращаемые для каждого события, только теми, которые будут реально использоваться представлением.</span><span class="sxs-lookup"><span data-stu-id="cf185-107">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
    - <span data-ttu-id="cf185-108">`$orderby` Параметр сортирует результаты по дате и времени создания, начиная с самого последнего элемента.</span><span class="sxs-lookup"><span data-stu-id="cf185-108">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

1. <span data-ttu-id="cf185-109">Обновите маршруты в файле **./раутес/веб.ФП** , чтобы добавить маршрут к этому новому контроллеру.</span><span class="sxs-lookup"><span data-stu-id="cf185-109">Update the routes in **./routes/web.php** to add a route to this new controller.</span></span>

    ```php
    Route::get('/calendar', 'CalendarController@calendar');
    ```

1. <span data-ttu-id="cf185-110">Войдите и щелкните ссылку **Календарь** на панели навигации.</span><span class="sxs-lookup"><span data-stu-id="cf185-110">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="cf185-111">Если все работает, вы должны увидеть дамп событий JSON в календаре пользователя.</span><span class="sxs-lookup"><span data-stu-id="cf185-111">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="cf185-112">Отображение результатов</span><span class="sxs-lookup"><span data-stu-id="cf185-112">Display the results</span></span>

<span data-ttu-id="cf185-113">Теперь вы можете добавить представление для отображения результатов более удобным для пользователя способом.</span><span class="sxs-lookup"><span data-stu-id="cf185-113">Now you can add a view to display the results in a more user-friendly manner.</span></span>

1. <span data-ttu-id="cf185-114">Создайте новый файл в каталоге **./ресаурцес/виевс** `calendar.blade.php` и добавьте указанный ниже код.</span><span class="sxs-lookup"><span data-stu-id="cf185-114">Create a new file in the **./resources/views** directory named `calendar.blade.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/calendar.blade.php" id="CalendarSnippet":::

    <span data-ttu-id="cf185-115">Это приведет к перебору коллекции событий и добавлению строки таблицы для каждой из них.</span><span class="sxs-lookup"><span data-stu-id="cf185-115">That will loop through a collection of events and add a table row for each one.</span></span>

1. <span data-ttu-id="cf185-116">Удалите `return response()->json($events);` строку из `calendar` действия в файле **./АПП/хттп/контроллерс/календарконтроллер.ФП**и замените ее на приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="cf185-116">Remove the `return response()->json($events);` line from the `calendar` action in **./app/Http/Controllers/CalendarController.php**, and replace it with the following code.</span></span>

    ```php
    $viewData['events'] = $events;
    return view('calendar', $viewData);
    ```

1. <span data-ttu-id="cf185-117">Обновите страницу, после чего приложение должно отобразить таблицу событий.</span><span class="sxs-lookup"><span data-stu-id="cf185-117">Refresh the page and the app should now render a table of events.</span></span>

    ![Снимок экрана с таблицей событий](./images/add-msgraph-01.png)
