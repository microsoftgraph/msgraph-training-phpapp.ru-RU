<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="77d02-101">В этом упражнении вы будете расширяем приложение из предыдущего упражнения для поддержки проверки подлинности с помощью Azure AD.</span><span class="sxs-lookup"><span data-stu-id="77d02-101">In this exercise you will extend the application from the previous exercise to support authentication with Azure AD.</span></span> <span data-ttu-id="77d02-102">Это необходимо для получения необходимого маркера доступа OAuth для вызова Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="77d02-102">This is required to obtain the necessary OAuth access token to call the Microsoft Graph.</span></span> <span data-ttu-id="77d02-103">На этом этапе в приложение будет интегрирована библиотека [OAuth2 – Client](https://github.com/thephpleague/oauth2-client) .</span><span class="sxs-lookup"><span data-stu-id="77d02-103">In this step you will integrate the [oauth2-client](https://github.com/thephpleague/oauth2-client) library into the application.</span></span>

<span data-ttu-id="77d02-104">Откройте `.env` файл в корневом каталоге приложения PHP и добавьте следующий код в конец файла.</span><span class="sxs-lookup"><span data-stu-id="77d02-104">Open the `.env` file in the root of your PHP application, and add the following code to the end of the file.</span></span>

```text
OAUTH_APP_ID=YOUR_APP_ID_HERE
OAUTH_APP_PASSWORD=YOUR_APP_PASSWORD_HERE
OAUTH_REDIRECT_URI=http://localhost:8000/callback
OAUTH_SCOPES='openid profile offline_access user.read calendars.read'
OAUTH_AUTHORITY=https://login.microsoftonline.com/common
OAUTH_AUTHORIZE_ENDPOINT=/oauth2/v2.0/authorize
OAUTH_TOKEN_ENDPOINT=/oauth2/v2.0/token
```

<span data-ttu-id="77d02-105">Замените `YOUR APP ID HERE` идентификатором приложения на портале регистрации приложений и замените `YOUR APP SECRET HERE` созданным паролем.</span><span class="sxs-lookup"><span data-stu-id="77d02-105">Replace `YOUR APP ID HERE` with the application ID from the Application Registration Portal, and replace `YOUR APP SECRET HERE` with the password you generated.</span></span>

> [!IMPORTANT]
> <span data-ttu-id="77d02-106">Если вы используете систему управления версиями (например, Git), то в дальнейшем будет полезно исключить `.env` файл из системы управления версиями, чтобы избежать непреднамеренного утечки идентификатора и пароля приложения.</span><span class="sxs-lookup"><span data-stu-id="77d02-106">If you're using source control such as git, now would be a good time to exclude the `.env` file from source control to avoid inadvertently leaking your app ID and password.</span></span>

## <a name="implement-sign-in"></a><span data-ttu-id="77d02-107">Реализация входа</span><span class="sxs-lookup"><span data-stu-id="77d02-107">Implement sign-in</span></span>

<span data-ttu-id="77d02-108">Создайте новый файл в `./app/Http/Controllers` каталоге `AuthController.php` и добавьте указанный ниже код.</span><span class="sxs-lookup"><span data-stu-id="77d02-108">Create a new file in the `./app/Http/Controllers` directory named `AuthController.php` and add the following code.</span></span>

```PHP
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

    if (!isset($expectedState) || !isset($providedState) || $expectedState != $providedState) {
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

<span data-ttu-id="77d02-109">Этот параметр определяет контроллер с двумя действиями `signin` : `callback`и.</span><span class="sxs-lookup"><span data-stu-id="77d02-109">This defines a controller with two actions: `signin` and `callback`.</span></span>

<span data-ttu-id="77d02-110">`signin` Действие создает URL-адрес входа Azure AD, сохраняет `state` значение, созданное клиентом OAuth, а затем перенаправляет браузер на страницу входа Azure AD.</span><span class="sxs-lookup"><span data-stu-id="77d02-110">The `signin` action generates the Azure AD signin URL, saves the `state` value generated by the OAuth client, then redirects the browser to the Azure AD signin page.</span></span>

<span data-ttu-id="77d02-111">`callback` Действие перенаправляет Azure после завершения входа.</span><span class="sxs-lookup"><span data-stu-id="77d02-111">The `callback` action is where Azure redirects after the signin is complete.</span></span> <span data-ttu-id="77d02-112">Это действие гарантирует, что `state` значение соответствует сохраненному значению, а затем пользователю код авторизации, отправленный Azure, запрашивает маркер доступа.</span><span class="sxs-lookup"><span data-stu-id="77d02-112">That action makes sure the `state` value matches the saved value, then users the authorization code sent by Azure to request an access token.</span></span> <span data-ttu-id="77d02-113">Затем он перенаправляется обратно на домашнюю страницу с маркером доступа во временном значении ошибки.</span><span class="sxs-lookup"><span data-stu-id="77d02-113">It then redirects back to the home page with the access token in the temporary error value.</span></span> <span data-ttu-id="77d02-114">Мы будем использовать эту проверку, чтобы убедиться, что наш вход работает перед переходом.</span><span class="sxs-lookup"><span data-stu-id="77d02-114">We'll use this to verify that our sign-in is working before moving on.</span></span> <span data-ttu-id="77d02-115">Перед тестированием необходимо добавить маршруты в `./routes/web.php`.</span><span class="sxs-lookup"><span data-stu-id="77d02-115">Before we test, we need to add the routes to `./routes/web.php`.</span></span>

```PHP
Route::get('/signin', 'AuthController@signin');
Route::get('/callback', 'AuthController@callback');
```

<span data-ttu-id="77d02-116">Запустите сервер и перейдите к `https://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="77d02-116">Start the server and browse to `https://localhost:8000`.</span></span> <span data-ttu-id="77d02-117">Нажмите кнопку входа, и вы будете перенаправлены на `https://login.microsoftonline.com`.</span><span class="sxs-lookup"><span data-stu-id="77d02-117">Click the sign-in button and you should be redirected to `https://login.microsoftonline.com`.</span></span> <span data-ttu-id="77d02-118">Войдите с помощью учетной записи Майкрософт и согласия с запрошенными разрешениями.</span><span class="sxs-lookup"><span data-stu-id="77d02-118">Login with your Microsoft account and consent to the requested permissions.</span></span> <span data-ttu-id="77d02-119">Браузер перенаправляется на приложение, отображая маркер.</span><span class="sxs-lookup"><span data-stu-id="77d02-119">The browser redirects to the app, showing the token.</span></span>

### <a name="get-user-details"></a><span data-ttu-id="77d02-120">Получение сведений о пользователе</span><span class="sxs-lookup"><span data-stu-id="77d02-120">Get user details</span></span>

<span data-ttu-id="77d02-121">Обновите `callback` метод в `/app/Http/Controllers/AuthController.php` , чтобы получить профиль пользователя из Microsoft Graph.</span><span class="sxs-lookup"><span data-stu-id="77d02-121">Update the `callback` method in `/app/Http/Controllers/AuthController.php` to get the user's profile from Microsoft Graph.</span></span>

<span data-ttu-id="77d02-122">Сначала добавьте следующие `use` операторы в верхнюю часть файла, под `namespace App\Http\Controllers;` строкой.</span><span class="sxs-lookup"><span data-stu-id="77d02-122">First, add the following `use` statements to the top of the file, beneath the `namespace App\Http\Controllers;` line.</span></span>

```php
use Microsoft\Graph\Graph;
use Microsoft\Graph\Model;
```

<span data-ttu-id="77d02-123">Замените `try` блок в `callback` методе на приведенный ниже код.</span><span class="sxs-lookup"><span data-stu-id="77d02-123">Replace the `try` block in the `callback` method with the following code.</span></span>

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

<span data-ttu-id="77d02-124">Новый код создает `Graph` объект, назначает маркер доступа, а затем использует его для запроса профиля пользователя.</span><span class="sxs-lookup"><span data-stu-id="77d02-124">The new code creates a `Graph` object, assigns the access token, then uses it to request the user's profile.</span></span> <span data-ttu-id="77d02-125">Он добавляет отображаемое имя пользователя во временный вывод для тестирования.</span><span class="sxs-lookup"><span data-stu-id="77d02-125">It adds the user's display name to the temporary output for testing.</span></span>

## <a name="storing-the-tokens"></a><span data-ttu-id="77d02-126">Сохранение маркеров</span><span class="sxs-lookup"><span data-stu-id="77d02-126">Storing the tokens</span></span>

<span data-ttu-id="77d02-127">Теперь, когда вы можете получить маркеры, следует реализовать способ их хранения в приложении.</span><span class="sxs-lookup"><span data-stu-id="77d02-127">Now that you can get tokens, it's time to implement a way to store them in the app.</span></span> <span data-ttu-id="77d02-128">Так как это пример приложения, для простоты вы будете хранить их в сеансе.</span><span class="sxs-lookup"><span data-stu-id="77d02-128">Since this is a sample app, for simplicity's sake, you'll store them in the session.</span></span> <span data-ttu-id="77d02-129">Реальное приложение использует более надежное решение для безопасного хранения, например базу данных.</span><span class="sxs-lookup"><span data-stu-id="77d02-129">A real-world app would use a more reliable secure storage solution, like a database.</span></span>

<span data-ttu-id="77d02-130">Создайте каталог в `./app` каталоге с именем `TokenStore`, затем создайте в этом каталоге новый файл с именем `TokenCache.php`и добавьте следующий код.</span><span class="sxs-lookup"><span data-stu-id="77d02-130">Create a new directory in the `./app` directory named `TokenStore`, then create a new file in that directory named `TokenCache.php`, and add the following code.</span></span>

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

<span data-ttu-id="77d02-131">Затем обновите `callback` функцию в `AuthController` классе для хранения маркеров в сеансе и перенаправление обратно на главную страницу.</span><span class="sxs-lookup"><span data-stu-id="77d02-131">Then, update the `callback` function in the `AuthController` class to store the tokens in the session and redirect back to the main page.</span></span>

<span data-ttu-id="77d02-132">Сначала добавьте следующий `use` оператор в начало `./app/Http/Controllers/AuthController.php` `namespace App\Http\Controllers;` строки, расположенной ниже.</span><span class="sxs-lookup"><span data-stu-id="77d02-132">First, add the following `use` statement to the top of `./app/Http/Controllers/AuthController.php`, beneath the `namespace App\Http\Controllers;` line.</span></span>

```php
use App\TokenStore\TokenCache;
```

<span data-ttu-id="77d02-133">Затем замените `try` блок в функции existing `callback` приведенным ниже блоком.</span><span class="sxs-lookup"><span data-stu-id="77d02-133">Then replace the `try` block in the existing `callback` function with the following.</span></span>

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

  $tokenCache = new TokenCache();
  $tokenCache->storeTokens($accessToken, $user);

  return redirect('/');
}
```

## <a name="implement-sign-out"></a><span data-ttu-id="77d02-134">Реализация выхода</span><span class="sxs-lookup"><span data-stu-id="77d02-134">Implement sign-out</span></span>

<span data-ttu-id="77d02-135">Перед тестированием новой функции добавьте способ выхода. Добавьте в `AuthController` класс следующее действие.</span><span class="sxs-lookup"><span data-stu-id="77d02-135">Before you test this new feature, add a way to sign out. Add the following action to the `AuthController` class.</span></span>

```PHP
public function signout()
{
  $tokenCache = new TokenCache();
  $tokenCache->clearTokens();
  return redirect('/');
}
```

<span data-ttu-id="77d02-136">Добавьте это действие в `./routes/web.php`.</span><span class="sxs-lookup"><span data-stu-id="77d02-136">Add this action to `./routes/web.php`.</span></span>

```PHP
Route::get('/signout', 'AuthController@signout');
```

<span data-ttu-id="77d02-137">Перезапустите сервер и пройдите процесс входа.</span><span class="sxs-lookup"><span data-stu-id="77d02-137">Restart the server and go through the sign-in process.</span></span> <span data-ttu-id="77d02-138">Необходимо вернуться на домашнюю страницу, но пользовательский интерфейс должен измениться, чтобы показать, что вы вошли в систему.</span><span class="sxs-lookup"><span data-stu-id="77d02-138">You should end up back on the home page, but the UI should change to indicate that you are signed-in.</span></span>

![Снимок экрана домашней страницы после входа](./images/add-aad-auth-01.png)

<span data-ttu-id="77d02-140">Щелкните аватар пользователя в правом верхнем углу, чтобы получить доступ к ссылке **выхода** .</span><span class="sxs-lookup"><span data-stu-id="77d02-140">Click the user avatar in the top right corner to access the **Sign Out** link.</span></span> <span data-ttu-id="77d02-141">При нажатии кнопки **выйти** сбрасывается сеанс и возвращается на домашнюю страницу.</span><span class="sxs-lookup"><span data-stu-id="77d02-141">Clicking **Sign Out** resets the session and returns you to the home page.</span></span>

![Снимок экрана с раскрывающимся меню со ссылкой "выйти"](./images/add-aad-auth-02.png)

## <a name="refreshing-tokens"></a><span data-ttu-id="77d02-143">Обновление маркеров</span><span class="sxs-lookup"><span data-stu-id="77d02-143">Refreshing tokens</span></span>

<span data-ttu-id="77d02-144">На этом шаге приложение имеет маркер доступа, который отправляется в `Authorization` заголовке вызовов API.</span><span class="sxs-lookup"><span data-stu-id="77d02-144">At this point your application has an access token, which is sent in the `Authorization` header of API calls.</span></span> <span data-ttu-id="77d02-145">Это маркер, который позволяет приложению получать доступ к Microsoft Graph от имени пользователя.</span><span class="sxs-lookup"><span data-stu-id="77d02-145">This is the token that allows the app to access the Microsoft Graph on the user's behalf.</span></span>

<span data-ttu-id="77d02-146">Однако этот маркер кратковременно используется.</span><span class="sxs-lookup"><span data-stu-id="77d02-146">However, this token is short-lived.</span></span> <span data-ttu-id="77d02-147">Срок действия маркера истечет через час после его выдачи.</span><span class="sxs-lookup"><span data-stu-id="77d02-147">The token expires an hour after it is issued.</span></span> <span data-ttu-id="77d02-148">В этом случае маркер обновления становится полезен.</span><span class="sxs-lookup"><span data-stu-id="77d02-148">This is where the refresh token becomes useful.</span></span> <span data-ttu-id="77d02-149">Маркер обновления позволяет приложению запросить новый маркер доступа, не требуя от пользователя повторного входа.</span><span class="sxs-lookup"><span data-stu-id="77d02-149">The refresh token allows the app to request a new access token without requiring the user to sign in again.</span></span> <span data-ttu-id="77d02-150">Обновите код управления маркером, чтобы реализовать обновление маркеров.</span><span class="sxs-lookup"><span data-stu-id="77d02-150">Update the token management code to implement token refresh.</span></span>

<span data-ttu-id="77d02-151">Откройте `./app/TokenStore/TokenCache.php` и добавьте указанную ниже функцию в `TokenCache` класс.</span><span class="sxs-lookup"><span data-stu-id="77d02-151">Open `./app/TokenStore/TokenCache.php` and add the following function to the `TokenCache` class.</span></span>

```php
public function updateTokens($accessToken) {
  session([
    'accessToken' => $accessToken->getToken(),
    'refreshToken' => $accessToken->getRefreshToken(),
    'tokenExpires' => $accessToken->getExpires()
  ]);
}
```

<span data-ttu-id="77d02-152">Затем замените существующую `getAccessToken` функцию на приведенную ниже.</span><span class="sxs-lookup"><span data-stu-id="77d02-152">Then replace the existing `getAccessToken` function with the following.</span></span>

```php
public function getAccessToken() {
  // Check if tokens exist
  if (empty(session('accessToken')) ||
      empty(session('refreshToken')) ||
      empty(session('tokenExpires'))) {
    return '';
  }

  // Check if token is expired
  //Get current time + 5 minutes (to allow for time differences)
  $now = time() + 300;
  if (session('tokenExpires') <= $now) {
    // Token is expired (or very close to it)
    // so let's refresh

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
      $newToken = $oauthClient->getAccessToken('refresh_token', [
        'refresh_token' => session('refreshToken')
      ]);

      // Store the new values
      $this->updateTokens($newToken);

      return $newToken->getToken();
    }
    catch (League\OAuth2\Client\Provider\Exception\IdentityProviderException $e) {
      return '';
    }
  }

  // Token is still valid, just return it
  return session('accessToken');
}
```

<span data-ttu-id="77d02-153">Этот метод сначала проверяет, истек ли срок действия маркера доступа, или закройте его до истечения срока действия.</span><span class="sxs-lookup"><span data-stu-id="77d02-153">This method first checks if the access token is expired or close to expiring.</span></span> <span data-ttu-id="77d02-154">Если это так, то он использует маркер обновления для получения новых токенов, затем обновляет кэш и возвращает новый маркер доступа.</span><span class="sxs-lookup"><span data-stu-id="77d02-154">If it is, then it uses the refresh token to get new tokens, then updates the cache and returns the new access token.</span></span>