# <a name="how-to-run-the-completed-project"></a><span data-ttu-id="a452d-101">Выполнение завершенного проекта</span><span class="sxs-lookup"><span data-stu-id="a452d-101">How to run the completed project</span></span>

## <a name="prerequisites"></a><span data-ttu-id="a452d-102">Необходимые компоненты</span><span class="sxs-lookup"><span data-stu-id="a452d-102">Prerequisites</span></span>

<span data-ttu-id="a452d-103">Чтобы запустить завершенный проект в этой папке, вам потребуются следующие компоненты:</span><span class="sxs-lookup"><span data-stu-id="a452d-103">To run the completed project in this folder, you need the following:</span></span>

- <span data-ttu-id="a452d-104">[PHP](http://php.net/downloads.php) , установленный на компьютере для разработки.</span><span class="sxs-lookup"><span data-stu-id="a452d-104">[PHP](http://php.net/downloads.php) installed on your development machine.</span></span> <span data-ttu-id="a452d-105">Если у вас нет PHP, посетите предыдущую ссылку для получения вариантов загрузки.</span><span class="sxs-lookup"><span data-stu-id="a452d-105">If you do not have PHP, visit the previous link for download options.</span></span> <span data-ttu-id="a452d-106">(**Примечание:** это руководство было написано с помощью PHP версии 7,2.</span><span class="sxs-lookup"><span data-stu-id="a452d-106">(**Note:** This tutorial was written with PHP version 7.2.</span></span> <span data-ttu-id="a452d-107">Действия, описанные в этом руководстве, могут работать с другими версиями, но не тестировались.</span><span class="sxs-lookup"><span data-stu-id="a452d-107">The steps in this guide may work with other versions, but that has not been tested.)</span></span>
- <span data-ttu-id="a452d-108">[Композитор](https://getcomposer.org/) , установленный на компьютере разработчика.</span><span class="sxs-lookup"><span data-stu-id="a452d-108">[Composer](https://getcomposer.org/) installed on your development machine.</span></span>
- <span data-ttu-id="a452d-109">[Ларавел](https://laravel.com/) , установленный на компьютере разработчика.</span><span class="sxs-lookup"><span data-stu-id="a452d-109">[Laravel](https://laravel.com/) installed on your development machine.</span></span>
- <span data-ttu-id="a452d-110">Личная учетная запись Майкрософт с почтовым ящиком на Outlook.com или рабочей или учебной учетной записью Майкрософт.</span><span class="sxs-lookup"><span data-stu-id="a452d-110">Either a personal Microsoft account with a mailbox on Outlook.com, or a Microsoft work or school account.</span></span>

<span data-ttu-id="a452d-111">Если у вас нет учетной записи Майкрософт, у вас есть несколько вариантов для получения бесплатной учетной записи:</span><span class="sxs-lookup"><span data-stu-id="a452d-111">If you don't have a Microsoft account, there are a couple of options to get a free account:</span></span>

- <span data-ttu-id="a452d-112">Вы можете [зарегистрироваться для создания новой личной учетной записи Майкрософт](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span><span class="sxs-lookup"><span data-stu-id="a452d-112">You can [sign up for a new personal Microsoft account](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).</span></span>
- <span data-ttu-id="a452d-113">Вы можете [зарегистрироваться в программе для разработчиков office 365](https://developer.microsoft.com/office/dev-program) , чтобы получить бесплатную подписку на Office 365.</span><span class="sxs-lookup"><span data-stu-id="a452d-113">You can [sign up for the Office 365 Developer Program](https://developer.microsoft.com/office/dev-program) to get a free Office 365 subscription.</span></span>

## <a name="register-a-web-application-with-the-application-registration-portal"></a><span data-ttu-id="a452d-114">Регистрация веб-приложения с помощью портала регистрации приложений</span><span class="sxs-lookup"><span data-stu-id="a452d-114">Register a web application with the Application Registration Portal</span></span>

1. <span data-ttu-id="a452d-115">Откройте браузер и перейдите на [портал регистрации приложений](https://apps.dev.microsoft.com).</span><span class="sxs-lookup"><span data-stu-id="a452d-115">Open a browser and navigate to the [Application Registration Portal](https://apps.dev.microsoft.com).</span></span> <span data-ttu-id="a452d-116">Вход с использованием **личной учетной записи** (с учетной записью Майкрософт) или **рабочей или учебНой учетной записи**.</span><span class="sxs-lookup"><span data-stu-id="a452d-116">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="a452d-117">В верхней части страницы выберите **Добавить приложение** .</span><span class="sxs-lookup"><span data-stu-id="a452d-117">Select **Add an app** at the top of the page.</span></span>

    > <span data-ttu-id="a452d-118">**Примечание:** Если на странице отображается несколько кнопок **Добавить приложение** , выберите ту, которая соответствует списку **приложений** для конвергенции.</span><span class="sxs-lookup"><span data-stu-id="a452d-118">**Note:** If you see more than one **Add an app** button on the page, select the one that corresponds to the **Converged apps** list.</span></span>

1. <span data-ttu-id="a452d-119">На странице **Регистрация приложения** задайте для параметра **имя приложения** значение учебное **руководство по диаграмме PHP** и нажмите кнопку **создать**.</span><span class="sxs-lookup"><span data-stu-id="a452d-119">On the **Register your application** page, set the **Application Name** to **PHP Graph Tutorial** and select **Create**.</span></span>

    ![Снимок экрана: создание нового приложения на веб-сайте портала регистрации приложений](/tutorial/images/arp-create-app-01.png)

1. <span data-ttu-id="a452d-121">На странице **Регистрация учебника PHP Graph** в разделе **свойства** скопируйте **идентификатор приложения** так, как он понадобится позже.</span><span class="sxs-lookup"><span data-stu-id="a452d-121">On the **PHP Graph Tutorial Registration** page, under the **Properties** section, copy the **Application Id** as you will need it later.</span></span>

    ![Снимок экрана с ИДЕНТИФИКАТОРом только что созданного приложения](/tutorial/images/arp-create-app-02.png)

1. <span data-ttu-id="a452d-123">ПроКрутите список вниз до раздела **секреты приложения** .</span><span class="sxs-lookup"><span data-stu-id="a452d-123">Scroll down to the **Application Secrets** section.</span></span>

    1. <span data-ttu-id="a452d-124">Выберите **создать новый пароль**.</span><span class="sxs-lookup"><span data-stu-id="a452d-124">Select **Generate New Password**.</span></span>
    1. <span data-ttu-id="a452d-125">В диалоговом окне **новый пароль** в созданном пароле скопируйте содержимое поля так, как оно потребуется позже.</span><span class="sxs-lookup"><span data-stu-id="a452d-125">In the **New password generated** dialog, copy the contents of the box as you will need it later.</span></span>

        > <span data-ttu-id="a452d-126">**Важно!** Этот пароль никогда не отображается еще раз, поэтому убедитесь, что вы хотите скопировать его сейчас.</span><span class="sxs-lookup"><span data-stu-id="a452d-126">**Important:** This password is never shown again, so make sure you copy it now.</span></span>

    ![Снимок экрана с новым паролем приложения](/tutorial/images/arp-create-app-03.png)

1. <span data-ttu-id="a452d-128">ПроКрутите окно вниз до раздела **платформы** .</span><span class="sxs-lookup"><span data-stu-id="a452d-128">Scroll down to the **Platforms** section.</span></span>

    1. <span data-ttu-id="a452d-129">Нажмите кнопку **Добавить платформу**.</span><span class="sxs-lookup"><span data-stu-id="a452d-129">Select **Add Platform**.</span></span>
    1. <span data-ttu-id="a452d-130">В диалоговом окне **Добавление платформы** выберите **веб**.</span><span class="sxs-lookup"><span data-stu-id="a452d-130">In the **Add Platform** dialog, select **Web**.</span></span>

        ![Снимок экрана: создание платформы для приложения](/tutorial/images/arp-create-app-04.png)

    1. <span data-ttu-id="a452d-132">В поле **веб-** платформа введите URL-адрес `http://localhost:8000/callback` для **URL-адресов перенаправления**.</span><span class="sxs-lookup"><span data-stu-id="a452d-132">In the **Web** platform box, enter the URL `http://localhost:8000/callback` for the **Redirect URLs**.</span></span>

        ![Снимок экрана: недавно добавленная веб-платформа для приложения](/tutorial/images/arp-create-app-05.png)

1. <span data-ttu-id="a452d-134">ПроКрутите страницу вниз и выберите команду **сохранить**.</span><span class="sxs-lookup"><span data-stu-id="a452d-134">Scroll to the bottom of the page and select **Save**.</span></span>

## <a name="configure-the-sample"></a><span data-ttu-id="a452d-135">Настройка примера</span><span class="sxs-lookup"><span data-stu-id="a452d-135">Configure the sample</span></span>

1. <span data-ttu-id="a452d-136">Переименуйте `.env.example` файл в `.env`.</span><span class="sxs-lookup"><span data-stu-id="a452d-136">Rename the `.env.example` file to `.env`.</span></span>
1. <span data-ttu-id="a452d-137">Измените `.env` файл и внесите следующие изменения.</span><span class="sxs-lookup"><span data-stu-id="a452d-137">Edit the `.env` file and make the following changes.</span></span>
    1. <span data-ttu-id="a452d-138">Замените `YOUR_APP_ID_HERE` идентификатором **приложения** , полученНым на портале регистрации приложений.</span><span class="sxs-lookup"><span data-stu-id="a452d-138">Replace `YOUR_APP_ID_HERE` with the **Application Id** you got from the App Registration Portal.</span></span>
    1. <span data-ttu-id="a452d-139">Замените `YOUR_APP_PASSWORD_HERE` паролем, полученНым на портале регистрации приложений.</span><span class="sxs-lookup"><span data-stu-id="a452d-139">Replace `YOUR_APP_PASSWORD_HERE` with the password you got from the App Registration Portal.</span></span>
1. <span data-ttu-id="a452d-140">В интерфейсе командной строки (CLI) перейдите к этому каталогу и выполните следующую команду, чтобы установить требования.</span><span class="sxs-lookup"><span data-stu-id="a452d-140">In your command-line interface (CLI), navigate to this directory and run the following command to install requirements.</span></span>

    ```Shell
    composer install
    ```
1. <span data-ttu-id="a452d-141">В интерфейсе командной строки (CLI) выполните приведенную ниже команду, чтобы создать ключ приложения.</span><span class="sxs-lookup"><span data-stu-id="a452d-141">In your command-line interface (CLI), run the following command to generate an application key.</span></span>

    ```Shell
    php artisan key:generate
    ```

## <a name="run-the-sample"></a><span data-ttu-id="a452d-142">Запуск приложения</span><span class="sxs-lookup"><span data-stu-id="a452d-142">Run the sample</span></span>

1. <span data-ttu-id="a452d-143">Выполните следующую команду в командной панели CLI, чтобы запустить приложение.</span><span class="sxs-lookup"><span data-stu-id="a452d-143">Run the following command in your CLI to start the application.</span></span>

    ```Shell
    php artisan serve
    ```

1. <span data-ttu-id="a452d-144">Откройте браузер и перейдите по адресу `http://localhost:8000`.</span><span class="sxs-lookup"><span data-stu-id="a452d-144">Open a browser and browse to `http://localhost:8000`.</span></span>