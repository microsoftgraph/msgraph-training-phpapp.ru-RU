<!-- markdownlint-disable MD002 MD041 -->

Начните с создания нового проекта Laravel.

1. Откройте интерфейс командной строки (CLI), перейдите в каталог, в котором у вас есть права на создание файлов, и запустите следующую команду, чтобы создать приложение PHP.

    ```Shell
    laravel new graph-tutorial
    ```

1. Перейдите в **каталог учебника** по графику и введите следующую команду, чтобы запустить локальный веб-сервер.

    ```Shell
    php artisan serve
    ```

1. Откройте браузер и перейдите по адресу `http://localhost:8000`. Если все работает, вы увидите страницу Laravel по умолчанию. Если вы не видите эту страницу, проверьте [документы Laravel.](https://laravel.com/docs/8.x)

## <a name="install-packages"></a>Установка пакетов

Прежде чем двигаться дальше, установите некоторые дополнительные пакеты, которые вы будете использовать позже:

- [oauth2-client](https://github.com/thephpleague/oauth2-client) для обработки потоков входов и маркеров OAuth.
- [Microsoft-graph](https://github.com/microsoftgraph/msgraph-sdk-php) для звонков в Microsoft Graph.

1. Запустите следующую команду в CLI.

    ```Shell
    composer require league/oauth2-client microsoft/microsoft-graph
    ```

## <a name="design-the-app"></a>Проектирование приложения

1. Создайте файл с именем **./resources/views** и `layout.blade.php` добавьте следующий код.

    :::code language="php" source="../demo/graph-tutorial/resources/views/layout.blade.php" id="LayoutSnippet":::

    В этом коде [добавляется Bootstrap](http://getbootstrap.com/) для простого стиля, а для некоторых простых значков [font : "Отличный".](https://fontawesome.com/) Он также определяет глобальный макет с панели nav.

1. Создайте новый каталог в каталоге с именем , а затем создайте новый файл `./public` `css` в `./public/css` каталоге с именем `app.css` . Добавьте в него указанный ниже код.

    :::code language="css" source="../demo/graph-tutorial/public/css/app.css":::

1. Откройте файл `./resources/views/welcome.blade.php` и замените его содержимое на следующее.

    :::code language="php" source="../demo/graph-tutorial/resources/views/welcome.blade.php" id="WelcomeSnippet":::

1. Обновив `Controller` базовый класс **в ./app/Http/Controllers/Controller.php,** добавив в класс следующую функцию.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/Controller.php" id="LoadViewDataSnippet":::

1. Создайте новый файл в `./app/Http/Controllers` каталоге с именем `HomeController.php` и добавьте следующий код.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/HomeController.php":::

1. Обновим маршрут `./routes/web.php` для использования нового контроллера. Замените все содержимое этого файла на следующее.

    ```php
    <?php

    use Illuminate\Support\Facades\Route;

    Route::get('/', 'HomeController@welcome');
    ```

1. Откройте **./app/Providers/RouteServiceProvider.php** и раскомментирование `$namespace` объявления.

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

1. Сохраните все изменения и перезапустите сервер. Теперь приложение должно выглядеть совершенно по-другому.

    ![Снимок экрана с измененной домашней страницей](./images/create-app-01.png)
