<!-- markdownlint-disable MD002 MD041 -->

Откройте интерфейс командной строки (CLI), перейдите к каталогу, в котором у вас есть права на создание файлов, и выполните следующую команду, чтобы создать новое приложение PHP.

```Shell
laravel new graph-tutorial
```

Ларавел создает новый каталог с именем `graph-tutorial` и формирование шаблонов для приложения PHP. Перейдите к новому каталогу и введите следующую команду для запуска локального веб-сервера.

```Shell
php artisan serve
```

Откройте браузер и перейдите по адресу `http://localhost:8000`. Если все работает, вы увидите страницу Ларавел по умолчанию. Если вы не видите эту страницу, проверьте [документы ларавел](https://laravel.com/docs/5.6).

Прежде чем переходить, установите несколько дополнительных библиотек, которые будут использоваться позже:

- [OAuth2 — клиент](https://github.com/thephpleague/oauth2-client) для обработки потоков маркеров входа и маркеров OAuth.
- [Microsoft](https://github.com/microsoftgraph/msgraph-sdk-php) Graph для совершения звонков в Microsoft Graph.

Выполните следующую команду в командной панели CLI.

```Shell
composer require league/oauth2-client:dev-master microsoft/microsoft-graph
```

## <a name="design-the-app"></a>Проектирование приложения

Начните с создания глобального макета для приложения. Создайте новый файл в `./resources/views` каталоге `layout.blade.php` и добавьте указанный ниже код.

```php
<!DOCTYPE html>
<html>
  <head>
    <title>PHP Graph Tutorial</title>

    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/css/bootstrap.min.css"
        integrity="sha384-WskhaSGFgHYWDcbwN70/dfYBj47jz9qbsMId/iRN3ewGhXQFZCSftd1LZCfmhktB" crossorigin="anonymous">
    <link rel="stylesheet" href="https://use.fontawesome.com/releases/v5.1.0/css/all.css"
        integrity="sha384-lKuwvrZot6UHsBSfcMvOkWwlCMgc0TaWr+30HWe3a4ltaBwTZhyTEggF5tJv8tbt" crossorigin="anonymous">
    <link rel="stylesheet" href="{{ asset('/css/app.css') }}">
    <script src="https://code.jquery.com/jquery-3.3.1.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js"
        integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" crossorigin="anonymous"></script>
    <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.1/js/bootstrap.min.js"
        integrity="sha384-smHYKdLADwkXOn1EmN1qk/HfnUcbVRZyYmZ4qpPea6sjB/pTJ0euyQp0Mk8ck+5T" crossorigin="anonymous"></script>
  </head>

  <body>
    <nav class="navbar navbar-expand-md navbar-dark fixed-top bg-dark">
      <div class="container">
        <a href="/" class="navbar-brand">PHP Graph Tutorial</a>
        <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarCollapse"
            aria-controls="navbarCollapse" aria-expanded="false" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="collapse navbar-collapse" id="navbarCollapse">
          <ul class="navbar-nav mr-auto">
            <li class="nav-item">
              <a href="/" class="nav-link {{$_SERVER['REQUEST_URI'] == '/' ? ' active' : ''}}">Home</a>
            </li>
            @if(isset($userName))
              <li class="nav-item" data-turbolinks="false">
                <a href="/calendar" class="nav-link{{$_SERVER['REQUEST_URI'] == '/calendar' ? ' active' : ''}}">Calendar</a>
              </li>
            @endif
          </ul>
          <ul class="navbar-nav justify-content-end">
            <li class="nav-item">
              <a class="nav-link" href="https://developer.microsoft.com/graph/docs/concepts/overview" target="_blank">
                <i class="fas fa-external-link-alt mr-1"></i>Docs
              </a>
            </li>
            @if(isset($userName))
              <li class="nav-item dropdown">
                <a class="nav-link dropdown-toggle" data-toggle="dropdown" href="#" role="button"
                  aria-haspopup="true" aria-expanded="false">
                  @if(isset($user_avatar))
                    <img src="{{ $user_avatar }}" class="rounded-circle align-self-center mr-2" style="width: 32px;">
                  @else
                    <i class="far fa-user-circle fa-lg rounded-circle align-self-center mr-2" style="width: 32px;"></i>
                  @endif
                </a>
                <div class="dropdown-menu dropdown-menu-right">
                  <h5 class="dropdown-item-text mb-0">{{ $userName }}</h5>
                  <p class="dropdown-item-text text-muted mb-0">{{ $userEmail }}</p>
                  <div class="dropdown-divider"></div>
                  <a href="/signout" class="dropdown-item">Sign Out</a>
                </div>
              </li>
            @else
              <li class="nav-item">
                <a href="/signin" class="nav-link">Sign In</a>
              </li>
            @endif
          </ul>
        </div>
      </div>
    </nav>
    <main role="main" class="container">
      @if(session('error'))
        <div class="alert alert-danger" role="alert">
          <p class="mb-3">{{ session('error') }}</p>
          @if(session('errorDetail'))
            <pre class="alert-pre border bg-light p-2"><code>{{ session('errorDetail') }}</code></pre>
          @endif
        </div>
      @endif

      @yield('content')
    </main>
  </body>
</html>
```

В этом коде [](http://getbootstrap.com/) добавляется начальная загрузка для простых стилей и [Шрифт Awesome](https://fontawesome.com/) для некоторых простых значков. Он также определяет глобальную структуру с помощью панели навигации.

Теперь откройте `./public/css/app.css` и замените все содержимое приведенным ниже.

```css
body {
  padding-top: 4.5rem;
}

.alert-pre {
  word-wrap: break-word;
  word-break: break-all;
  white-space: pre-wrap;
}
```

Теперь обновите страницу по умолчанию. Откройте `./resources/views/welcome.blade.php` файл и замените его содержимое на приведенный ниже код.

```php
@extends('layout')

@section('content')
<div class="jumbotron">
  <h1>PHP Graph Tutorial</h1>
  <p class="lead">This sample app shows how to use the Microsoft Graph API to access Outlook and OneDrive data from PHP</p>
  @if(isset($userName))
    <h4>Welcome {{ $userName }}!</h4>
    <p>Use the navigation bar at the top of the page to get started.</p>
  @else
    <a href="/signin" class="btn btn-primary btn-large">Click here to sign in</a>
  @endif
</div>
@endsection
```

Обновите базовый `Controller` класс `./app/Http/Controllers/Controller.php` , добавив приведенную ниже функцию в класс.

```php
public function loadViewData()
{
  $viewData = [];

  // Check for flash errors
  if (session('error')) {
    $viewData['error'] = session('error');
    $viewData['errorDetail'] = session('errorDetail');
  }

  // Check for logged on user
  if (session('userName'))
  {
    $viewData['userName'] = session('userName');
    $viewData['userEmail'] = session('userEmail');
  }

  return $viewData;
}
```

Затем добавьте контроллер для домашней страницы. Создайте новый файл в `./app/Http/Controllers` каталоге `HomeController.php` и добавьте указанный ниже код.

```php
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class HomeController extends Controller
{
  public function welcome()
  {
    $viewData = $this->loadViewData();

    return view('welcome', $viewData);
  }
}
```

Наконец, обновите маршрут в `./routes/web.php` , чтобы использовать новый контроллер. Замените все содержимое этого файла на приведенный ниже код.

```php
<?php

Route::get('/', 'HomeController@welcome');
```

Сохраните все изменения и перезапустите сервер. Теперь приложение должно выглядеть по-другому.

![Снимок экрана с переработанной домашней страницей](./images/create-app-01.png)