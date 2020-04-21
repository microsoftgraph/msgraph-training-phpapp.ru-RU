<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы будете расширяем приложение из предыдущего упражнения для поддержки проверки подлинности с помощью Azure AD. Это необходимо для получения необходимого маркера доступа OAuth для вызова Microsoft Graph. На этом этапе в приложение будет интегрирована библиотека [OAuth2 – Client](https://github.com/thephpleague/oauth2-client) .

1. Откройте `.env` файл в корневом каталоге приложения PHP и добавьте следующий код в конец файла.

    :::code language="ini" source="../demo/graph-tutorial/.env.example" id="OAuthSettingsSnippet":::

1. Замените `YOUR_APP_ID_HERE` идентификатором приложения на портале регистрации приложений и замените `YOUR_APP_PASSWORD_HERE` созданным паролем.

    > [!IMPORTANT]
    > Если вы используете систему управления версиями (например, Git), то в дальнейшем будет полезно исключить `.env` файл из системы управления версиями, чтобы избежать непреднамеренного утечки идентификатора и пароля приложения.

## <a name="implement-sign-in"></a>Реализация входа

1. Создайте новый файл в каталоге **./АПП/хттп/контроллерс** `AuthController.php` и добавьте указанный ниже код.

    ```php
    <?php

    namespace App\Http\Controllers;

    use App\Http\Controllers\Controller;
    use Illuminate\Http\Request;

    class AuthController extends Controller
    {
      public function signin()
      {
        // Initialize the OAuth client
        $oauthClient = new \League\OAuth2\Client\Provider\GenericProvider([
          'clientId'                => env('OAUTH_APP_ID'),
          'clientSecret'            => env('OAUTH_APP_PASSWORD'),
          'redirectUri'             => env('OAUTH_REDIRECT_URI'),
          'urlAuthorize'            => env('OAUTH_AUTHORITY').env('OAUTH_AUTHORIZE_ENDPOINT'),
          'urlAccessToken'          => env('OAUTH_AUTHORITY').env('OAUTH_TOKEN_ENDPOINT'),
          'urlResourceOwnerDetails' => '',
          'scopes'                  => env('OAUTH_SCOPES')
        ]);

        $authUrl = $oauthClient->getAuthorizationUrl();

        // Save client state so we can validate in callback
        session(['oauthState' => $oauthClient->getState()]);

        // Redirect to AAD signin page
        return redirect()->away($authUrl);
      }

      public function callback(Request $request)
      {
        // Validate state
        $expectedState = session('oauthState');
        $request->session()->forget('oauthState');
        $providedState = $request->query('state');

        if (!isset($expectedState)) {
          // If there is no expected state in the session,
          // do nothing and redirect to the home page.
          return redirect('/');
        }

        if (!isset($providedState) || $expectedState != $providedState) {
          return redirect('/')
            ->with('error', 'Invalid auth state')
            ->with('errorDetail', 'The provided auth state did not match the expected value');
        }

        // Authorization code should be in the "code" query param
        $authCode = $request->query('code');
        if (isset($authCode)) {
          // Initialize the OAuth client
          $oauthClient = new \League\OAuth2\Client\Provider\GenericProvider([
            'clientId'                => env('OAUTH_APP_ID'),
            'clientSecret'            => env('OAUTH_APP_PASSWORD'),
            'redirectUri'             => env('OAUTH_REDIRECT_URI'),
            'urlAuthorize'            => env('OAUTH_AUTHORITY').env('OAUTH_AUTHORIZE_ENDPOINT'),
            'urlAccessToken'          => env('OAUTH_AUTHORITY').env('OAUTH_TOKEN_ENDPOINT'),
            'urlResourceOwnerDetails' => '',
            'scopes'                  => env('OAUTH_SCOPES')
          ]);

          try {
            // Make the token request
            $accessToken = $oauthClient->getAccessToken('authorization_code', [
              'code' => $authCode
            ]);

            // TEMPORARY FOR TESTING!
            return redirect('/')
              ->with('error', 'Access token received')
              ->with('errorDetail', $accessToken->getToken());
          }
          catch (League\OAuth2\Client\Provider\Exception\IdentityProviderException $e) {
            return redirect('/')
              ->with('error', 'Error requesting access token')
              ->with('errorDetail', $e->getMessage());
          }
        }

        return redirect('/')
          ->with('error', $request->query('error'))
          ->with('errorDetail', $request->query('error_description'));
      }
    }
    ```

    Этот параметр определяет контроллер с двумя действиями `signin` : `callback`и.

    `signin` Действие создает URL-адрес входа Azure AD, сохраняет `state` значение, созданное клиентом OAuth, а затем перенаправляет браузер на страницу входа Azure AD.

    `callback` Действие перенаправляет Azure после завершения входа. Это действие гарантирует, что `state` значение соответствует сохраненному значению, а затем пользователю код авторизации, отправленный Azure, запрашивает маркер доступа. Затем он перенаправляется обратно на домашнюю страницу с маркером доступа во временном значении ошибки. Вы будете использовать этот параметр, чтобы проверить, работает ли вход, прежде чем переходить к.

1. Добавьте маршруты в **./раутес/веб.ФП**.

    ```php
    Route::get('/signin', 'AuthController@signin');
    Route::get('/callback', 'AuthController@callback');
    ```

1. Запустите сервер и перейдите к `https://localhost:8000`. Нажмите кнопку входа, и вы будете перенаправлены на `https://login.microsoftonline.com`. Войдите с помощью учетной записи Майкрософт и согласия с запрошенными разрешениями. Браузер перенаправляется на приложение, отображая маркер.

### <a name="get-user-details"></a>Получение сведений о пользователе

В этом разделе описывается обновление `callback` метода для получения профиля пользователя из Microsoft Graph.

1. Добавьте следующие `use` операторы в верхнюю часть **/АПП/хттп/контроллерс/аусконтроллер.ФП**, расположенную под `namespace App\Http\Controllers;` строкой.

    ```php
    use Microsoft\Graph\Graph;
    use Microsoft\Graph\Model;
    ```

1. Замените `try` блок в `callback` методе на приведенный ниже код.

    ```php
    try {
      // Make the token request
      $accessToken = $oauthClient->getAccessToken('authorization_code', [
        'code' => $authCode
      ]);

      $graph = new Graph();
      $graph->setAccessToken($accessToken->getToken());

      $user = $graph->createRequest('GET', '/me')
        ->setReturnType(Model\User::class)
        ->execute();

      // TEMPORARY FOR TESTING!
      return redirect('/')
        ->with('error', 'Access token received')
        ->with('errorDetail', 'User:'.$user->getDisplayName().', Token:'.$accessToken->getToken());
    }
    ```

Новый код создает `Graph` объект, назначает маркер доступа, а затем использует его для запроса профиля пользователя. Он добавляет отображаемое имя пользователя во временный вывод для тестирования.

## <a name="storing-the-tokens"></a>Сохранение маркеров

Теперь, когда вы можете получить маркеры, следует реализовать способ их хранения в приложении. Так как это пример приложения, для простоты вы будете хранить их в сеансе. Реальное приложение использует более надежное решение для безопасного хранения, например базу данных.

1. Создайте каталог в каталоге **./АПП** с именем `TokenStore`, затем создайте в этом каталоге новый файл с именем `TokenCache.php`и добавьте приведенный ниже код.

    ```php
    <?php

    namespace App\TokenStore;

    class TokenCache {
      public function storeTokens($accessToken, $user) {
        session([
          'accessToken' => $accessToken->getToken(),
          'refreshToken' => $accessToken->getRefreshToken(),
          'tokenExpires' => $accessToken->getExpires(),
          'userName' => $user->getDisplayName(),
          'userEmail' => null !== $user->getMail() ? $user->getMail() : $user->getUserPrincipalName()
        ]);
      }

      public function clearTokens() {
        session()->forget('accessToken');
        session()->forget('refreshToken');
        session()->forget('tokenExpires');
        session()->forget('userName');
        session()->forget('userEmail');
      }

      public function getAccessToken() {
        // Check if tokens exist
        if (empty(session('accessToken')) ||
            empty(session('refreshToken')) ||
            empty(session('tokenExpires'))) {
          return '';
        }

        return session('accessToken');
      }
    }
    ```

1. Добавьте следующий `use` оператор в верхнюю часть файла **./АПП/хттп/контроллерс/аусконтроллер.ФП**, расположенную под `namespace App\Http\Controllers;` строкой.

    ```php
    use App\TokenStore\TokenCache;
    ```

1. Замените `try` блок в существующей `callback` функции приведенным ниже блоком.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="StoreTokensSnippet":::

## <a name="implement-sign-out"></a>Реализация выхода

Перед тестированием новой функции добавьте способ выхода.

1. Добавьте в `AuthController` класс следующее действие.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="SignOutSnippet":::

1. Добавьте это действие в **./раутес/веб.ФП**.

    ```php
    Route::get('/signout', 'AuthController@signout');
    ```

1. Перезапустите сервер и пройдите процесс входа. Необходимо вернуться на домашнюю страницу, но пользовательский интерфейс должен измениться, чтобы показать, что вы вошли в систему.

    ![Снимок экрана домашней страницы после входа](./images/add-aad-auth-01.png)

1. Щелкните аватар пользователя в правом верхнем углу, чтобы получить доступ к ссылке **выхода** . При нажатии кнопки **выйти** сбрасывается сеанс и возвращается на домашнюю страницу.

    ![Снимок экрана с раскрывающимся меню со ссылкой "выйти"](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Обновление маркеров

На этом шаге приложение имеет маркер доступа, который отправляется в `Authorization` заголовке вызовов API. Это маркер, который позволяет приложению получать доступ к Microsoft Graph от имени пользователя.

Однако этот маркер кратковременно используется. Срок действия маркера истечет через час после его выдачи. В этом случае маркер обновления становится полезен. Маркер обновления позволяет приложению запросить новый маркер доступа, не требуя от пользователя повторного входа. Обновите код управления маркером, чтобы реализовать обновление маркеров.

1. Откройте **./АПП/токенсторе/токенкаче.ФП** и добавьте в `TokenCache` класс следующую функцию.

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="UpdateTokensSnippet":::

1. Замените имеющуюся функцию `getAccessToken` указанным ниже кодом.

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="GetAccessTokenSnippet":::

Этот метод сначала проверяет, истек ли срок действия маркера доступа, или закройте его до истечения срока действия. Если это так, то он использует маркер обновления для получения новых токенов, затем обновляет кэш и возвращает новый маркер доступа.
