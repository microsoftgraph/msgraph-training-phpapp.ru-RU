<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="055d2-101">Начните с создания нового проекта Ларавел.</span><span class="sxs-lookup"><span data-stu-id="055d2-101">Begin by creating a new Laravel project.</span></span>

1. <span data-ttu-id="055d2-102">Откройте интерфейс командной строки (CLI), перейдите к каталогу, в котором у вас есть права на создание файлов, и выполните следующую команду, чтобы создать новое приложение PHP.</span><span class="sxs-lookup"><span data-stu-id="055d2-102">Open your command-line interface (CLI), navigate to a directory where you have rights to create files, and run the following command to create a new PHP app.</span></span>

    ```Shell
    laravel new graph-tutorial
    ```

1. <span data-ttu-id="055d2-103">Перейдите к каталогу **Graph-Tutorial** и введите следующую команду для запуска локального веб-сервера.</span><span class="sxs-lookup"><span data-stu-id="055d2-103">Navigate to the **graph-tutorial** directory and enter the following command to start a local web server.</span></span>

    ```Shell
    php artisan serve
    ```

1. <span data-ttu-id="055d2-104">Откройте браузер и перейдите по адресу `http://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="055d2-104">Open your browser and navigate to `http://localhost:8000`.</span></span> <span data-ttu-id="055d2-105">Если все работает, вы увидите страницу Ларавел по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="055d2-105">If everything is working, you will see a default Laravel page.</span></span> <span data-ttu-id="055d2-106">Если вы не видите эту страницу, проверьте [документы ларавел](https://laravel.com/docs/7.x).</span><span class="sxs-lookup"><span data-stu-id="055d2-106">If you don't see that page, check the [Laravel docs](https://laravel.com/docs/7.x).</span></span>

## <a name="install-packages"></a><span data-ttu-id="055d2-107">Установка пакетов</span><span class="sxs-lookup"><span data-stu-id="055d2-107">Install packages</span></span>

<span data-ttu-id="055d2-108">Прежде чем переходить, установите несколько дополнительных пакетов, которые будут использоваться позже:</span><span class="sxs-lookup"><span data-stu-id="055d2-108">Before moving on, install some additional packages that you will use later:</span></span>

- <span data-ttu-id="055d2-109">[OAuth2 — клиент](https://github.com/thephpleague/oauth2-client) для обработки потоков маркеров входа и маркеров OAuth.</span><span class="sxs-lookup"><span data-stu-id="055d2-109">[oauth2-client](https://github.com/thephpleague/oauth2-client) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="055d2-110">[Microsoft](https://github.com/microsoftgraph/msgraph-sdk-php) Graph для совершения звонков в Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="055d2-110">[microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) for making calls to Microsoft Graph.</span></span>

1. <span data-ttu-id="055d2-111">Выполните следующую команду в командной панели CLI.</span><span class="sxs-lookup"><span data-stu-id="055d2-111">Run the following command in your CLI.</span></span>

    ```Shell
    composer require league/oauth2-client microsoft/microsoft-graph
    ```

## <a name="design-the-app"></a><span data-ttu-id="055d2-112">Проектирование приложения</span><span class="sxs-lookup"><span data-stu-id="055d2-112">Design the app</span></span>

1. <span data-ttu-id="055d2-113">Создайте новый файл в каталоге **./ресаурцес/виевс** `layout.blade.php` и добавьте указанный ниже код.</span><span class="sxs-lookup"><span data-stu-id="055d2-113">Create a new file in the **./resources/views** directory named `layout.blade.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/layout.blade.php" id="LayoutSnippet":::

    <span data-ttu-id="055d2-114">В этом коде добавляется [Начальная](http://getbootstrap.com/) загрузка для простых стилей и [Шрифт Awesome](https://fontawesome.com/) для некоторых простых значков.</span><span class="sxs-lookup"><span data-stu-id="055d2-114">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="055d2-115">Он также определяет глобальную структуру с помощью панели навигации.</span><span class="sxs-lookup"><span data-stu-id="055d2-115">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="055d2-116">Создайте `./public` новый каталог в каталоге с именем `css`, а затем создайте новый файл в `./public/css` каталоге с именем. `app.css`</span><span class="sxs-lookup"><span data-stu-id="055d2-116">Create a new directory in the `./public` directory named `css`, then create a new file in the `./public/css` directory named `app.css`.</span></span> <span data-ttu-id="055d2-117">Добавьте в него указанный ниже код.</span><span class="sxs-lookup"><span data-stu-id="055d2-117">Add the following code.</span></span>

    :::code language="css" source="../demo/graph-tutorial/public/css/app.css":::

1. <span data-ttu-id="055d2-118">Откройте `./resources/views/welcome.blade.php` файл и замените его содержимое на приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="055d2-118">Open the `./resources/views/welcome.blade.php` file and replace its contents with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/welcome.blade.php" id="WelcomeSnippet":::

1. <span data-ttu-id="055d2-119">Обновите базовый `Controller` класс в **/АПП/хттп/контроллерс/контроллер.ФП** , добавив приведенную ниже функцию в класс.</span><span class="sxs-lookup"><span data-stu-id="055d2-119">Update the base `Controller` class in **./app/Http/Controllers/Controller.php** by adding the following function to the class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/Controller.php" id="LoadViewDataSnippet":::

1. <span data-ttu-id="055d2-120">Создайте новый файл в `./app/Http/Controllers` каталоге `HomeController.php` и добавьте указанный ниже код.</span><span class="sxs-lookup"><span data-stu-id="055d2-120">Create a new file in the `./app/Http/Controllers` directory named `HomeController.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/HomeController.php":::

1. <span data-ttu-id="055d2-121">Обновите маршрут, `./routes/web.php` чтобы использовать новый контроллер.</span><span class="sxs-lookup"><span data-stu-id="055d2-121">Update the route in `./routes/web.php` to use the new controller.</span></span> <span data-ttu-id="055d2-122">Замените все содержимое этого файла на приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="055d2-122">Replace the entire contents of this file with the following.</span></span>

    ```php
    <?php

    use Illuminate\Support\Facades\Route;

    Route::get('/', 'HomeController@welcome');
    ```

1. <span data-ttu-id="055d2-123">Сохраните все изменения и перезапустите сервер.</span><span class="sxs-lookup"><span data-stu-id="055d2-123">Save all of your changes and restart the server.</span></span> <span data-ttu-id="055d2-124">Теперь приложение должно выглядеть по-другому.</span><span class="sxs-lookup"><span data-stu-id="055d2-124">Now, the app should look very different.</span></span>

    ![Снимок экрана с переработанной домашней страницей](./images/create-app-01.png)
