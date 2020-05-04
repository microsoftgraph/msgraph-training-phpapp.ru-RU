<!-- markdownlint-disable MD002 MD041 -->

В этом руководстве рассказывается, как создать веб-приложение PHP, использующее API Microsoft Graph для получения сведений о календаре для пользователя.

> [!TIP]
> Если вы хотите просто скачать заполненный учебник, можно скачать его двумя способами.
>
> - Скачайте [Краткое руководство по PHP](https://developer.microsoft.com/graph/quick-start?platform=option-php) , чтобы получить рабочий код в минутах.
> - Скачайте или скопируйте [репозиторий GitHub](https://github.com/microsoftgraph/msgraph-training-phpapp).

## <a name="prerequisites"></a>Предварительные требования

Прежде чем приступить к работе с этим руководством, на компьютере для разработки должен быть установлен [PHP](http://php.net/downloads.php), [Composer](https://getcomposer.org/)и [ларавел](https://laravel.com/) .

Кроме того, у вас также должна быть личная учетная запись Майкрософт с почтовым ящиком на Outlook.com или рабочей или учебной учетной записью Майкрософт. Если у вас нет учетной записи Майкрософт, у вас есть несколько вариантов для получения бесплатной учетной записи:

- Вы можете [зарегистрироваться для создания новой личной учетной записи Майкрософт](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1).
- Вы можете [зарегистрироваться в программе для разработчиков office 365](https://developer.microsoft.com/office/dev-program) , чтобы получить бесплатную подписку на Office 365.

> [!NOTE]
> Это руководство было написано с использованием PHP версии 7.4.4, composer версии 1.10.1 и установщика Ларавел версии 3.0.1. Действия, описанные в этом руководстве, могут работать с другими версиями, но не тестировались.

## <a name="feedback"></a>Отзывы

Сообщите о нем в [репозиторий GitHub](https://github.com/microsoftgraph/msgraph-training-phpapp).
