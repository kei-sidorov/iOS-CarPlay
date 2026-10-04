## WWDC18 - CarPlay: аудио- и навигационные приложения для автомобиля

**Ссылка**: https://developer.apple.com/wwdc18/213

**Длительность**: 38 минут

**Обзор**: Узнайте, как обновить аудио- или навигационное приложение для поддержки CarPlay. Приложения в CarPlay оптимизированы для использования в автомобиле и автоматически адаптируются к доступному экрану и элементам управления автомобиля. Аудиоприложения могут воспроизводить музыку, новости, подкасты и многое другое. Благодаря новому фреймворку CarPlay навигационные приложения могут показывать подробные карты, выполнять поиск пунктов назначения, вести пошаговую навигацию и отправлять уведомления пользователю.

**Источник**: https://juejin.cn/post/6844903619192422413

**Резюме**: Основное содержание этой сессии сводится к двум пунктам. Поскольку автор разрабатывает аудиоприложение для CarPlay, материал о навигационных приложениях он не изучал.

* Улучшение производительности и оптимизация аудиоприложений CarPlay
* Новый фреймворк CarPlay для навигационных приложений

**Как добавить в аудиоприложение поддержку системы CarPlay**

* Template based
* Works with all CarPlay systems
* Uses existing MediaPlayer APIs

Шаблоны, которые использует CarPlay, абстрагируют и скрывают различия и сложности CarPlay-систем, такие как способы ввода и размеры экрана, поэтому вашему аудиоприложению достаточно предоставить данные и отобразить их в CarPlay. Обычно это делается через `tableview` или `tabs`, в зависимости от того, как вы хотите представить свои данные.

Вам нужно думать о том, как передавать данные в CarPlay. Если вы уже разрабатываете аудиоприложение на существующих API, то, скорее всего, они вам хорошо знакомы. Ниже приведены 3 API, которые нужно знать, чтобы выпустить приложение для системы CarPlay; каждый из них подробно разбирался в видео WWDC прошлого года.

Для просмотра контента в CarPlay используется API `MPPlayableContent`: у него есть `dataSource` и `delegate`, так что ваше аудиоприложение может передавать данные в CarPlay, а делегат получает обратные вызовы. API `MPNowPlayingInfoCenter` и `MPRemoteCommandCenter` используются для реализации функции «Сейчас играет».

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1630374246223-1cad5cf8-1c87-4479-882d-aa337d98532b.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

Следующий код содержит минимум действий, необходимых для поддержки CarPlay в аудиоприложении:

```swift
func application(
_ application: UIApplication, didFinishLaunchingWithOptions launchOptions:
[UIApplicationLaunchOptionsKey: Any]?) -> Bool {
    
    // Set up data source and delegate
    MPPlayableContentManager.shared().dataSource = SrirockaContentManager.shared
    MPPlayableContentManager.shared().delegate = SrirockaContentManager.shared
    
    // Set Now Playing metadata in MPNowPlayingInfoCenter
    let nowPlayingInfo: [String: Any] = [:]
    MPNowPlayingInfoCenter.default().nowPlayingInfo = nowPlayingInfo
    
    // Respond to remote command events
    let commandCenter = MPRemoteCommandCenter.shared()
    commandCenter.playCommand.isEnabled = true
    commandCenter.playCommand.addTarget { _ in .success }
}
```

**Улучшения производительности и оптимизации в iOS12**

* Improved performance in `MPPlayableContent`
* Faster startup sequence 
* Smoother animations 
* Better communication to your app

Оптимизирован API `MPPlayableContent`: улучшена производительность запроса data source и делегата, при этом менять код не нужно. Ускорен запуск и добавлены более плавные анимации. Кроме того, при любом изменении контента вашего аудиоприложения в CarPlay система лучше взаимодействует с приложением, чтобы предугадать, что пользователь, вероятно, захочет воспроизвести или просмотреть в CarPlay.

**Best Practices**

Давайте подробнее посмотрим, как оптимизировать аудиоприложение:

* Call `reloadData()` only when needed
  * Это API из `MPPlayableContent`, который помогает лучше оптимизировать ваше аудиоприложение. Вызывать `reloadData()` следует только при необходимости: это очень дорогая операция, снижающая отзывчивость приложения.
