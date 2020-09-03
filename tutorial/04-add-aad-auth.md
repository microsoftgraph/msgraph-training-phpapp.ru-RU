<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="172c8-101">В этом упражнении вы будете расширяем приложение из предыдущего упражнения для поддержки проверки подлинности с помощью Azure AD.</span><span class="sxs-lookup"><span data-stu-id="172c8-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="172c8-102">Это необходимо для получения необходимого маркера доступа OAuth для вызова Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="172c8-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="172c8-103">На этом этапе в приложение будет интегрирована библиотека [OAuth2 – Client](https://github.com/thephpleague/oauth2-client) .</span><span class="sxs-lookup"><span data-stu-id="172c8-103">In this step you will integrate the [oauth2-client](https://github.com/thephpleague/oauth2-client) library into the application.</span></span>

1. <span data-ttu-id="172c8-104">Откройте **env** файл в корневом каталоге приложения PHP и добавьте следующий код в конец файла.</span><span class="sxs-lookup"><span data-stu-id="172c8-104">Open the **.env** file in the root of your PHP application, and add the following code to the end of the file.</span></span>

    :::code language="ini" source="../demo/graph-tutorial/.env.example" id="OAuthSettingsSnippet":::

1. <span data-ttu-id="172c8-105">Замените `YOUR_APP_ID_HERE` идентификатором приложения на портале регистрации приложений и замените `YOUR_APP_PASSWORD_HERE` созданным паролем.</span><span class="sxs-lookup"><span data-stu-id="172c8-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_PASSWORD_HERE` with the password you generated.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="172c8-106">Если вы используете систему управления версиями (например, Git), то в дальнейшем будет полезно исключить `.env` файл из системы управления версиями, чтобы избежать непреднамеренного утечки идентификатора и пароля приложения.</span><span class="sxs-lookup"><span data-stu-id="172c8-106">If you're using source control such as git, now would be a good time to exclude the `.env` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="172c8-107">Реализация входа</span><span class="sxs-lookup"><span data-stu-id="172c8-107">Implement sign-in</span></span>

1. <span data-ttu-id="172c8-108">Создайте новый файл в каталоге **./АПП/хттп/контроллерс** `AuthController.php` и добавьте указанный ниже код.</span><span class="sxs-lookup"><span data-stu-id="172c8-108">Create a new file in the **./app/Http/Controllers** directory named `AuthController.php` and add the following code.</span></span>

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

    <span data-ttu-id="172c8-109">Этот параметр определяет контроллер с двумя действиями: `signin` и `callback` .</span><span class="sxs-lookup"><span data-stu-id="172c8-109">This defines a controller with two actions: `signin` and `callback`.</span></span>

    <span data-ttu-id="172c8-110">`signin`Действие создает URL-адрес входа Azure AD, сохраняет `state` значение, созданное клиентом OAuth, а затем перенаправляет браузер на страницу входа Azure AD.</span><span class="sxs-lookup"><span data-stu-id="172c8-110">The `signin` action generates the Azure AD signin URL, saves the `state` value generated by the OAuth client, then redirects the browser to the Azure AD signin page.</span></span>

    <span data-ttu-id="172c8-111">`callback`Действие перенаправляет Azure после завершения входа.</span><span class="sxs-lookup"><span data-stu-id="172c8-111">The `callback` action is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="172c8-112">Это действие гарантирует, что `state` значение соответствует сохраненному значению, а затем пользователю код авторизации, отправленный Azure, запрашивает маркер доступа.</span><span class="sxs-lookup"><span data-stu-id="172c8-112">That action makes sure the `state` value matches the saved value, then users the authorization code sent by Azure to request an access token.</span></span> <span data-ttu-id="172c8-113">Затем он перенаправляется обратно на домашнюю страницу с маркером доступа во временном значении ошибки.</span><span class="sxs-lookup"><span data-stu-id="172c8-113">It then redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="172c8-114">Вы будете использовать этот параметр, чтобы проверить, работает ли вход, прежде чем переходить к.</span><span class="sxs-lookup"><span data-stu-id="172c8-114">You'll use this to verify that sign-in is working before moving on.</span></span>

1. <span data-ttu-id="172c8-115">Добавьте маршруты в **./раутес/веб.ФП**.</span><span class="sxs-lookup"><span data-stu-id="172c8-115">Add the routes to **./routes/web.php**.</span></span>

    ```php
    Route::get('/signin', 'AuthController@signin');
    Route::get('/callback', 'AuthController@callback');
    ```

1. <span data-ttu-id="172c8-116">Запустите сервер и перейдите к `https://localhost:8000` .</span><span class="sxs-lookup"><span data-stu-id="172c8-116">Start the server and browse to `https://localhost:8000`.</span></span> <span data-ttu-id="172c8-117">Нажмите кнопку входа, и вы будете перенаправлены на `https://login.microsoftonline.com` .</span><span class="sxs-lookup"><span data-stu-id="172c8-117">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="172c8-118">Войдите с помощью учетной записи Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="172c8-118">Login with your Microsoft account.</span></span>

1. <span data-ttu-id="172c8-119">Проверьте запрос согласия.</span><span class="sxs-lookup"><span data-stu-id="172c8-119">Examine the consent prompt.</span></span> <span data-ttu-id="172c8-120">Список разрешений соответствует списку областей разрешений, настроенных в файле **env**.</span><span class="sxs-lookup"><span data-stu-id="172c8-120">The list of permissions correspond to list of permissions scopes configured in **.env**.</span></span>

    - <span data-ttu-id="172c8-121">**Поддержка доступа к данным, доступ к которым предоставлен:** ( `offline_access` ) это разрешение запрашивается с помощью MSAL для получения маркеров обновления.</span><span class="sxs-lookup"><span data-stu-id="172c8-121">**Maintain access to data you have given it access to:** (`offline_access`) This permission is requested by MSAL in order to retrieve refresh tokens.</span></span>
    - <span data-ttu-id="172c8-122">**Вход и чтение профиля:** ( `User.Read` ) это разрешение позволяет приложению получать профиль пользователя, вошедшего в систему, и фотографию профиля.</span><span class="sxs-lookup"><span data-stu-id="172c8-122">**Sign you in and read your profile:** (`User.Read`) This permission allows the app to get the logged-in user's profile and profile photo.</span></span>
    - <span data-ttu-id="172c8-123">**Считывание параметров почтового ящика:** ( `MailboxSettings.Read` ) это разрешение позволяет приложению считывать параметры почтового ящика пользователя, в том числе часовой пояс и формат времени.</span><span class="sxs-lookup"><span data-stu-id="172c8-123">**Read your mailbox settings:** (`MailboxSettings.Read`) This permission allows the app to read the user's mailbox settings, including time zone and time format.</span></span>
    - <span data-ttu-id="172c8-124">**Полный доступ к Вашим календарям:** ( `Calendars.ReadWrite` ) это разрешение позволяет приложению считывать события в календаре пользователя, добавлять новые события и изменять существующие.</span><span class="sxs-lookup"><span data-stu-id="172c8-124">**Have full access to your calendars:** (`Calendars.ReadWrite`) This permission allows the app to read events on the user's calendar, add new events, and modify existing ones.</span></span>

1. <span data-ttu-id="172c8-125">Согласие на запрошенные разрешения.</span><span class="sxs-lookup"><span data-stu-id="172c8-125">Consent to the requested permissions.</span></span> <span data-ttu-id="172c8-126">Браузер перенаправляется на приложение, отображая маркер.</span><span class="sxs-lookup"><span data-stu-id="172c8-126">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="172c8-127">Получение сведений о пользователе</span><span class="sxs-lookup"><span data-stu-id="172c8-127">Get user details</span></span>

<span data-ttu-id="172c8-128">В этом разделе описывается обновление `callback` метода для получения профиля пользователя из Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="172c8-128">In this section you'll update the `callback` method to get the user's profile from Microsoft Graph.</span></span>

1. <span data-ttu-id="172c8-129">Добавьте следующие `use` операторы в верхнюю часть **/АПП/хттп/контроллерс/аусконтроллер.ФП**, расположенную под `namespace App\Http\Controllers;` строкой.</span><span class="sxs-lookup"><span data-stu-id="172c8-129">Add the following `use` statements to the top of **/app/Http/Controllers/AuthController.php**, beneath the `namespace App\Http\Controllers;` line.</span></span>

    ```php
    use Microsoft\Graph\Graph;
    use Microsoft\Graph\Model;
    ```

1. <span data-ttu-id="172c8-130">Замените `try` блок в `callback` методе на приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="172c8-130">Replace the `try` block in the `callback` method with the following code.</span></span>

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

<span data-ttu-id="172c8-131">Новый код создает `Graph` объект, назначает маркер доступа, а затем использует его для запроса профиля пользователя.</span><span class="sxs-lookup"><span data-stu-id="172c8-131">The new code creates a `Graph` object, assigns the access token, then uses it to request the user's profile.</span></span> <span data-ttu-id="172c8-132">Он добавляет отображаемое имя пользователя во временный вывод для тестирования.</span><span class="sxs-lookup"><span data-stu-id="172c8-132">It adds the user's display name to the temporary output for testing.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="172c8-133">Сохранение маркеров</span><span class="sxs-lookup"><span data-stu-id="172c8-133">Storing the tokens</span></span>

<span data-ttu-id="172c8-134">Теперь, когда вы можете получить маркеры, следует реализовать способ их хранения в приложении.</span><span class="sxs-lookup"><span data-stu-id="172c8-134">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="172c8-135">Так как это пример приложения, для простоты вы будете хранить их в сеансе.</span><span class="sxs-lookup"><span data-stu-id="172c8-135">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="172c8-136">Реальное приложение использует более надежное решение для безопасного хранения, например базу данных.</span><span class="sxs-lookup"><span data-stu-id="172c8-136">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

1. <span data-ttu-id="172c8-137">Создайте каталог в каталоге **./АПП** с именем `TokenStore` , затем создайте в этом каталоге новый файл с именем `TokenCache.php` и добавьте приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="172c8-137">Create a new directory in the **./app** directory named `TokenStore`, then create a new file in that directory named `TokenCache.php`, and add the following code.</span></span>

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

1. <span data-ttu-id="172c8-138">Добавьте следующий `use` оператор в верхнюю часть файла **./АПП/хттп/контроллерс/аусконтроллер.ФП**, расположенную под `namespace App\Http\Controllers;` строкой.</span><span class="sxs-lookup"><span data-stu-id="172c8-138">Add the following `use` statement to the top of **./app/Http/Controllers/AuthController.php**, beneath the `namespace App\Http\Controllers;` line.</span></span>

    ```php
    use App\TokenStore\TokenCache;
    ```

1. <span data-ttu-id="172c8-139">Замените `try` блок в существующей `callback` функции приведенным ниже блоком.</span><span class="sxs-lookup"><span data-stu-id="172c8-139">Replace the `try` block in the existing `callback` function with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="StoreTokensSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="172c8-140">Реализация выхода</span><span class="sxs-lookup"><span data-stu-id="172c8-140">Implement sign-out</span></span>

<span data-ttu-id="172c8-141">Перед тестированием новой функции добавьте способ выхода.</span><span class="sxs-lookup"><span data-stu-id="172c8-141">Before you test this new feature, add a way to sign out.</span></span>

1. <span data-ttu-id="172c8-142">Добавьте в класс следующее действие `AuthController` .</span><span class="sxs-lookup"><span data-stu-id="172c8-142">Add the following action to the `AuthController` class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="SignOutSnippet":::

1. <span data-ttu-id="172c8-143">Добавьте это действие в **./раутес/веб.ФП**.</span><span class="sxs-lookup"><span data-stu-id="172c8-143">Add this action to **./routes/web.php**.</span></span>

    ```php
    Route::get('/signout', 'AuthController@signout');
    ```

1. <span data-ttu-id="172c8-144">Перезапустите сервер и пройдите процесс входа.</span><span class="sxs-lookup"><span data-stu-id="172c8-144">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="172c8-145">Необходимо вернуться на домашнюю страницу, но пользовательский интерфейс должен измениться, чтобы показать, что вы вошли в систему.</span><span class="sxs-lookup"><span data-stu-id="172c8-145">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Снимок экрана домашней страницы после входа](./images/add-aad-auth-01.png)

