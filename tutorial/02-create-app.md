<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="1b772-101">Начните с создания нового проекта Ларавел.</span><span class="sxs-lookup"><span data-stu-id="1b772-101">Begin by creating a new Laravel project.</span></span>

1. <span data-ttu-id="1b772-102">Откройте интерфейс командной строки (CLI), перейдите к каталогу, в котором у вас есть права на создание файлов, и выполните следующую команду, чтобы создать новое приложение PHP.</span><span class="sxs-lookup"><span data-stu-id="1b772-102">Open your command-line interface (CLI), navigate to a directory where you have rights to create files, and run the following command to create a new PHP app.</span></span>

    ```Shell
    laravel new graph-tutorial
    ```

1. <span data-ttu-id="1b772-103">Перейдите к каталогу **Graph-Tutorial** и введите следующую команду для запуска локального веб-сервера.</span><span class="sxs-lookup"><span data-stu-id="1b772-103">Navigate to the **graph-tutorial** directory and enter the following command to start a local web server.</span></span>

    ```Shell
    php artisan serve
    ```

1. <span data-ttu-id="1b772-104">Откройте браузер и перейдите по адресу `http://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="1b772-104">Open your browser and navigate to `http://localhost:8000`.</span></span> <span data-ttu-id="1b772-105">Если все работает, вы увидите страницу Ларавел по умолчанию.</span><span class="sxs-lookup"><span data-stu-id="1b772-105">If everything is working, you will see a default Laravel page.</span></span> <span data-ttu-id="1b772-106">Если вы не видите эту страницу, проверьте [документы ларавел](https://laravel.com/docs/7.x).</span><span class="sxs-lookup"><span data-stu-id="1b772-106">If you don't see that page, check the [Laravel docs](https://laravel.com/docs/7.x).</span></span>

## <a name="install-packages"></a><span data-ttu-id="1b772-107">Установка пакетов</span><span class="sxs-lookup"><span data-stu-id="1b772-107">Install packages</span></span>

<span data-ttu-id="1b772-108">Прежде чем переходить, установите несколько дополнительных пакетов, которые будут использоваться позже:</span><span class="sxs-lookup"><span data-stu-id="1b772-108">Before moving on, install some additional packages that you will use later:</span></span>

