<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="f95ff-101">В этом упражнении вы создадите регистрацию нового веб-приложения Azure AD с помощью центра администрирования Azure Active Directory.</span><span class="sxs-lookup"><span data-stu-id="f95ff-101">In this exercise, you will create a new Azure AD web application registration using the Azure Active Directory admin center.</span></span>

1. <span data-ttu-id="f95ff-102">Откройте браузер и перейдите в [центр администрирования Azure Active Directory](https://aad.portal.azure.com).</span><span class="sxs-lookup"><span data-stu-id="f95ff-102">Open a browser and navigate to the [Azure Active Directory admin center](https://aad.portal.azure.com).</span></span> <span data-ttu-id="f95ff-103">Вход с использованием **личной учетной записи** (с учетной записью Майкрософт) или **рабочей или учебНой учетной записи**.</span><span class="sxs-lookup"><span data-stu-id="f95ff-103">Login using a **personal account** (aka: Microsoft Account) or **Work or School Account**.</span></span>

1. <span data-ttu-id="f95ff-104">Выберите **Azure Active Directory** в левой панели навигации, а затем выберите **Регистрация приложений (Предварительная версия)** в разделе **Управление**.</span><span class="sxs-lookup"><span data-stu-id="f95ff-104">Select **Azure Active Directory** in the left-hand navigation, then select **App registrations (Preview)** under **Manage**.</span></span>

    ![<span data-ttu-id="f95ff-105">Снимок экрана с регистрациями приложений</span><span class="sxs-lookup"><span data-stu-id="f95ff-105">A screenshot of the App registrations</span></span> ](./images/aad-portal-app-registrations.png)

1. <span data-ttu-id="f95ff-106">Нажмите кнопку **создать регистрацию**.</span><span class="sxs-lookup"><span data-stu-id="f95ff-106">Select **New registration**.</span></span> <span data-ttu-id="f95ff-107">На странице **Регистрация приложения** задайте указанные ниже значения.</span><span class="sxs-lookup"><span data-stu-id="f95ff-107">On the **Register an application** page, set the values as follows.</span></span>

    - <span data-ttu-id="f95ff-108">Задайте \*\*\*\* для `PHP Graph Tutorial`параметра Name значение.</span><span class="sxs-lookup"><span data-stu-id="f95ff-108">Set **Name** to `PHP Graph Tutorial`.</span></span>
    - <span data-ttu-id="f95ff-109">Установите **Поддерживаемые типы учетных** записей для **учетных записей в любом организационном каталоге и личных учетНых записях Майкрософт**.</span><span class="sxs-lookup"><span data-stu-id="f95ff-109">Set **Supported account types** to **Accounts in any organizational directory and personal Microsoft accounts**.</span></span>
    - <span data-ttu-id="f95ff-110">В разделе **URI перенаправления**установите первый раскрывающийся список `Web` и присвойте ему значение `http://localhost:8000/callback`.</span><span class="sxs-lookup"><span data-stu-id="f95ff-110">Under **Redirect URI**, set the first drop-down to `Web` and set the value to `http://localhost:8000/callback`.</span></span>

    ![Снимок страницы "регистрация приложения"](./images/aad-register-an-app.png)

1. <span data-ttu-id="f95ff-112">Выберите **регистр**.</span><span class="sxs-lookup"><span data-stu-id="f95ff-112">Choose **Register**.</span></span> <span data-ttu-id="f95ff-113">На странице **учебника по диаграмме PHP** СКОПИРУЙТЕ значение **идентификатора Application (Client)** и сохраните его, он понадобится на следующем шаге.</span><span class="sxs-lookup"><span data-stu-id="f95ff-113">On the **PHP Graph Tutorial** page, copy the value of the **Application (client) ID** and save it, you will need it in the next step.</span></span>

    ![Снимок экрана с ИДЕНТИФИКАТОРом приложения для новой регистрации приложения](./images/aad-application-id.png)

1. <span data-ttu-id="f95ff-115">Выберите **Сертификаты _амп_ секреты** в разделе **Управление**.</span><span class="sxs-lookup"><span data-stu-id="f95ff-115">Select **Certificates & secrets** under **Manage**.</span></span> <span data-ttu-id="f95ff-116">Нажмите кнопку **создать секрет клиента** .</span><span class="sxs-lookup"><span data-stu-id="f95ff-116">Select the **New client secret** button.</span></span> <span data-ttu-id="f95ff-117">Введите значение в поле **Описание** и выберите один из вариантов исТечения **срока действия** и нажмите кнопку **добавить**.</span><span class="sxs-lookup"><span data-stu-id="f95ff-117">Enter a value in **Description** and select one of the options for **Expires** and choose **Add**.</span></span>

    ![Снимок экрана: диалоговое окно добавления секрета клиента](./images/aad-new-client-secret.png)

1. <span data-ttu-id="f95ff-119">Скопируйте значение секрета клиента, прежде чем покинуть эту страницу.</span><span class="sxs-lookup"><span data-stu-id="f95ff-119">Copy the client secret value before you leave this page.</span></span> <span data-ttu-id="f95ff-120">Это потребуется на следующем этапе.</span><span class="sxs-lookup"><span data-stu-id="f95ff-120">You will need it in the next step.</span></span>

    > [!IMPORTANT]
    > <span data-ttu-id="f95ff-121">Этот секрет клиента никогда не отображается еще раз, поэтому убедитесь, что вы хотите скопировать его сейчас.</span><span class="sxs-lookup"><span data-stu-id="f95ff-121">This client secret is never shown again, so make sure you copy it now.</span></span>

    ![Снимок экрана с недавно добавленным секретом клиента](./images/aad-copy-client-secret.png)