1. <span data-ttu-id="172c8-147">Щелкните аватар пользователя в правом верхнем углу, чтобы получить доступ к ссылке **выхода** .</span><span class="sxs-lookup"><span data-stu-id="172c8-147">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="172c8-148">При нажатии кнопки **выйти** сбрасывается сеанс и возвращается на домашнюю страницу.</span><span class="sxs-lookup"><span data-stu-id="172c8-148">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Снимок экрана с раскрывающимся меню со ссылкой "выйти"](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="172c8-150">Обновление маркеров</span><span class="sxs-lookup"><span data-stu-id="172c8-150">Refreshing tokens</span></span>

<span data-ttu-id="172c8-151">На этом шаге приложение имеет маркер доступа, который отправляется в `Authorization` заголовке вызовов API.</span><span class="sxs-lookup"><span data-stu-id="172c8-151">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="172c8-152">Это маркер, который позволяет приложению получать доступ к Microsoft Graph от имени пользователя.</span><span class="sxs-lookup"><span data-stu-id="172c8-152">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="172c8-153">Однако этот маркер кратковременно используется.</span><span class="sxs-lookup"><span data-stu-id="172c8-153">However, this token is short-lived.</span></span> <span data-ttu-id="172c8-154">Срок действия маркера истечет через час после его выдачи.</span><span class="sxs-lookup"><span data-stu-id="172c8-154">The token expires an hour after it is issued.</span></span> <span data-ttu-id="172c8-155">В этом случае маркер обновления становится полезен.</span><span class="sxs-lookup"><span data-stu-id="172c8-155">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="172c8-156">Маркер обновления позволяет приложению запросить новый маркер доступа, не требуя от пользователя повторного входа.</span><span class="sxs-lookup"><span data-stu-id="172c8-156">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="172c8-157">Обновите код управления маркером, чтобы реализовать обновление маркеров.</span><span class="sxs-lookup"><span data-stu-id="172c8-157">Update the token management code to implement token refresh.</span></span>

1. <span data-ttu-id="172c8-158">Откройте **./АПП/токенсторе/токенкаче.ФП** и добавьте в класс следующую функцию `TokenCache` .</span><span class="sxs-lookup"><span data-stu-id="172c8-158">Open **./app/TokenStore/TokenCache.php** and add the following function to the `TokenCache` class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="UpdateTokensSnippet":::

1. <span data-ttu-id="172c8-159">Замените имеющуюся функцию `getAccessToken` указанным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="172c8-159">Replace the existing `getAccessToken` function with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="GetAccessTokenSnippet":::

<span data-ttu-id="172c8-160">Этот метод сначала проверяет, истек ли срок действия маркера доступа, или закройте его до истечения срока действия.</span><span class="sxs-lookup"><span data-stu-id="172c8-160">This method first checks if the access token is expired or close to expiring.</span></span> <span data-ttu-id="172c8-161">Если это так, то он использует маркер обновления для получения новых токенов, затем обновляет кэш и возвращает новый маркер доступа.</span><span class="sxs-lookup"><span data-stu-id="172c8-161">If it is, then it uses the refresh token to get new tokens, then updates the cache and returns the new access token.</span></span>