* Use `beginUpdates()` and `endUpdates()`
  * Если же нужно обновить лишь часть контента, оберните изменения в вызовы этих двух API, чтобы контент обновился корректно.
* Keep an internal representation of the data source to optimize performance
  * Вызовы `MPPlayableContent` сейчас асинхронны, когда мы запрашиваем данные у вашего приложения. Поэтому где-то в приложении нужно хранить внутреннее представление данных или кэш информации, чтобы при запросе контента вы могли быстро отдать данные и сохранить отзывчивость приложения.

**Don’t Miss a Beat!**

Account for these common scenarios

* Screen locked with passcode
* Unreliable network connectivity

Теперь обсудим несколько способов «ещё больше улучшить производительность аудиоприложения в CarPlay»:

Плейлисты загружаются с задержками, а если приложение не отдаёт контент вовремя, CarPlay может завершить загрузку по тайм-ауту. Что может быть причиной?

* Пользователи CarPlay часто ездят там, где нет быстрого соединения, или с заблокированным экраном телефона, а для разблокировки нужен пароль. Если политика защиты данных вашего приложения требует, чтобы телефон был разблокирован, вы не сможете получить данные приложения, и в итоге CarPlay завершит ожидание по тайм-ауту. Поэтому, если ваши данные доступны только при разблокированном телефоне, нужно пересмотреть политику защиты данных приложения.
* Пользователи CarPlay могут ездить там, где сеть слабая или отсутствует; разные системы CarPlay предоставляют разные сервисы передачи данных, и вам нужно протестировать работу без постоянного Wi-Fi-соединения.

![image.png](https://cdn.nlark.com/yuque/0/2021/png/12376889/1630376442054-bafd9ea7-678c-48b2-86b8-5d2b260313f4.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

Как оптимизировать?

Используйте `beginLoadingChildItems()` для начала загрузки контента.

API `beginLoadingChildItems` служит для запуска получения контента и вызывается всякий раз, когда любой из ваших index path отображается на экране автомобиля CarPlay (можно сравнить с методом `cellForRow` у `tableview`). Когда пользователь прокручивает `tableview` или переключает `tabs`, `beginLoadingChildItems` вызывается для каждого отдельного index path на экране. Так у приложения появляется возможность начать загрузку до того, как пользователь фактически выберет контент. К моменту выбора запрос в сеть уже идёт либо данные уже готовы.

```swift
func beginLoadingChildItems(at indexPath: IndexPath,
                            completionHandler: @escaping (Error?) -> Void) {
    if indexPath.indexAtPosition(2) == 0 {
        // Start fetching content that requires a network connection or needs some setup
        startProcessingHeatingHabaneros()
        
    }
    ...
    completionHandler(nil)
}
```

Ниже инженер Apple приводит сценарий, с которым вы можете столкнуться при разработке приложения для CarPlay.

Если вашему аудиоприложению для показа контента требуется вход в аккаунт, то в CarPlay впечатления будут очень плохими: пользователь не сможет взаимодействовать с приложением. Нужно обеспечить хоть какой-то доступный без входа в аккаунт сценарий использования приложения в CarPlay, чтобы улучшить пользовательский опыт. 

![](https://cdn.nlark.com/yuque/0/2021/png/12376889/1630378644753-6e95dfd9-dd93-4dfd-bbb3-a77ae77e92c4.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

**Greatest Hits**

* Use `MPPlayableContent` to populate the CarPlay screen 
* Anticipate user scenarios while driving 
* Run your audio apps in CarPlay!

Подведём итог: у аудиоприложений CarPlay есть свои сильные стороны, и вы можете напрямую использовать API `MPPlayableContent`, чтобы передавать данные в своё приложение для CarPlay. Нужно учитывать некоторые ситуации, например блокировку экрана iPhone или отсутствие входа пользователя в аккаунт: приложение всё равно должно хорошо работать в CarPlay.

В iOS12 мы сделали ряд отличных оптимизаций и улучшений производительности, чтобы ваше приложение хорошо работало в CarPlay. Запустите своё приложение ещё раз и посмотрите, есть ли ещё запас для улучшения производительности, чтобы сделать его ещё лучше.
