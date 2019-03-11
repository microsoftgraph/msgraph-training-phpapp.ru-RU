<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="b6fac-101">В этом упражнении вы добавите Microsoft Graph в приложение.</span><span class="sxs-lookup"><span data-stu-id="b6fac-101">In this exercise you will incorporate the Microsoft Graph into the application.</span></span> <span data-ttu-id="b6fac-102">Для этого приложения вы будете использовать библиотеку [Microsoft Graph](https://github.com/microsoftgraph/msgraph-sdk-php) , чтобы совершать вызовы в Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="b6fac-102">For this application, you will use the [microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) library to make calls to Microsoft Graph.</span></span>

## <a name="get-calendar-events-from-outlook"></a><span data-ttu-id="b6fac-103">Получение событий календаря из Outlook</span><span class="sxs-lookup"><span data-stu-id="b6fac-103">Get calendar events from Outlook</span></span>

<span data-ttu-id="b6fac-104">Для начала добавим контроллер для представления календаря.</span><span class="sxs-lookup"><span data-stu-id="b6fac-104">Let's start by adding a controller for the calendar view.</span></span> <span data-ttu-id="b6fac-105">Создайте новый файл в `./app/Http/Controllers` папке с именем `CalendarController.php`и добавьте приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="b6fac-105">Create a new file in the `./app/Http/Controllers` folder named `CalendarController.php`, and add the following code.</span></span>

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

<span data-ttu-id="b6fac-106">РасСмотрите, что делает этот код.</span><span class="sxs-lookup"><span data-stu-id="b6fac-106">Consider what this code is doing.</span></span>

- <span data-ttu-id="b6fac-107">URL-адрес, который будет вызываться — это `/v1.0/me/events`.</span><span class="sxs-lookup"><span data-stu-id="b6fac-107">The URL that will be called is `/v1.0/me/events`.</span></span>
- <span data-ttu-id="b6fac-108">`$select` Параметр позволяет ограничить поля, возвращаемые для каждого события, только теми, которые будут реально использоваться представлением.</span><span class="sxs-lookup"><span data-stu-id="b6fac-108">The `$select` parameter limits the fields returned for each events to just those the view will actually use.</span></span>
- <span data-ttu-id="b6fac-109">`$orderby` Параметр сортирует результаты по дате и времени создания, начиная с самого последнего элемента.</span><span class="sxs-lookup"><span data-stu-id="b6fac-109">The `$orderby` parameter sorts the results by the date and time they were created, with the most recent item being first.</span></span>

<span data-ttu-id="b6fac-110">Обновление маршрутов в `./routes/web.php` Добавление маршрута к этому новому контроллеру</span><span class="sxs-lookup"><span data-stu-id="b6fac-110">Update the routes in `./routes/web.php` to add a route to this new controller</span></span>

```php
Route::get('/calendar', 'CalendarController@calendar');
```

<span data-ttu-id="b6fac-111">Теперь вы можете протестировать это.</span><span class="sxs-lookup"><span data-stu-id="b6fac-111">Now you can test this.</span></span> <span data-ttu-id="b6fac-112">Войдите и щелкните ссылку **Календарь** на панели навигации.</span><span class="sxs-lookup"><span data-stu-id="b6fac-112">Sign in and click the **Calendar** link in the nav bar.</span></span> <span data-ttu-id="b6fac-113">Если все работает, вы должны увидеть дамп событий JSON в календаре пользователя.</span><span class="sxs-lookup"><span data-stu-id="b6fac-113">If everything works, you should see a JSON dump of events on the user's calendar.</span></span>

## <a name="display-the-results"></a><span data-ttu-id="b6fac-114">Отображение результатов</span><span class="sxs-lookup"><span data-stu-id="b6fac-114">Display the results</span></span>

<span data-ttu-id="b6fac-115">Теперь вы можете добавить представление для отображения результатов более удобным для пользователя способом.</span><span class="sxs-lookup"><span data-stu-id="b6fac-115">Now you can add a view to display the results in a more user-friendly manner.</span></span> <span data-ttu-id="b6fac-116">Создайте новый файл в `./resources/views` каталоге `calendar.blade.php` и добавьте указанный ниже код.</span><span class="sxs-lookup"><span data-stu-id="b6fac-116">Create a new file in the `./resources/views` directory named `calendar.blade.php` and add the following code.</span></span>

```php
@extends('layout')

@section('content')
<h1>Calendar</h1>
<table class="table">
  <thead>
    <tr>
      <th scope="col">Organizer</th>
      <th scope="col">Subject</th>
      <th scope="col">Start</th>
      <th scope="col">End</th>
    </tr>
  </thead>
  <tbody>
    @isset($events)
      @foreach($events as $event)
        <tr>
          <td>{{ $event->getOrganizer()->getEmailAddress()->getName() }}</td>
          <td>{{ $event->getSubject() }}</td>
          <td>{{ \Carbon\Carbon::parse($event->getStart()->getDateTime())->format('n/j/y g:i A') }}</td>
          <td>{{ \Carbon\Carbon::parse($event->getEnd()->getDateTime())->format('n/j/y g:i A') }}</td>
        </tr>
      @endforeach
    @endif
  </tbody>
</table>
@endsection
```

<span data-ttu-id="b6fac-117">Это приведет к перебору коллекции событий и добавлению строки таблицы для каждой из них.</span><span class="sxs-lookup"><span data-stu-id="b6fac-117">That will loop through a collection of events and add a table row for each one.</span></span> <span data-ttu-id="b6fac-118">Удалите `return response()->json($events);` строку из `calendar` действия в `./app/Http/Controllers/CalendarController.php`и замените ее на приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="b6fac-118">Remove the `return response()->json($events);` line from the `calendar` action in `./app/Http/Controllers/CalendarController.php`, and replace it with the following code.</span></span>

```php
$viewData['events'] = $events;
return view('calendar', $viewData);
```

<span data-ttu-id="b6fac-119">Обновите страницу, после чего приложение должно отобразить таблицу событий.</span><span class="sxs-lookup"><span data-stu-id="b6fac-119">Refresh the page and the app should now render a table of events.</span></span>

![Снимок экрана С таблицей событий](./images/add-msgraph-01.png)