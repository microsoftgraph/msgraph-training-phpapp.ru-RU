<!-- markdownlint-disable MD002 MD041 -->

В этом упражнении вы расширим приложение из предыдущего упражнения, чтобы поддерживать проверку подлинности с помощью Azure AD. Это необходимо для получения необходимого маркера доступа OAuth для вызова Microsoft Graph. На этом этапе вы интегрируете клиентскую библиотеку [oauth2](https://github.com/thephpleague/oauth2-client) в приложение.

1. Откройте **ENV-файл** в корневом файле приложения PHP и добавьте следующий код в конец файла.

    :::code language="ini" source="../demo/graph-tutorial/example.env" range="48-54":::

1. Замените код приложения на портале регистрации приложений `YOUR_APP_ID_HERE` и пароль, `YOUR_APP_PASSWORD_HERE` созданный вами.

    > [!IMPORTANT]
    > Если вы используете управление исходным кодом, например git, пришло бы время исключить файл из системы управления источником, чтобы избежать случайной утечки ИД приложения и `.env` пароля.

## <a name="implement-sign-in"></a>Реализация входов

1. Создайте файл с именем **./app/Http/Controllers** и добавьте `AuthController.php` следующий код.

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

    Он определяет контроллер с двумя действиями: `signin` и `callback` .

    Это действие создает URL-адрес для подписи Azure AD, сохраняет значение, сгенерированное клиентом OAuth, а затем перенаправляет браузер на страницу для подписи `signin` `state` Azure AD.

    Это `callback` действие перенаправляет Azure после завершения регистрации. Это действие позволяет убедиться, что значение совпадает с сохраненным значением, а затем пользователи код авторизации, отправленный `state` Azure для запроса маркера доступа. Затем он перенаправляется обратно на домашную страницу с маркером доступа во временном значении ошибки. Вы будете использовать его, чтобы убедиться, что вход работает, прежде чем двигаться дальше.

1. Добавьте маршруты в **./routes/web.php.**

    ```php
    Route::get('/signin', 'AuthController@signin');
    Route::get('/callback', 'AuthController@callback');
    ```

1. Запустите сервер и перейдите к `https://localhost:8000` . Нажмите кнопку "Вход" и перенаправите вас на `https://login.microsoftonline.com` . Войдите с помощью учетной записи Майкрософт.

1. Проверьте запрос согласия. Список разрешений соответствует списку областей разрешений, настроенных в **ENV.**

    - **Поддержив доступ** к данным, к ним предоставлен доступ: ( ) Это разрешение запрашивается MSAL для получения `offline_access` маркеров обновления.
    - **Во sign you in and read your profile:** ( `User.Read` ) This permission allows the app to get the logged-in user's profile and profile photo.
    - **Прочитайте параметры** почтового ящика: ( ) Это разрешение позволяет приложению читать параметры почтового ящика пользователя, включая часовой пояс `MailboxSettings.Read` и формат времени.
    - **Полный доступ к** календарям: ( ) Это разрешение позволяет приложению читать события в календаре пользователя, добавлять новые события и изменять `Calendars.ReadWrite` существующие.

1. Согласие на запрашиваемую разрешения. Браузер перенаправляет пользователя в приложение с отображением маркера.

### <a name="get-user-details"></a>Получить сведения о пользователе

В этом разделе вы обновим метод для получения профиля пользователя `callback` из Microsoft Graph.

1. Добавьте следующие утверждения в верхнюю часть `use` **/app/Http/Controllers/AuthController.php** под `namespace App\Http\Controllers;` строкой.

    ```php
    use Microsoft\Graph\Graph;
    use Microsoft\Graph\Model;
    ```

1. Замените `try` блок в `callback` методе следующим кодом.

    ```php
    try {
      // Make the token request
      $accessToken = $oauthClient->getAccessToken('authorization_code', [
        'code' => $authCode
      ]);

      $graph = new Graph();
      $graph->setAccessToken($accessToken->getToken());

      $user = $graph->createRequest('GET', '/me?$select=displayName,mail,mailboxSettings,userPrincipalName')
        ->setReturnType(Model\User::class)
        ->execute();

      // TEMPORARY FOR TESTING!
      return redirect('/')
        ->with('error', 'Access token received')
        ->with('errorDetail', 'User:'.$user->getDisplayName().', Token:'.$accessToken->getToken());
    }
    ```

Новый код создает объект, назначает маркер доступа, а затем использует его для запроса `Graph` профиля пользователя. Он добавляет отображаемую имя пользователя во временные выходные данные для тестирования.

## <a name="storing-the-tokens"></a>Хранение маркеров

Теперь, когда вы можете получить маркеры, пора реализовать способ их хранения в приложении. Так как это пример приложения, для простоты вы храните их в сеансе. В реальных приложениях используется более надежное решение для безопасного хранения данных, например база данных.

1. Создайте новый каталог в **каталоге ./app** с именем , а затем создайте новый файл с именем и `TokenStore` добавьте следующий `TokenCache.php` код.

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
          'userEmail' => null !== $user->getMail() ? $user->getMail() : $user->getUserPrincipalName(),
          'userTimeZone' => $user->getMailboxSettings()->getTimeZone()
        ]);
      }

      public function clearTokens() {
        session()->forget('accessToken');
        session()->forget('refreshToken');
        session()->forget('tokenExpires');
        session()->forget('userName');
        session()->forget('userEmail');
        session()->forget('userTimeZone');
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

1. Добавьте следующий выписку в верхнюю часть `use` **./app/Http/Controllers/AuthController.php** под `namespace App\Http\Controllers;` строкой.

    ```php
    use App\TokenStore\TokenCache;
    ```

1. Замените `try` блок в существующей `callback` функции следующим:

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="StoreTokensSnippet":::

## <a name="implement-sign-out"></a>Реализация выходов

Перед тестированием этой новой функции добавьте способ выйти из нее.

1. Добавьте в класс следующее `AuthController` действие.

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="SignOutSnippet":::

1. Добавьте это действие в **./routes/web.php.**

    ```php
    Route::get('/signout', 'AuthController@signout');
    ```

1. Перезапустите сервер и войдите в нее. Вы должны вернуться на home-страницу, но пользовательский интерфейс должен измениться, чтобы указать, что вы вписались.

    ![Снимок экрана с домашней страницей после входов](./images/add-aad-auth-01.png)

1. Щелкните аватар пользователя в правом верхнем углу, чтобы получить доступ к ссылке **"Выйти".** При **нажатии** кнопки "Выйти" сеанс сбрасывается и возвращается на домашней странице.

    ![Снимок экрана с выпадающим меню со ссылкой "Выйти"](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a>Обновление маркеров

На этом этапе приложение имеет маркер доступа, который отправляется в `Authorization` заголовке вызовов API. Это маркер, который позволяет приложению получать доступ к Microsoft Graph от имени пользователя.

Однако этот маркер является кратковременной. Срок действия маркера истекает через час после его выпуска. В этом случае маркер обновления становится полезным. Маркер обновления позволяет приложению запрашивать новый маркер доступа, не требуя от пользователя повторного входить. Обновите код управления маркерами, чтобы реализовать обновление маркера.

1. Откройте **./app/TokenStore/TokenCache.php** и добавьте в класс следующую `TokenCache` функцию.

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="UpdateTokensSnippet":::

1. Замените имеющуюся функцию `getAccessToken` указанным ниже кодом.

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="GetAccessTokenSnippet":::

Этот метод сначала проверяет, истек ли срок действия маркера доступа или почти истек. Если это так, то он использует маркер обновления для получения новых маркеров, а затем обновляет кэш и возвращает новый маркер доступа.
