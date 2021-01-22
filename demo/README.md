# <a name="how-to-run-the-completed-project"></a><span data-ttu-id="c5eb5-101">Как запустить завершенный проект</span><span class="sxs-lookup"><span data-stu-id="c5eb5-101">How to run the completed project</span></span>

## <a name="prerequisites"></a><span data-ttu-id="c5eb5-102">Предварительные условия</span><span class="sxs-lookup"><span data-stu-id="c5eb5-102">Prerequisites</span></span>

<span data-ttu-id="c5eb5-103">Чтобы запустить завершенный проект в этой папке, необходимо следующее:</span><span class="sxs-lookup"><span data-stu-id="c5eb5-103">To run the completed project in this folder, you need the following:</span></span>

- <span data-ttu-id="c5eb5-104">[PHP,](http://php.net/downloads.php) установленный на компьютере разработчика.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-104">[PHP](http://php.net/downloads.php) installed on your development machine.</span></span> <span data-ttu-id="c5eb5-105">Если у вас нет PHP, посетите предыдущую ссылку для скачивания.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-105">If you do not have PHP, visit the previous link for download options.</span></span> <span data-ttu-id="c5eb5-106">(**Примечание.** Это руководство было написано с помощью PHP версии 7.4.4.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-106">(**Note:** This tutorial was written with PHP version 7.4.4.</span></span> <span data-ttu-id="c5eb5-107">Действия в этом руководстве могут работать с другими версиями, но не были протестированы.)</span><span class="sxs-lookup"><span data-stu-id="c5eb5-107">The steps in this guide may work with other versions, but that has not been tested.)</span></span>
- <span data-ttu-id="c5eb5-108">[Автор,](https://getcomposer.org/) установленный на компьютере разработчика.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-108">[Composer](https://getcomposer.org/) installed on your development machine.</span></span>
- <span data-ttu-id="c5eb5-109">[Laravel,](https://laravel.com/) установленный на компьютере разработчика.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-109">[Laravel](https://laravel.com/) installed on your development machine.</span></span>
- <span data-ttu-id="c5eb5-110">Личная учетная запись Майкрософт с почтовым ящиком Outlook.com или учетная запись Майкрософт для работы или учебного заведения.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-110">Either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span>

<span data-ttu-id="c5eb5-111">Если у вас нет учетной записи Майкрософт, существует несколько вариантов получения бесплатной учетной записи:</span><span class="sxs-lookup"><span data-stu-id="c5eb5-111">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="c5eb5-112">Вы можете [зарегистрироваться для новой личной учетной записи Майкрософт.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)</span><span class="sxs-lookup"><span data-stu-id="c5eb5-112">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="c5eb5-113">Вы можете зарегистрироваться в программе для разработчиков [Office 365,](https://developer.microsoft.com/office/dev-program) чтобы получить бесплатную подписку на Office 365.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-113">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

## <a name="register-a-web-application-with-the-azure-active-directory-admin-center"></a><span data-ttu-id="c5eb5-114">Регистрация веб-приложения в Центре администрирования Azure Active Directory</span><span class="sxs-lookup"><span data-stu-id="c5eb5-114">Register a web application with the Azure Active Directory admin center</span></span>

1. <span data-ttu-id="c5eb5-115">Откройте браузер и перейдите к [Центру администрирования Azure Active Directory](https://aad.portal.azure.com).</span><span class="sxs-lookup"><span data-stu-id="c5eb5-115">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com).</span></span> <span data-ttu-id="c5eb5-116">Войдите с помощью **личной учетной записи** (т.е. учетной записи Microsoft) или **рабочей (учебной) учетной записи**.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-116">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="c5eb5-117">Выберите **Azure Active Directory** на панели навигации слева, затем выберите **Регистрация приложений** в разделе **Управление**.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-117">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations** under **Manage**.</span></span>

    ![<span data-ttu-id="c5eb5-118">Снимок экрана с регистрацией приложений</span><span class="sxs-lookup"><span data-stu-id="c5eb5-118">A screenshot of the App registrations</span></span> ](/tutorial/images/aad-portal-app-registrations.png)

1. <span data-ttu-id="c5eb5-119">Выберите **Новая регистрация**.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-119">Select **New registration**.</span></span> <span data-ttu-id="c5eb5-120">На странице **Зарегистрировать приложение** задайте необходимые значения следующим образом.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-120">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="c5eb5-121">Введите **имя** `PHP Graph Tutorial`.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-121">Set **Name** to `PHP Graph Tutorial`.</span></span>
    - <span data-ttu-id="c5eb5-122">Введите **поддерживаемые типы учетных записей** для **учетных записей в любом каталоге организаций и личных учетных записей Microsoft**.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-122">Set **Supported account types** to **Accounts in any organizational directory and personal Microsoft accounts**.</span></span>
    - <span data-ttu-id="c5eb5-123">В разделе **URI адрес перенаправления** введите значение в первом раскрывающемся списке `Web` и задайте значение `http://localhost:8000/callback`.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-123">Under **Redirect URI**, set the first drop-down to `Web` and set the value to `http://localhost:8000/callback`.</span></span>

    ![Снимок экрана: страница "Регистрация приложения"](/tutorial/images/aad-register-an-app.png)

1. <span data-ttu-id="c5eb5-125">Нажмите кнопку **Зарегистрировать**.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-125">Choose **Register**.</span></span> <span data-ttu-id="c5eb5-126">На странице **учебника PHP Graph** скопируйте значение ИД приложения **(клиента)** и сохраните его, оно потребуется на следующем этапе.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-126">On the **PHP Graph Tutorial** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

    ![Снимок экрана: ИД нового приложения для регистрации](/tutorial/images/aad-application-id.png)

1. <span data-ttu-id="c5eb5-128">Выберите **Сертификаты и секреты** в разделе **Управление**.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-128">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="c5eb5-129">Нажмите кнопку **Новый секрет клиента**.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-129">Select the **New client secret** button.</span></span> <span data-ttu-id="c5eb5-130">Введите значение в **Описание** и выберите один из вариантов для **Срок действия** и нажмите **Добавить**.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-130">Enter a value in **Description** and select one of the options for **Expires** and choose **Add**.</span></span>

    ![Снимок экрана: диалоговое окно "Добавление секрета клиента"](/tutorial/images/aad-new-client-secret.png)

1. <span data-ttu-id="c5eb5-132">Скопируйте значение секрета клиента, а затем покиньте эту страницу.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-132">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="c5eb5-133">Оно вам понадобится на следующем шаге.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-133">You will need it in the next step.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="c5eb5-134">Это секрет клиента, он никогда не отображается еще раз, поэтому убедитесь, что вы скопировали его.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-134">This client secret is never shown again, so make sure you copy it now.</span></span>

    ![Снимок экрана с недавно добавленным секретом клиента](/tutorial/images/aad-copy-client-secret.png)

## <a name="configure-the-sample"></a><span data-ttu-id="c5eb5-136">Настройка примера</span><span class="sxs-lookup"><span data-stu-id="c5eb5-136">Configure the sample</span></span>

1. <span data-ttu-id="c5eb5-137">Переименуем `example.env` файл в `.env` .</span><span class="sxs-lookup"><span data-stu-id="c5eb5-137">Rename the `example.env` file to `.env`.</span></span>
1. <span data-ttu-id="c5eb5-138">`.env`Отредактируете файл и внести следующие изменения.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-138">Edit the `.env` file and make the following changes.</span></span>
    1. <span data-ttu-id="c5eb5-139">`YOUR_APP_ID_HERE`Замените **ид приложения,** который вы получили с портала регистрации приложений.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-139">Replace `YOUR_APP_ID_HERE` with the **Application Id** you got from the App Registration Portal.</span></span>
    1. <span data-ttu-id="c5eb5-140">Замените `YOUR_APP_PASSWORD_HERE` пароль, который вы получили с портала регистрации приложений.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-140">Replace `YOUR_APP_PASSWORD_HERE` with the password you got from the App Registration Portal.</span></span>
1. <span data-ttu-id="c5eb5-141">В интерфейсе командной строки перейдите в этот каталог и запустите следующую команду, чтобы установить требования.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-141">In your command-line interface (CLI), navigate to this directory and run the following command to install requirements.</span></span>

    ```Shell
    composer install
    ```

1. <span data-ttu-id="c5eb5-142">В интерфейсе командной строки (CLI) запустите следующую команду, чтобы создать ключ приложения.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-142">In your command-line interface (CLI), run the following command to generate an application key.</span></span>

    ```Shell
    php artisan key:generate
    ```

## <a name="run-the-sample"></a><span data-ttu-id="c5eb5-143">Запуск приложения</span><span class="sxs-lookup"><span data-stu-id="c5eb5-143">Run the sample</span></span>

1. <span data-ttu-id="c5eb5-144">Чтобы запустить приложение, запустите следующую команду в CLI.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-144">Run the following command in your CLI to start the application.</span></span>

    ```Shell
    php artisan serve
    ```

1. <span data-ttu-id="c5eb5-145">Откройте браузер и перейдите по адресу `http://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="c5eb5-145">Open a browser and browse to `http://localhost:8000`.</span></span>
