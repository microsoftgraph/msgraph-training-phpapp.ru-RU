<!-- markdownlint-disable MD002 MD041 -->

В этом руководстве рассказывается, как создать веб-приложение PHP, использующее API Microsoft Graph для получения сведений календаря для пользователя.

> [!TIP]
> Если вы предпочитаете просто скачать завершенный учебник, вы можете скачать его двумя способами.
>
> - Скачайте [краткое начало PHP,](https://developer.microsoft.com/graph/quick-start?platform=option-php) чтобы получить рабочий код в минутах.
> - Скачайте или клонировать [репозиторий GitHub.](https://github.com/microsoftgraph/msgraph-training-phpapp)

## <a name="prerequisites"></a>Предварительные условия

Прежде чем приступить к этому учебнику, на компьютере разработчика должны быть установлены [PHP,](http://php.net/downloads.php) [Composer](https://getcomposer.org/)и [Laravel.](https://laravel.com/)

У вас также должна быть личная учетная запись Майкрософт с почтовым ящиком на Outlook.com или учетная запись Майкрософт для работы или учебного заведения. Если у вас нет учетной записи Майкрософт, существует несколько вариантов получения бесплатной учетной записи:

- Вы можете [зарегистрироваться для новой личной учетной записи Майкрософт.](https://signup.live.com/signup?wa=wsignin1.0&rpsnv=12&ct=1454618383&rver=6.4.6456.0&wp=MBI_SSL_SHARED&wreply=https://mail.live.com/default.aspx&id=64855&cbcxt=mai&bk=1454618383&uiflavor=web&uaid=b213a65b4fdc484382b6622b3ecaa547&mkt=E-US&lc=1033&lic=1)
- Вы можете зарегистрироваться в программе для разработчиков [Office 365,](https://developer.microsoft.com/office/dev-program) чтобы получить бесплатную подписку на Office 365.

> [!NOTE]
> Это руководство было написано с помощью PHP версии 8.0.1, Composer версии 2.0.8 и установщика Laravel версии 4.1.1. Действия в этом руководстве могут работать с другими версиями, но не были протестированы.

## <a name="feedback"></a>Отзывы

Поделитесь с этим учебником отзывами в [репозитории GitHub.](https://github.com/microsoftgraph/msgraph-training-phpapp)
