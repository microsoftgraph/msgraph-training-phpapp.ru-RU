<!-- markdownlint-disable MD002 MD041 -->

<span data-ttu-id="41e3c-101">В этом руководстве рассказывается, как создать веб-приложение PHP, использующее API Microsoft Graph для получения сведений о календаре для пользователя.</span><span class="sxs-lookup"><span data-stu-id="41e3c-101">This tutorial teaches you how to build a PHP web app that uses the Microsoft Graph API to retrieve calendar information for a user.</span></span>

> [!TIP]
> <span data-ttu-id="41e3c-102">Если вы хотите просто скачать заполненный учебник, можно скачать его двумя способами.</span><span class="sxs-lookup"><span data-stu-id="41e3c-102">If you prefer to just download the completed tutorial, you can download it in two ways.</span></span>
>
> - <span data-ttu-id="41e3c-103">Скачайте [Краткое руководство по PHP](https://developer.microsoft.com/graph/quick-start?platform=option-php) , чтобы получить рабочий код в минутах.</span><span class="sxs-lookup"><span data-stu-id="41e3c-103">Download the [PHP quick start](https://developer.microsoft.com/graph/quick-start?platform=option-php) to get working code in minutes.</span></span>
> - <span data-ttu-id="41e3c-104">Скачайте или скопируйте [репозиторий GitHub](https://github.com/microsoftgraph/msgraph-training-phpapp).</span><span class="sxs-lookup"><span data-stu-id="41e3c-104">Download or clone the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-phpapp).</span></span>

## <a name="prerequisites"></a><span data-ttu-id="41e3c-105">Необходимые компоненты</span><span class="sxs-lookup"><span data-stu-id="41e3c-105">Prerequisites</span></span>

<span data-ttu-id="41e3c-106">Прежде чем приступить к работе с этим руководством, на компьютере для разработки должен быть установлен [PHP](http://php.net/downloads.php), [Composer](https://getcomposer.org/)и [ларавел](https://laravel.com/) .</span><span class="sxs-lookup"><span data-stu-id="41e3c-106">Before you start this tutorial, you should have [PHP](http://php.net/downloads.php), [Composer](https://getcomposer.org/), and [Laravel](https://laravel.com/) installed on your development machine.</span></span>

> [!NOTE]
> <span data-ttu-id="41e3c-107">Это руководство было написано с PHP версии 7,2.</span><span class="sxs-lookup"><span data-stu-id="41e3c-107">This tutorial was written with PHP version 7.2.</span></span> <span data-ttu-id="41e3c-108">Действия, описанные в этом руководстве, могут работать с другими версиями, но не тестировались.</span><span class="sxs-lookup"><span data-stu-id="41e3c-108">The steps in this guide may work with other versions, but that has not been tested.</span></span>

## <a name="feedback"></a><span data-ttu-id="41e3c-109">Отзывы</span><span class="sxs-lookup"><span data-stu-id="41e3c-109">Feedback</span></span>

<span data-ttu-id="41e3c-110">Сообщите о нем в [репозиторий GitHub](https://github.com/microsoftgraph/msgraph-training-phpapp).</span><span class="sxs-lookup"><span data-stu-id="41e3c-110">Please provide any feedback on this tutorial in the [GitHub repository](https://github.com/microsoftgraph/msgraph-training-phpapp).</span></span>