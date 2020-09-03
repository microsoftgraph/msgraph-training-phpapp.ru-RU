<!-- markdownlint-disable MD002 MD041 -->

В этом разделе мы добавим возможность создания событий в календаре пользователя.

## <a name="create-new-event-form"></a>Форма создания нового события

1. Создайте новый файл в каталоге **./ресаурцес/виевс** `newevent.blade.php` и добавьте указанный ниже код.

    :::code language="php" source="../demo/graph-tutorial/resources/views/newevent.blade.php" id="NewEventFormSnippet":::

## <a name="add-controller-actions"></a>Добавление действий контроллера

1. Откройте **./АПП/хттп/контроллерс/календарконтроллер.ФП** и добавьте приведенную ниже функцию, чтобы отобразить форму.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="getNewEventFormSnippet":::

1. Добавьте следующую функцию для получения данных формы при отсылке пользователя и создания нового события в календаре пользователя.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="createNewEventSnippet":::

    Рассмотрите, что делает этот код.

    - Он преобразует входные данные участников в массив объектов [участников](https://docs.microsoft.com/graph/api/resources/attendee?view=graph-rest-1.0) графа.
    - Он создает [событие](https://docs.microsoft.com/graph/api/resources/event?view=graph-rest-1.0) из ввода формы.
    - Он отправляет сообщение на `/me/events` конечную точку, а затем перенаправляет обратно в представление календаря.

1. Обновите маршруты в файле **./раутес/веб.ФП** , чтобы добавить маршруты для новых функций на контроллере.

    ```php
    Route::get('/calendar/new', 'CalendarController@getNewEventForm');
    Route::post('/calendar/new', 'CalendarController@createNewEvent');
    ```

1. Сохраните все изменения и перезапустите сервер. Используйте кнопку **создать событие** , чтобы перейти к форме создания события.

1. Заполните значения в форме. Использовать дату начала текущей недели. Нажмите **Создать**.

    ![Снимок экрана с формой создания события](images/create-event-01.png)

1. Когда приложение перенаправляется в представление календаря, убедитесь, что новое событие присутствует в результатах.
