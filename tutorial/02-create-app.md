<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="27d5a-101">Начните с создания нового проекта Laravel.</span><span class="sxs-lookup"><span data-stu-id="27d5a-101">Begin by creating a new Laravel project.</span></span>

1. <span data-ttu-id="27d5a-102">Откройте интерфейс командной строки (CLI), перейдите в каталог, в котором у вас есть права на создание файлов, и запустите следующую команду, чтобы создать приложение PHP.</span><span class="sxs-lookup"><span data-stu-id="27d5a-102">Open your command-line interface (CLI), navigate to a directory where you have rights to create files, and run the following command to create a new PHP app.</span></span>

    ```Shell
    laravel new graph-tutorial
    ```

1. <span data-ttu-id="27d5a-103">Перейдите в **каталог учебника** по графику и введите следующую команду, чтобы запустить локальный веб-сервер.</span><span class="sxs-lookup"><span data-stu-id="27d5a-103">Navigate to the **graph-tutorial** directory and enter the following command to start a local web server.</span></span>

    ```Shell
    php artisan serve
    ```

1. <span data-ttu-id="27d5a-104">Откройте браузер и перейдите по адресу `http://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="27d5a-104">Open your browser and navigate to `http://localhost:8000`.</span></span> <span data-ttu-id="27d5a-105">Если все работает, вы увидите страницу Laravel по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="27d5a-105">If everything is working, you will see a default Laravel page.</span></span> <span data-ttu-id="27d5a-106">Если вы не видите эту страницу, проверьте [документы Laravel.](https://laravel.com/docs/8.x)</span><span class="sxs-lookup"><span data-stu-id="27d5a-106">If you don't see that page, check the [Laravel docs](https://laravel.com/docs/8.x).</span></span>

## <a name="install-packages"></a><span data-ttu-id="27d5a-107">Установка пакетов</span><span class="sxs-lookup"><span data-stu-id="27d5a-107">Install packages</span></span>

<span data-ttu-id="27d5a-108">Прежде чем двигаться дальше, установите некоторые дополнительные пакеты, которые вы будете использовать позже:</span><span class="sxs-lookup"><span data-stu-id="27d5a-108">Before moving on, install some additional packages that you will use later:</span></span>

- <span data-ttu-id="27d5a-109">[oauth2-client](https://github.com/thephpleague/oauth2-client) для обработки потоков входов и маркеров OAuth.</span><span class="sxs-lookup"><span data-stu-id="27d5a-109">[oauth2-client](https://github.com/thephpleague/oauth2-client) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="27d5a-110">[Microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) для звонков в Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="27d5a-110">[microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) for making calls to Microsoft Graph.</span></span>

1. <span data-ttu-id="27d5a-111">Запустите следующую команду в CLI.</span><span class="sxs-lookup"><span data-stu-id="27d5a-111">Run the following command in your CLI.</span></span>

    ```Shell
    composer require league/oauth2-client microsoft/microsoft-graph
    ```

## <a name="design-the-app"></a><span data-ttu-id="27d5a-112">Проектирование приложения</span><span class="sxs-lookup"><span data-stu-id="27d5a-112">Design the app</span></span>

1. <span data-ttu-id="27d5a-113">Создайте файл с именем **./resources/views** и `layout.blade.php` добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="27d5a-113">Create a new file in the **./resources/views** directory named `layout.blade.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/layout.blade.php" id="LayoutSnippet":::

    <span data-ttu-id="27d5a-114">В этом коде [добавляется Bootstrap](http://getbootstrap.com/) для простого стиля, а для некоторых простых значков [font : "Отличный".](https://fontawesome.com/)</span><span class="sxs-lookup"><span data-stu-id="27d5a-114">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="27d5a-115">Он также определяет глобальный макет с панели nav.</span><span class="sxs-lookup"><span data-stu-id="27d5a-115">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="27d5a-116">Создайте новый каталог в каталоге с именем , а затем создайте новый файл `./public` `css` в `./public/css` каталоге с именем `app.css` .</span><span class="sxs-lookup"><span data-stu-id="27d5a-116">Create a new directory in the `./public` directory named `css`, then create a new file in the `./public/css` directory named `app.css`.</span></span> <span data-ttu-id="27d5a-117">Добавьте в него указанный ниже код.</span><span class="sxs-lookup"><span data-stu-id="27d5a-117">Add the following code.</span></span>

    :::code language="css" source="../demo/graph-tutorial/public/css/app.css":::

1. <span data-ttu-id="27d5a-118">Откройте файл `./resources/views/welcome.blade.php` и замените его содержимое на следующее.</span><span class="sxs-lookup"><span data-stu-id="27d5a-118">Open the `./resources/views/welcome.blade.php` file and replace its contents with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/welcome.blade.php" id="WelcomeSnippet":::

1. <span data-ttu-id="27d5a-119">Обновив `Controller` базовый класс **в ./app/Http/Controllers/Controller.php,** добавив в класс следующую функцию.</span><span class="sxs-lookup"><span data-stu-id="27d5a-119">Update the base `Controller` class in **./app/Http/Controllers/Controller.php** by adding the following function to the class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/Controller.php" id="LoadViewDataSnippet":::

1. <span data-ttu-id="27d5a-120">Создайте новый файл в `./app/Http/Controllers` каталоге с именем `HomeController.php` и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="27d5a-120">Create a new file in the `./app/Http/Controllers` directory named `HomeController.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/HomeController.php":::

1. <span data-ttu-id="27d5a-121">Обновим маршрут `./routes/web.php` для использования нового контроллера.</span><span class="sxs-lookup"><span data-stu-id="27d5a-121">Update the route in `./routes/web.php` to use the new controller.</span></span> <span data-ttu-id="27d5a-122">Замените все содержимое этого файла на следующее.</span><span class="sxs-lookup"><span data-stu-id="27d5a-122">Replace the entire contents of this file with the following.</span></span>

    ```php
    <?php

    use Illuminate\Support\Facades\Route;

    Route::get('/', 'HomeController@welcome');
    ```

1. <span data-ttu-id="27d5a-123">Откройте **./app/Providers/RouteServiceProvider.php** и раскомментирование `$namespace` объявления.</span><span class="sxs-lookup"><span data-stu-id="27d5a-123">Open **./app/Providers/RouteServiceProvider.php** and uncomment the `$namespace` declaration.</span></span>

    ```php
    /**
     * This namespace is applied to your controller routes.
     *
     * In addition, it is set as the URL generator's root namespace.
     *
     * @var string
     */
    protected $namespace = 'App\Http\Controllers';
    ```

1. <span data-ttu-id="27d5a-124">Сохраните все изменения и перезапустите сервер.</span><span class="sxs-lookup"><span data-stu-id="27d5a-124">Save all of your changes and restart the server.</span></span> <span data-ttu-id="27d5a-125">Теперь приложение должно выглядеть совершенно по-другому.</span><span class="sxs-lookup"><span data-stu-id="27d5a-125">Now, the app should look very different.</span></span>

    ![Снимок экрана с измененной домашней страницей](./images/create-app-01.png)
