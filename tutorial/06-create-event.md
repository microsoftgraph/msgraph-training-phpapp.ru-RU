<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="d608a-101">В этом разделе вы добавим возможность создания событий в календаре пользователя.</span><span class="sxs-lookup"><span data-stu-id="d608a-101">In this section you will add the ability to create events on the user's calendar.</span></span>

## <a name="create-new-event-form"></a><span data-ttu-id="d608a-102">Создание формы события</span><span class="sxs-lookup"><span data-stu-id="d608a-102">Create new event form</span></span>

1. <span data-ttu-id="d608a-103">Создайте файл с именем **./resources/views** и `newevent.blade.php` добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="d608a-103">Create a new file in the **./resources/views** directory named `newevent.blade.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/newevent.blade.php" id="NewEventFormSnippet":::

## <a name="add-controller-actions"></a><span data-ttu-id="d608a-104">Добавление действий контроллера</span><span class="sxs-lookup"><span data-stu-id="d608a-104">Add controller actions</span></span>

1. <span data-ttu-id="d608a-105">Откройте **./app/Http/Controllers/CalendarController.php** и добавьте следующую функцию для отрисовки формы.</span><span class="sxs-lookup"><span data-stu-id="d608a-105">Open **./app/Http/Controllers/CalendarController.php** and add the following function to render the form.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="getNewEventFormSnippet":::

1. <span data-ttu-id="d608a-106">Добавьте следующую функцию, чтобы получать данные формы при отправке пользователем, и создайте новое событие в календаре пользователя.</span><span class="sxs-lookup"><span data-stu-id="d608a-106">Add the following function to receive the form data when the user's submits, and create a new event on the user's calendar.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/CalendarController.php" id="createNewEventSnippet":::

    <span data-ttu-id="d608a-107">Подумайте, что делает этот код.</span><span class="sxs-lookup"><span data-stu-id="d608a-107">Consider what this code does.</span></span>

    - <span data-ttu-id="d608a-108">Он преобразует входные данные поля участников в массив объектов [участников](https://docs.microsoft.com/graph/api/resources/attendee?view=graph-rest-1.0) Graph.</span><span class="sxs-lookup"><span data-stu-id="d608a-108">It converts the attendees field input to an array of Graph [attendee](https://docs.microsoft.com/graph/api/resources/attendee?view=graph-rest-1.0) objects.</span></span>
    - <span data-ttu-id="d608a-109">Он создает событие [на](https://docs.microsoft.com/graph/api/resources/event?view=graph-rest-1.0) форме ввода.</span><span class="sxs-lookup"><span data-stu-id="d608a-109">It builds an [event](https://docs.microsoft.com/graph/api/resources/event?view=graph-rest-1.0) from the form input.</span></span>
    - <span data-ttu-id="d608a-110">Она отправляет POST в конечную точку, а затем перенаправляет `/me/events` обратно в представление календаря.</span><span class="sxs-lookup"><span data-stu-id="d608a-110">It sends a POST to the `/me/events` endpoint, then redirects back to the calendar view.</span></span>

1. <span data-ttu-id="d608a-111">Сохраните все изменения и перезапустите сервер.</span><span class="sxs-lookup"><span data-stu-id="d608a-111">Save all of your changes and restart the server.</span></span> <span data-ttu-id="d608a-112">Используйте **кнопку "Создать** событие", чтобы перейти к новой форме события.</span><span class="sxs-lookup"><span data-stu-id="d608a-112">Use the **New event** button to navigate to the new event form.</span></span>

1. <span data-ttu-id="d608a-113">Заполните значения формы.</span><span class="sxs-lookup"><span data-stu-id="d608a-113">Fill in the values on the form.</span></span> <span data-ttu-id="d608a-114">Используйте дату начала текущей недели.</span><span class="sxs-lookup"><span data-stu-id="d608a-114">Use a start date from the current week.</span></span> <span data-ttu-id="d608a-115">Нажмите кнопку **Создать**.</span><span class="sxs-lookup"><span data-stu-id="d608a-115">Select **Create**.</span></span>

    ![Снимок экрана с новой формой события](images/create-event-01.png)

1. <span data-ttu-id="d608a-117">Когда приложение перенаправляется в представление календаря, убедитесь, что новое событие присутствует в результатах.</span><span class="sxs-lookup"><span data-stu-id="d608a-117">When the app redirects to the calendar view, verify that your new event is present in the results.</span></span>
