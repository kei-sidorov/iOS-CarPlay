![](https://user-images.githubusercontent.com/76877122/143779514-75ad23b9-ee06-4c88-8a55-92dae9f7ef04.png)

## Серия статей

* [iOS CarPlay｜Совместимость с UIScene](https://github.com/teney97/iOS-CarPlay/blob/main/iOS%20CarPlay｜兼容%20UIScene)
* [iOS CarPlay｜Добавляем поддержку CarPlay в аудио-приложение](https://github.com/teney97/iOS-CarPlay/blob/main/iOS%20CarPlay%EF%BD%9C%E8%AE%A9%E4%BD%A0%E7%9A%84%E9%9F%B3%E9%A2%91%20App%20%E6%94%AF%E6%8C%81%20CarPlay.md)
* [iOS CarPlay｜Часто задаваемые вопросы](https://github.com/teney97/iOS-CarPlay/blob/main/iOS%20CarPlay%EF%BD%9C%E8%AE%A9%E4%BD%A0%E7%9A%84%E9%9F%B3%E9%A2%91%20App%20%E6%94%AF%E6%8C%81%20CarPlay.md)
* [iOS CarPlay｜Заметки по WWDC](https://github.com/teney97/iOS-CarPlay/blob/main/iOS%20CarPlay%EF%BD%9CWWDC%20%E7%AC%94%E8%AE%B0.md)
* iOS CarPlay｜Разработка с использованием MediaPlayer framework
* [iOS CarPlay｜WWDC22 10016 - Расширьте возможности своего приложения с помощью CarPlay](https://github.com/teney97/iOS-CarPlay/blob/main/iOS%20CarPlay｜WWDC22%2010016%20-%20通过%20CarPlay%20让你的%20App%20发挥更大的作用.md)

## Предисловие

Автор отвечал за разработку CarPlay-приложения Babybus (宝宝巴士) и хочет поделиться опытом разработки. Из этой статьи вы узнаете:

* что такое CarPlay и какие типы приложений он поддерживает;
* как сделать проект совместимым с UIScene, то есть перейти от традиционных UIWindow и AppDelegate к SceneDelegate;
* как шаг за шагом разработать аудио-приложение для CarPlay.

## Что такое CarPlay

CarPlay — это автомобильная система от Apple, которая работает совместно с iPhone (iPad не поддерживается). Ранее она называлась iOS in the Car, а в 2014 году была переименована в CarPlay. CarPlay — это более умный и безопасный способ пользоваться iPhone за рулём.

Проще говоря, если ваш автомобиль поддерживает CarPlay, то при подключении iPhone к автомобилю (по кабелю, через Bluetooth или Wi-Fi) экран автомобиля автоматически переключается на CarPlay, а все приложения на iPhone, поддерживающие CarPlay, автоматически отображаются в CarPlay. В настройках iPhone (Основные > CarPlay) можно скрыть отдельные приложения или изменить их порядок. Apple использует единый дизайн пользовательского интерфейса CarPlay-приложений, а содержимое предоставляет само приложение.

Управлять CarPlay можно тремя основными способами: через Siri, сенсорный экран и физические кнопки.

## Какие приложения и функции поддерживает CarPlay

* Аудио-приложения могут предоставлять музыку, новости, подкасты и т. д.
* Коммуникационные приложения (Messaging, VoIP calling) могут отправлять и принимать сообщения, а также работать совместно с Siri.
* Навигационные приложения могут предоставлять подробные карты, поиск пунктов назначения, пошаговые маршруты и уведомления для пользователя.
* Приложения автопроизводителей могут предоставлять управление и отображение данных, специфичных для автомобиля, позволяя водителю оставаться на связи, не покидая CarPlay.
* Новое в iOS 14: поддержка приложений для зарядки электромобилей, парковки и заказа фастфуда. Кроме того, все CarPlay-приложения могут использовать CarPlay framework для единообразного дизайна, оптимизированного для использования в автомобиле.

## Обзор процесса разработки CarPlay-приложения

1. Сначала нужно определить, подходит ли ваше приложение для CarPlay, затем на сайте для разработчиков запросить соответствующий CarPlay entitlement для вашего типа приложения и настроить проект. Только после этого проект сможет работать с CarPlay Simulator. Поэтому, если вы планируете разрабатывать CarPlay-приложение, лучше подать заявку на разрешение заранее: проверка в Apple занимает время. За это время можно изучить документацию, а когда разрешение будет получено, можно отлаживать приложение в CarPlay Simulator. Документация: [Apple｜Запрос CarPlay entitlement](https://developer.Apple.com/documentation/carplay/requesting_the_carplay_entitlements?language=objc).

2. Начиная с iOS 14 для разработки аудио-приложений для CarPlay можно использовать CarPlay framework (для навигационных приложений он доступен уже в iOS 12). Он предоставляет набор UI-шаблонов, позволяющих разработчикам настраивать интерфейс. Если нужна совместимость с iOS 13 и более ранними версиями, придётся использовать MediaPlayer framework, который работает и на старых версиях. Таким образом, если вашему приложению нужно использовать CarPlay framework на iOS 14 и выше и при этом поддерживать iOS 13 и ниже, придётся поддерживать две версии кода, и объём работы может вырасти почти вдвое. Автор поддержал только iOS 14 и выше, поэтому в этой статье подробно разбирается разработка с CarPlay framework. Если вы хотите поддерживать и старые версии, посмотрите также заметки автора по сессиям «WWDC17 - Добавьте поддержку CarPlay в своё приложение» и «WWDC18 - Аудио- и навигационные приложения CarPlay».

3. Для разработки CarPlay-приложения с помощью CarPlay framework на iOS 14 и выше необходимо использовать UIScene (UIScene появился в iOS 13 и предназначен для создания многооконных приложений), поэтому проект должен перейти от традиционных UIWindow и AppDelegate к SceneDelegate. Если ваш проект уже совместим с UIScene, этот шаг можно пропустить; если нет — обратите внимание на некоторые моменты, о которых говорится в этой статье.

4. CarPlay-приложение является частью iPhone-приложения, они работают в одном процессе.

   * Если первым запущено CarPlay-приложение, система запустит ваше iPhone-приложение в фоновом режиме;
   * Если процесс iPhone-приложения завершён, CarPlay-приложение закроется (в UIScene это отключение сцены);
   * Если закрыть CarPlay-приложение (в UIScene это отключение сцены), процесс iPhone-приложения завершён не будет. (По-видимому, все CarPlay-приложения закрываются только при выходе из CarPlay целиком, закрыть отдельное CarPlay-приложение нельзя);

   В iOS 13 Apple улучшила CarPlay: CarPlay-приложение и iPhone-приложение могут находиться одно в фоне, а другое на переднем плане. До iOS 13 CarPlay-приложение и iPhone-приложение были жёстко связаны и могли находиться только одновременно на переднем плане или в фоне, что ухудшало пользовательский опыт. Например, при использовании навигации в CarPlay на телефоне нельзя было выполнять другие действия, иначе навигация прерывалась.

5. Автор разрабатывал аудио-приложение для CarPlay и не изучал другие типы приложений, но общий процесс разработки должен быть примерно одинаковым. Разработка аудио-приложения для CarPlay начинается с точки входа CPTemplateApplicationSceneDelegate: в ней строится UI и заполняется данными. Пользовательский интерфейс CarPlay-приложения относительно жёстко задан, но с CarPlay framework Apple позволяет гораздо больше его настраивать. После подключения к головному устройству автомобиля звук воспроизводится через динамики автомобиля. Независимо от того, используете ли вы CarPlay framework или MediaPlayer framework, информация об аудио на экране воспроизведения и обработка событий удалённого управления воспроизведением осуществляются через MPNowPlayingInfoCenter и MPRemoteCommandCenter. Разница лишь в том, что в CarPlay framework часть событий удалённого управления, например режим воспроизведения и скорость воспроизведения, обрабатывается через handler у CPNowPlayingButton. Если ваше приложение аудио-типа, оно, скорее всего, уже поддерживает эти функции, поскольку информация о воспроизведении и элементы управления на экране блокировки iPhone и в Пункте управления тоже предоставляются через них. Поэтому для CarPlay нам нужны лишь оптимизация или расширение функциональности.

   * Установка и обновление nowPlayingInfo у MPNowPlayingInfoCenter, которое содержит информацию о текущем аудио: название, автор, длительность и т. д.;
   * Обработка событий MPRemoteCommandCenter — реакция на события удалённого управления воспроизведением: воспроизведение, пауза, переключение трека и т. д.;

   Помимо этих двух пунктов, вам, возможно, понадобится:

   * Установка и обновление playbackState у MPNowPlayingInfoCenter, чтобы обновлять состояние воспроизведения, отображаемое в CarPlay-приложении: воспроизведение/пауза;
   * Установка и обновление changeRepeatModeCommand.currentRepeatType у MPRemoteCommandCenter, чтобы обновлять режим воспроизведения, отображаемый в CarPlay-приложении: повтор списка/повтор одного трека

   Даже если ваше приложение пока не планирует поддерживать CarPlay, правильная реализация MPNowPlayingInfoCenter и MPRemoteCommandCenter позволит ему отображаться в CarPlay как приложение «Сейчас играет».

6. Изучите, какие функции и элементы UI поддерживает CarPlay framework, и помогите своему PM и дизайнеру подготовить требования, прототипы и дизайн. Изучите работу с UI: интерфейс по сути состоит из Template и Item, наиболее часто используются CPTabBarTemplate, CPListTemplate, CPListItem, CPListImageRowItem и т. д.

7. Дальше — собственно разработка! Здесь опущено ... слов.

8. Наконец, тестирование в реальных условиях (в автомобиле). В документе [Apple｜Запуск и отладка CarPlay-приложения в CarPlay Simulator](https://developer.Apple.com/documentation/carplay/using_the_carplay_simulator?language=objc) перечислены функции, которые нельзя протестировать в CarPlay Simulator. Кроме того, Apple рекомендует больше тестировать работу при слабой сети и при отсутствии сети, поскольку во время поездки можно проезжать участки или районы с плохим покрытием.

9. Можно поэкспериментировать с примером аудио-приложения для CarPlay от Apple: [Apple｜CarPlay Music App](https://developer.Apple.com/documentation/carplay/integrating_carplay_with_your_music_App?language=objc).

## Дополнительные материалы

### WWDC

* [WWDC16｜Разработка системы CarPlay — часть 1](https://developer.Apple.com/wwdc16/722)
* [WWDC16｜Разработка системы CarPlay — часть 2](https://developer.Apple.com/wwdc16/723)
* [WWDC17｜Разработка беспроводного CarPlay](https://developer.Apple.com/wwdc17/717)
* [WWDC17｜Добавьте поддержку CarPlay в своё приложение](https://developer.Apple.com/wwdc17/719)
* [WWDC18｜Аудио- и навигационные приложения CarPlay](https://developer.Apple.com/wwdc18/213)
  * [WWDC 2018：CarPlay в аудио- и навигационных приложениях](https://juejin.cn/post/6844903619192422413)
* [WWDC19｜Улучшения CarPlay](https://developer.Apple.com/wwdc19/252)
* [WWDC20｜Ускорьте работу своего приложения с помощью CarPlay](https://developer.Apple.com/wwdc20/10635)
  * [WWDC20 Inside｜WWDC20 10635 - Ускорьте работу своего приложения с помощью CarPlay](<https://xiaozhuanlan.com/topic/7620814593>)

### Документация

* [CarPlay｜Введение](https://www.Apple.com.cn/ios/carplay/)
* [CarPlay｜Главная страница документации](https://developer.Apple.com/carplay/)
* [CarPlay｜Руководство по дизайну](https://developer.Apple.com/design/human-interface-guidelines/carplay/overview/introduction/)
* [CarPlay｜Документация для разработчиков](https://developer.Apple.com/documentation/carplay?language=objc)
  * [Запрос CarPlay entitlement](https://developer.Apple.com/documentation/carplay/requesting_the_carplay_entitlements?language=objc)
  * [Запуск и отладка CarPlay-приложения в CarPlay Simulator](https://developer.Apple.com/documentation/carplay/using_the_carplay_simulator?language=objc) (сначала нужно выполнить предыдущий шаг, иначе CarPlay Simulator не заработает и ваше приложение не появится на главном экране CarPlay. На Mac с M1 CarPlay Simulator может не работать)
  * [Отображение контента в вашем CarPlay-приложении](https://developer.Apple.com/documentation/carplay/displaying_content_in_carplay?language=objc)
  * [Поддержка iOS 13 и более ранних версий iOS](https://developer.Apple.com/documentation/carplay/supporting_previous_versions_of_ios?language=objc)
* [CarPlay｜Руководство по программированию приложений](https://developer.Apple.com/carplay/documentation/CarPlay-App-Programming-Guide.pdf)
  * Машинный перевод Sogou (с английского на китайский): https://github.com/teney97/iOS-CarPlay/blob/main/Content/CarPlay-App-Programming-Guide【搜狗文档翻译_译文_英译中】.pdf
* [CarPlay｜Поддерживаемые модели автомобилей](https://www.Apple.com.cn/ios/carplay/available-models/)
* [CarPlay｜Пример Apple CarPlay Music App](https://developer.Apple.com/documentation/carplay/integrating_carplay_with_your_music_App?language=objc)
  * CarPlay Music — пример аудио-приложения для CarPlay от Apple, демонстрирующий, как отображать собственный UI в CarPlay. CarPlay Music использует CarPlay framework и интегрируется через реализацию CPNowPlayingTemplate и CPListTemplate. Пример приложения содержит экран журнала, который помогает разобраться в жизненном цикле CarPlay-приложения и в контроллере музыки.


### Прочее

* [Подробно о CarPlay в iOS 13 от Apple: новый дизайн UI + отдельное представление приложения ](https://www.sohu.com/a/336034138_120178230)