- <span data-ttu-id="1b772-109">[OAuth2 — клиент](https://github.com/thephpleague/oauth2-client) для обработки потоков маркеров входа и маркеров OAuth.</span><span class="sxs-lookup"><span data-stu-id="1b772-109">[oauth2-client](https://github.com/thephpleague/oauth2-client) for handling sign-in and OAuth token flows.</span></span>
- <span data-ttu-id="1b772-110">[Microsoft](https://github.com/microsoftgraph/msgraph-sdk-php) Graph для совершения звонков в Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="1b772-110">[microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) for making calls to Microsoft Graph.</span></span>

1. <span data-ttu-id="1b772-111">Выполните следующую команду, чтобы удалить существующую версию `guzzlehttp/guzzle` .</span><span class="sxs-lookup"><span data-stu-id="1b772-111">Run the following command to remove the existing version of `guzzlehttp/guzzle`.</span></span> <span data-ttu-id="1b772-112">Версия, установленная с помощью Ларавел, вступает в противоречие с версией, необходимой пакету SDK для Microsoft Graph PHP.</span><span class="sxs-lookup"><span data-stu-id="1b772-112">The version installed by Laravel conflicts with the version required by the Microsoft Graph PHP SDK.</span></span>

    ```Shell
    composer remove guzzlehttp/guzzle
    ```

1. <span data-ttu-id="1b772-113">Выполните следующую команду в командной панели CLI.</span><span class="sxs-lookup"><span data-stu-id="1b772-113">Run the following command in your CLI.</span></span>

    ```Shell
    composer require league/oauth2-client microsoft/microsoft-graph
    ```

## <a name="design-the-app"></a><span data-ttu-id="1b772-114">Проектирование приложения</span><span class="sxs-lookup"><span data-stu-id="1b772-114">Design the app</span></span>

1. <span data-ttu-id="1b772-115">Создайте новый файл в каталоге **./ресаурцес/виевс** `layout.blade.php` и добавьте указанный ниже код.</span><span class="sxs-lookup"><span data-stu-id="1b772-115">Create a new file in the **./resources/views** directory named `layout.blade.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/layout.blade.php" id="LayoutSnippet":::

    <span data-ttu-id="1b772-116">В этом коде добавляется [Начальная](http://getbootstrap.com/) загрузка для простых стилей и [Шрифт Awesome](https://fontawesome.com/) для некоторых простых значков.</span><span class="sxs-lookup"><span data-stu-id="1b772-116">This code adds [Bootstrap](http://getbootstrap.com/) for simple styling, and [Font Awesome](https://fontawesome.com/) for some simple icons.</span></span> <span data-ttu-id="1b772-117">Он также определяет глобальную структуру с помощью панели навигации.</span><span class="sxs-lookup"><span data-stu-id="1b772-117">It also defines a global layout with a nav bar.</span></span>

1. <span data-ttu-id="1b772-118">Создайте новый каталог в `./public` каталоге с именем `css` , а затем создайте новый файл в `./public/css` каталоге с именем `app.css` .</span><span class="sxs-lookup"><span data-stu-id="1b772-118">Create a new directory in the `./public` directory named `css`, then create a new file in the `./public/css` directory named `app.css`.</span></span> <span data-ttu-id="1b772-119">Добавьте в него указанный ниже код.</span><span class="sxs-lookup"><span data-stu-id="1b772-119">Add the following code.</span></span>

    :::code language="css" source="../demo/graph-tutorial/public/css/app.css":::

1. <span data-ttu-id="1b772-120">Откройте `./resources/views/welcome.blade.php` файл и замените его содержимое на приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="1b772-120">Open the `./resources/views/welcome.blade.php` file and replace its contents with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/resources/views/welcome.blade.php" id="WelcomeSnippet":::

1. <span data-ttu-id="1b772-121">Обновите базовый `Controller` класс в **/АПП/хттп/контроллерс/контроллер.ФП** , добавив приведенную ниже функцию в класс.</span><span class="sxs-lookup"><span data-stu-id="1b772-121">Update the base `Controller` class in **./app/Http/Controllers/Controller.php** by adding the following function to the class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/Controller.php" id="LoadViewDataSnippet":::

1. <span data-ttu-id="1b772-122">Создайте новый файл в `./app/Http/Controllers` каталоге `HomeController.php` и добавьте указанный ниже код.</span><span class="sxs-lookup"><span data-stu-id="1b772-122">Create a new file in the `./app/Http/Controllers` directory named `HomeController.php` and add the following code.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/HomeController.php":::

1. <span data-ttu-id="1b772-123">Обновите маршрут, `./routes/web.php` чтобы использовать новый контроллер.</span><span class="sxs-lookup"><span data-stu-id="1b772-123">Update the route in `./routes/web.php` to use the new controller.</span></span> <span data-ttu-id="1b772-124">Замените все содержимое этого файла на приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="1b772-124">Replace the entire contents of this file with the following.</span></span>

    ```php
    <?php

    use Illuminate\Support\Facades\Route;

    Route::get('/', 'HomeController@welcome');
    ```

1. <span data-ttu-id="1b772-125">Сохраните все изменения и перезапустите сервер.</span><span class="sxs-lookup"><span data-stu-id="1b772-125">Save all of your changes and restart the server.</span></span> <span data-ttu-id="1b772-126">Теперь приложение должно выглядеть по-другому.</span><span class="sxs-lookup"><span data-stu-id="1b772-126">Now, the app should look very different.</span></span>

    ![Снимок экрана с переработанной домашней страницей](./images/create-app-01.png)
