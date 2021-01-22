<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="66790-101">В этом упражнении вы расширим приложение из предыдущего упражнения, чтобы поддерживать проверку подлинности с помощью Azure AD.</span><span class="sxs-lookup"><span data-stu-id="66790-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="66790-102">Это необходимо для получения необходимого маркера доступа OAuth для вызова Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="66790-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="66790-103">На этом этапе вы интегрируете клиентскую библиотеку [oauth2](https://github.com/thephpleague/oauth2-client) в приложение.</span><span class="sxs-lookup"><span data-stu-id="66790-103">In this step you will integrate the [oauth2-client](https://github.com/thephpleague/oauth2-client) library into the application.</span></span>

1. <span data-ttu-id="66790-104">Откройте **ENV-файл** в корневом файле приложения PHP и добавьте следующий код в конец файла.</span><span class="sxs-lookup"><span data-stu-id="66790-104">Open the **.env** file in the root of your PHP application, and add the following code to the end of the file.</span></span>

    :::code language="ini" source="../demo/graph-tutorial/example.env" range="48-54":::

1. <span data-ttu-id="66790-105">Замените код приложения на портале регистрации приложений `YOUR_APP_ID_HERE` и пароль, `YOUR_APP_PASSWORD_HERE` созданный вами.</span><span class="sxs-lookup"><span data-stu-id="66790-105">Replace `YOUR_APP_ID_HERE` with the application ID from the Application Registration Portal, and replace `YOUR_APP_PASSWORD_HERE` with the password you generated.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="66790-106">Если вы используете управление исходным кодом, например git, пришло бы время исключить файл из системы управления источником, чтобы избежать случайной утечки ИД приложения и `.env` пароля.</span><span class="sxs-lookup"><span data-stu-id="66790-106">If you're using source control such as git, now would be a good time to exclude the `.env` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="66790-107">Реализация входов</span><span class="sxs-lookup"><span data-stu-id="66790-107">Implement sign-in</span></span>

1. <span data-ttu-id="66790-108">Создайте файл с именем **./app/Http/Controllers** и добавьте `AuthController.php` следующий код.</span><span class="sxs-lookup"><span data-stu-id="66790-108">Create a new file in the **./app/Http/Controllers** directory named `AuthController.php` and add the following code.</span></span>

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

    <span data-ttu-id="66790-109">Он определяет контроллер с двумя действиями: `signin` и `callback` .</span><span class="sxs-lookup"><span data-stu-id="66790-109">This defines a controller with two actions: `signin` and `callback`.</span></span>

    <span data-ttu-id="66790-110">Это действие создает URL-адрес для подписи Azure AD, сохраняет значение, сгенерированное клиентом OAuth, а затем перенаправляет браузер на страницу для подписи `signin` `state` Azure AD.</span><span class="sxs-lookup"><span data-stu-id="66790-110">The `signin` action generates the Azure AD signin URL, saves the `state` value generated by the OAuth client, then redirects the browser to the Azure AD signin page.</span></span>

    <span data-ttu-id="66790-111">Это `callback` действие перенаправляет Azure после завершения регистрации.</span><span class="sxs-lookup"><span data-stu-id="66790-111">The `callback` action is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="66790-112">Это действие позволяет убедиться, что значение совпадает с сохраненным значением, а затем пользователи код авторизации, отправленный `state` Azure для запроса маркера доступа.</span><span class="sxs-lookup"><span data-stu-id="66790-112">That action makes sure the `state` value matches the saved value, then users the authorization code sent by Azure to request an access token.</span></span> <span data-ttu-id="66790-113">Затем он перенаправляется обратно на домашную страницу с маркером доступа во временном значении ошибки.</span><span class="sxs-lookup"><span data-stu-id="66790-113">It then redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="66790-114">Вы будете использовать его, чтобы убедиться, что вход работает, прежде чем двигаться дальше.</span><span class="sxs-lookup"><span data-stu-id="66790-114">You'll use this to verify that sign-in is working before moving on.</span></span>

1. <span data-ttu-id="66790-115">Добавьте маршруты в **./routes/web.php.**</span><span class="sxs-lookup"><span data-stu-id="66790-115">Add the routes to **./routes/web.php**.</span></span>

    ```php
    Route::get('/signin', 'AuthController@signin');
    Route::get('/callback', 'AuthController@callback');
    ```

1. <span data-ttu-id="66790-116">Запустите сервер и перейдите к `https://localhost:8000` .</span><span class="sxs-lookup"><span data-stu-id="66790-116">Start the server and browse to `https://localhost:8000`.</span></span> <span data-ttu-id="66790-117">Нажмите кнопку "Вход" и перенаправите вас на `https://login.microsoftonline.com` .</span><span class="sxs-lookup"><span data-stu-id="66790-117">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="66790-118">Войдите с помощью учетной записи Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="66790-118">Login with your Microsoft account.</span></span>

1. <span data-ttu-id="66790-119">Проверьте запрос согласия.</span><span class="sxs-lookup"><span data-stu-id="66790-119">Examine the consent prompt.</span></span> <span data-ttu-id="66790-120">Список разрешений соответствует списку областей разрешений, настроенных в **ENV.**</span><span class="sxs-lookup"><span data-stu-id="66790-120">The list of permissions correspond to list of permissions scopes configured in **.env**.</span></span>

    - <span data-ttu-id="66790-121">**Поддержив доступ** к данным, к ним предоставлен доступ: ( ) Это разрешение запрашивается MSAL для получения `offline_access` маркеров обновления.</span><span class="sxs-lookup"><span data-stu-id="66790-121">**Maintain access to data you have given it access to:** (`offline_access`) This permission is requested by MSAL in order to retrieve refresh tokens.</span></span>
    - <span data-ttu-id="66790-122">**Во sign you in and read your profile:** ( `User.Read` ) This permission allows the app to get the logged-in user's profile and profile photo.</span><span class="sxs-lookup"><span data-stu-id="66790-122">**Sign you in and read your profile:** (`User.Read`) This permission allows the app to get the logged-in user's profile and profile photo.</span></span>
    - <span data-ttu-id="66790-123">**Прочитайте параметры** почтового ящика: ( ) Это разрешение позволяет приложению читать параметры почтового ящика пользователя, включая часовой пояс `MailboxSettings.Read` и формат времени.</span><span class="sxs-lookup"><span data-stu-id="66790-123">**Read your mailbox settings:** (`MailboxSettings.Read`) This permission allows the app to read the user's mailbox settings, including time zone and time format.</span></span>
    - <span data-ttu-id="66790-124">**Полный доступ к** календарям: ( ) Это разрешение позволяет приложению читать события в календаре пользователя, добавлять новые события и изменять `Calendars.ReadWrite` существующие.</span><span class="sxs-lookup"><span data-stu-id="66790-124">**Have full access to your calendars:** (`Calendars.ReadWrite`) This permission allows the app to read events on the user's calendar, add new events, and modify existing ones.</span></span>

1. <span data-ttu-id="66790-125">Согласие на запрашиваемую разрешения.</span><span class="sxs-lookup"><span data-stu-id="66790-125">Consent to the requested permissions.</span></span> <span data-ttu-id="66790-126">Браузер перенаправляет пользователя в приложение с отображением маркера.</span><span class="sxs-lookup"><span data-stu-id="66790-126">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="66790-127">Получить сведения о пользователе</span><span class="sxs-lookup"><span data-stu-id="66790-127">Get user details</span></span>

<span data-ttu-id="66790-128">В этом разделе вы обновим метод для получения профиля пользователя `callback` из Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="66790-128">In this section you'll update the `callback` method to get the user's profile from Microsoft Graph.</span></span>

1. <span data-ttu-id="66790-129">Добавьте следующие утверждения в верхнюю часть `use` **/app/Http/Controllers/AuthController.php** под `namespace App\Http\Controllers;` строкой.</span><span class="sxs-lookup"><span data-stu-id="66790-129">Add the following `use` statements to the top of **/app/Http/Controllers/AuthController.php**, beneath the `namespace App\Http\Controllers;` line.</span></span>

    ```php
    use Microsoft\Graph\Graph;
    use Microsoft\Graph\Model;
    ```

1. <span data-ttu-id="66790-130">Замените `try` блок в `callback` методе следующим кодом.</span><span class="sxs-lookup"><span data-stu-id="66790-130">Replace the `try` block in the `callback` method with the following code.</span></span>

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

<span data-ttu-id="66790-131">Новый код создает объект, назначает маркер доступа, а затем использует его для запроса `Graph` профиля пользователя.</span><span class="sxs-lookup"><span data-stu-id="66790-131">The new code creates a `Graph` object, assigns the access token, then uses it to request the user's profile.</span></span> <span data-ttu-id="66790-132">Он добавляет отображаемую имя пользователя во временные выходные данные для тестирования.</span><span class="sxs-lookup"><span data-stu-id="66790-132">It adds the user's display name to the temporary output for testing.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="66790-133">Хранение маркеров</span><span class="sxs-lookup"><span data-stu-id="66790-133">Storing the tokens</span></span>

<span data-ttu-id="66790-134">Теперь, когда вы можете получить маркеры, пора реализовать способ их хранения в приложении.</span><span class="sxs-lookup"><span data-stu-id="66790-134">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="66790-135">Так как это пример приложения, для простоты вы храните их в сеансе.</span><span class="sxs-lookup"><span data-stu-id="66790-135">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="66790-136">В реальных приложениях используется более надежное решение для безопасного хранения данных, например база данных.</span><span class="sxs-lookup"><span data-stu-id="66790-136">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

1. <span data-ttu-id="66790-137">Создайте новый каталог в **каталоге ./app** с именем , а затем создайте новый файл с именем и `TokenStore` добавьте следующий `TokenCache.php` код.</span><span class="sxs-lookup"><span data-stu-id="66790-137">Create a new directory in the **./app** directory named `TokenStore`, then create a new file in that directory named `TokenCache.php`, and add the following code.</span></span>

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

1. <span data-ttu-id="66790-138">Добавьте следующий выписку в верхнюю часть `use` **./app/Http/Controllers/AuthController.php** под `namespace App\Http\Controllers;` строкой.</span><span class="sxs-lookup"><span data-stu-id="66790-138">Add the following `use` statement to the top of **./app/Http/Controllers/AuthController.php**, beneath the `namespace App\Http\Controllers;` line.</span></span>

    ```php
    use App\TokenStore\TokenCache;
    ```

1. <span data-ttu-id="66790-139">Замените `try` блок в существующей `callback` функции следующим:</span><span class="sxs-lookup"><span data-stu-id="66790-139">Replace the `try` block in the existing `callback` function with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="StoreTokensSnippet":::

## <a name="implement-sign-out"></a><span data-ttu-id="66790-140">Реализация выходов</span><span class="sxs-lookup"><span data-stu-id="66790-140">Implement sign-out</span></span>

<span data-ttu-id="66790-141">Перед тестированием этой новой функции добавьте способ выйти из нее.</span><span class="sxs-lookup"><span data-stu-id="66790-141">Before you test this new feature, add a way to sign out.</span></span>

1. <span data-ttu-id="66790-142">Добавьте в класс следующее `AuthController` действие.</span><span class="sxs-lookup"><span data-stu-id="66790-142">Add the following action to the `AuthController` class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/Http/Controllers/AuthController.php" id="SignOutSnippet":::

1. <span data-ttu-id="66790-143">Добавьте это действие в **./routes/web.php.**</span><span class="sxs-lookup"><span data-stu-id="66790-143">Add this action to **./routes/web.php**.</span></span>

    ```php
    Route::get('/signout', 'AuthController@signout');
    ```

1. <span data-ttu-id="66790-144">Перезапустите сервер и войдите в нее.</span><span class="sxs-lookup"><span data-stu-id="66790-144">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="66790-145">Вы должны вернуться на home-страницу, но пользовательский интерфейс должен измениться, чтобы указать, что вы вписались.</span><span class="sxs-lookup"><span data-stu-id="66790-145">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

    ![Снимок экрана с домашней страницей после входов](./images/add-aad-auth-01.png)

1. <span data-ttu-id="66790-147">Щелкните аватар пользователя в правом верхнем углу, чтобы получить доступ к ссылке **"Выйти".**</span><span class="sxs-lookup"><span data-stu-id="66790-147">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="66790-148">При **нажатии** кнопки "Выйти" сеанс сбрасывается и возвращается на домашней странице.</span><span class="sxs-lookup"><span data-stu-id="66790-148">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

    ![Снимок экрана с выпадающим меню со ссылкой "Выйти"](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="66790-150">Обновление маркеров</span><span class="sxs-lookup"><span data-stu-id="66790-150">Refreshing tokens</span></span>

<span data-ttu-id="66790-151">На этом этапе приложение имеет маркер доступа, который отправляется в `Authorization` заголовке вызовов API.</span><span class="sxs-lookup"><span data-stu-id="66790-151">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="66790-152">Это маркер, который позволяет приложению получать доступ к Microsoft Graph от имени пользователя.</span><span class="sxs-lookup"><span data-stu-id="66790-152">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="66790-153">Однако этот маркер является кратковременной.</span><span class="sxs-lookup"><span data-stu-id="66790-153">However, this token is short-lived.</span></span> <span data-ttu-id="66790-154">Срок действия маркера истекает через час после его выпуска.</span><span class="sxs-lookup"><span data-stu-id="66790-154">The token expires an hour after it is issued.</span></span> <span data-ttu-id="66790-155">В этом случае маркер обновления становится полезным.</span><span class="sxs-lookup"><span data-stu-id="66790-155">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="66790-156">Маркер обновления позволяет приложению запрашивать новый маркер доступа, не требуя от пользователя повторного входить.</span><span class="sxs-lookup"><span data-stu-id="66790-156">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="66790-157">Обновите код управления маркерами, чтобы реализовать обновление маркера.</span><span class="sxs-lookup"><span data-stu-id="66790-157">Update the token management code to implement token refresh.</span></span>

1. <span data-ttu-id="66790-158">Откройте **./app/TokenStore/TokenCache.php** и добавьте в класс следующую `TokenCache` функцию.</span><span class="sxs-lookup"><span data-stu-id="66790-158">Open **./app/TokenStore/TokenCache.php** and add the following function to the `TokenCache` class.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="UpdateTokensSnippet":::

1. <span data-ttu-id="66790-159">Замените имеющуюся функцию `getAccessToken` указанным ниже кодом.</span><span class="sxs-lookup"><span data-stu-id="66790-159">Replace the existing `getAccessToken` function with the following.</span></span>

    :::code language="php" source="../demo/graph-tutorial/app/TokenStore/TokenCache.php" id="GetAccessTokenSnippet":::

<span data-ttu-id="66790-160">Этот метод сначала проверяет, истек ли срок действия маркера доступа или почти истек.</span><span class="sxs-lookup"><span data-stu-id="66790-160">This method first checks if the access token is expired or close to expiring.</span></span> <span data-ttu-id="66790-161">Если это так, то он использует маркер обновления для получения новых маркеров, а затем обновляет кэш и возвращает новый маркер доступа.</span><span class="sxs-lookup"><span data-stu-id="66790-161">If it is, then it uses the refresh token to get new tokens, then updates the cache and returns the new access token.</span></span>
