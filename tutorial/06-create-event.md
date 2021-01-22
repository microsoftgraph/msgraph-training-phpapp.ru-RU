<!-- markdownlint-disable MD002 MD041 -->

В этом разделе вы добавим возможность создания событий в календаре пользователя.

## <a name="create-new-event-form"></a>Создание формы события

1. Создайте файл с именем **./resources/views** и `newevent.blade.php` добавьте следующий код.

    :::code language="php" source="../demo/graph-tutorial/resources/views/newevent.blade.php" id="NewEventFormSnippet":::

## <a name="add-controller-actions"></a>Добавление действий контроллера

1. Откройте **./app/Http/Controllers/CalendarController.php** и добавьте следующую функцию для отрисовки формы.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="getNewEventFormSnippet":::

1. Добавьте следующую функцию, чтобы получать данные формы при отправке пользователем, и создайте новое событие в календаре пользователя.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="createNewEventSnippet":::

    Подумайте, что делает этот код.

    - Он преобразует входные данные поля участников в массив объектов [участников](https://docs.microsoft.com/graph/api/resources/attendee?view=graph-rest-1.0) Graph.
    - Он создает событие [на](https://docs.microsoft.com/graph/api/resources/event?view=graph-rest-1.0) форме ввода.
    - Она отправляет POST в конечную точку, а затем перенаправляет `/me/events` обратно в представление календаря.

1. Сохраните все изменения и перезапустите сервер. Используйте **кнопку "Создать** событие", чтобы перейти к новой форме события.

1. Заполните значения формы. Используйте дату начала текущей недели. Нажмите кнопку **Создать**.

    ![Снимок экрана с новой формой события](images/create-event-01.png)

1. Когда приложение перенаправляется в представление календаря, убедитесь, что новое событие присутствует в результатах.
