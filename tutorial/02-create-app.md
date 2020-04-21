<!-- markdownlint-disable MD002 MD041 -->

Начните с создания нового проекта Ларавел.

1. Откройте интерфейс командной строки (CLI), перейдите к каталогу, в котором у вас есть права на создание файлов, и выполните следующую команду, чтобы создать новое приложение PHP.

    ```Shell
    laravel new graph-tutorial
    ```

1. Перейдите к каталогу **Graph-Tutorial** и введите следующую команду для запуска локального веб-сервера.

    ```Shell
    php artisan serve
    ```

1. Откройте браузер и перейдите по адресу `http://localhost:8000`. Если все работает, вы увидите страницу Ларавел по умолчанию. Если вы не видите эту страницу, проверьте [документы ларавел](https://laravel.com/docs/7.x).

## <a name="install-packages"></a>Установка пакетов

Прежде чем переходить, установите несколько дополнительных пакетов, которые будут использоваться позже:

- [OAuth2 — клиент](https://github.com/thephpleague/oauth2-client) для обработки потоков маркеров входа и маркеров OAuth.
- [Microsoft](https://github.com/microsoftgraph/msgraph-sdk-php) Graph для совершения звонков в Microsoft Graph.

1. Выполните следующую команду в командной панели CLI.

    ```Shell
    composer require league/oauth2-client microsoft/microsoft-graph
    ```

## <a name="design-the-app"></a>Проектирование приложения

1. Создайте новый файл в каталоге **./ресаурцес/виевс** `layout.blade.php` и добавьте указанный ниже код.

    :::code language="php" source="../demo/graph-tutorial/resources/views/layout.blade.php" id="LayoutSnippet":::

    В этом коде добавляется [Начальная](http://getbootstrap.com/) загрузка для простых стилей и [Шрифт Awesome](https://fontawesome.com/) для некоторых простых значков. Он также определяет глобальную структуру с помощью панели навигации.

1. Создайте `./public` новый каталог в каталоге с именем `css`, а затем создайте новый файл в `./public/css` каталоге с именем. `app.css` Добавьте в него указанный ниже код.

    :::code language="css" source="../demo/graph-tutorial/public/css/app.css":::

1. Откройте `./resources/views/welcome.blade.php` файл и замените его содержимое на приведенный ниже код.

    :::code language="php" source="../demo/graph-tutorial/resources/views/welcome.blade.php" id="WelcomeSnippet":::

1. Обновите базовый `Controller` класс в **/АПП/хттп/контроллерс/контроллер.ФП** , добавив приведенную ниже функцию в класс.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/Controller.php" id="LoadViewDataSnippet":::

1. Создайте новый файл в `./app/Http/Controllers` каталоге `HomeController.php` и добавьте указанный ниже код.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/HomeController.php":::

1. Обновите маршрут, `./routes/web.php` чтобы использовать новый контроллер. Замените все содержимое этого файла на приведенный ниже код.

    ```php
    <?php

    use Illuminate\Support\Facades\Route;

    Route::get('/', 'HomeController@welcome');
    ```

1. Сохраните все изменения и перезапустите сервер. Теперь приложение должно выглядеть по-другому.

    ![Снимок экрана с переработанной домашней страницей](./images/create-app-01.png)
