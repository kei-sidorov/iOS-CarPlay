## iOS CarPlay｜разработка с использованием MediaPlayer framework

Чтобы ваше CarPlay-приложение поддерживало iOS 13 и более ранние версии, нужно изучить [поддержку iOS 13 и более ранних версий iOS](https://developer.Apple.com/documentation/carplay/supporting_previous_versions_of_ios?language=objc). Для аудиоприложений нужно сделать в основном две вещи:

1. Добавить ключ `com.apple.developer.playable-content` в Entitlements.plist и установить значение 1;

```xml
<key>com.apple.developer.playable-content</key>
<true/>
<key>com.apple.developer.carplay-audio</key>
<true/>
```

2. Использовать MediaPlayer framework для разработки.

### Процесс разработки

**CarPlay App Integration**

Разработчики аудиоприложений должны предоставить **источник данных** и **иерархию навигации**, чтобы они отображались в CarPlay.

![](https://gitee.com/junteng/images/raw/master/img/image.png)

Для просмотра контента в CarPlay используется API `MPPlayableContent`, у которого есть `dataSource` и `delegate`: ваше аудиоприложение может передавать данные в CarPlay, а делегат получает обратные вызовы. API `MPNowPlayingInfoCenter` и `MPRemoteCommandCenter` используются для разработки функциональности «Now Playing».

![](https://gitee.com/junteng/images/raw/master/img/image (9).png)

Следующий код — минимум действий, необходимых для поддержки CarPlay в аудиоприложении:

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

**Requirement for Audio Apps**

* После получения разрешения на использование CarPlay приложение должно как минимум реализовать API MPPlayableContent, включая `MPPlayableContentDataSource` и `MPPlayableContentDelegate`, чтобы CarPlay мог получать контент и начинать воспроизведение.
* Приложение должно отвечать на события MPRemoteCommandCenter, позволяя пользователю выполнять команды над вашим контентом, такие как воспроизведение, пауза, переключение треков и так далее.
* Нужно задавать и обновлять словарь MPNowPlayingInfoCenter, содержащий метаданные текущего воспроизводимого аудио, такие как название, автор, длительность и так далее.

**Структура данных аудиоприложения**

CarPlay использует NSIndexPath для запроса content item по определённому пути индекса. Это отличается от NSIndexPath, используемого в UITableView. По аналогии с виртуальной файловой системой, это будет количество файлов в конкретной папке. Пустой путь индекса (0, 0) обозначает корневой узел.

Рассмотрим пример: самый левый контент в этом примере — корневые элементы данных, которые в пользовательском интерфейсе могут быть представлены отдельными вкладками (individual tabs) или в виде rootTableView.

![](https://gitee.com/junteng/images/raw/master/img/image (1).png)

Ниже показаны пути индексов для каждого элемента контента, представленного в иерархии

![](https://gitee.com/junteng/images/raw/master/img/image (3).png)

Когда CarPlay запрашивает content item по определённому пути индекса, аудиоприложение обходит иерархию запрошенных content item, чтобы найти нужный контент.

В этом примере мы запрашиваем первый дочерний элемент первого контента — это content item Running.

![](https://gitee.com/junteng/images/raw/master/img/image (4).png)

CarPlay также запрашивает количество дочерних элементов по определённому индексу. В этом примере мы запрашиваем количество дочерних элементов третьего дочернего элемента второго content item. В данном случае мы вернём 2.

![](https://gitee.com/junteng/images/raw/master/img/image (5).png)

**Ограничения контента**

Некоторые автомобили могут принудительно отображать на экране ограниченный объём контента в зависимости от того, движется ли автомобиль. Это может ограничивать количество строк, отображаемых на экране, а также контент, который можно показать после перехода в определённый каталог. Ваше приложение может использовать MPPlayableContentManager для обработки этих изменений.

Чтобы узнать, ограничивается ли отображаемый контент, реализуйте следующий метод делегата: он позволяет определить, применяются ли ограничения контента, и получить `максимальное количество элементов в списке enforcedContentItemsCount` и `максимальную глубину иерархии enforcedContentTreeDept` при действующих ограничениях.

```swift
extension YourAppContentManager : MPPlayableContentDelegate {
    // called whenever CarPlay state changes
    func playableContentManager(
                _ contentManager: MPPlayableContentManager,
               didUpdate context: MPPlayableManagerContext) {
        // check to see if content limits are enforced
        let contentLimitsEnforced = context.contentLimitsEnforced
        if contentLimitsEnforced {
            // the maximum number of items shown in a list when content limits are enforced
            let contentLimitItemCount = context.enforcedContentItemsCount
            // the maximum depth in the hierarchy when content limits are enforced
            let contentLimitTreeDepth = context.enforcedContentTreeDept
        } else {
            …
        }
    }
}
```

Если вашему приложению больше подходят Tabs, а не TableView:

* добавьте tabs, добавив `UIBrowsableContentSupportsSectionedBrowsing` в Info.plist
* рекомендуется использовать не более 4 tabs с короткими заголовками, так как места мало, экраны некоторых автомобилей узкие, а при наличии воспроизводимого контента нужно ещё показывать кнопку «Now Playing»
* Tab image assets отображаются на экране CarPlay как template images

**Now Playing Screen**

Теперь рассмотрим экран «Now Playing» в CarPlay. Некоторые элементы управления и метаданные такие же, как в Пункте управления на iPhone. В целом элементы управления и данные, которые вы видите в Пункте управления, должны также появляться на экране «Now Playing» в CarPlay. Пользователь может перейти на экран «Now Playing», нажав «Now Playing» в правом верхнем углу навигационного экрана приложения или приложение «Now Playing» на главном экране CarPlay. Во время работы название вашего приложения отображается в правом верхнем углу экрана «Now Playing».

![](https://gitee.com/junteng/images/raw/master/img/image (6).png)

Метаданные на экране «Now Playing» в CarPlay задаются так же, как для `Пункта управления` и других источников. Задайте свойство nowPlayingInfo объекта `MPNowPlayingInfoCenter` и заполните как можно больше информации.

```swift
// Set Metadata to be Displayed in Now Playing Info Center

let infoCenter = MPNowPlayingInfoCenter.default()
infoCenter.nowPlayingInfo = [MPMediaItemPropertyTitle: "Style",
                             MPMediaItemPropertyArtist: "Taylor Swift",
                             MPMediaItemPropertyAlbumTitle: "1989",
                             MPMediaItemPropertyGenre: "Pop",
                             MPMediaItemPropertyReleaseDate: "2014",
                             MPMediaItemPropertyPlaybackDuration: 231,
                             MPMediaItemPropertyArtwork: mediaItemArtwork,
                             MPNowPlayingInfoPropertyElapsedPlayback: 53,
                             MPNowPlayingInfoPropertyDefaultPlaybackRate: 1,
                             MPNowPlayingInfoPropertyPlaybackRate: 1,
                             MPNowPlayingInfoPropertyPlaybackQueueCount: 13,
                             MPNowPlayingInfoPropertyPlaybackQueueIndex: 3,
                            … ]
```

**Playback Controls**

Кроме того, отвечайте на команды воспроизведения, чтобы пользователь мог управлять контентом: воспроизведение, пауза, переключение треков, случайный порядок или повтор плейлиста. Ниже приведены команды, поддерживаемые на экране «Now Playing» в CarPlay

MPRemoteCommandCenter

* Play, pause, stop
* Previous track, next track
* Seek backward, seek forward
* Skip backward, skip forward
* Shuffle, repeat
* Like, dislike, bookmark
* Change playback rate

**Changing Playback Rate**

Новая возможность iOS 11 — экран «Now Playing» поддерживает отображение и изменение скорости воспроизведения. Если вашему приложению нужна такая поддержка:

* добавьте `MPNowPlayingInfoPropertyDefaultPlaybackRate` в `MPNowPlayingInfoCenter`
* и используйте массив поддерживаемых скоростей воспроизведения, хранящийся в `MPNowPlayingInfoCenter`, чтобы отвечать на команду изменения скорости воспроизведения
* Implement `changePlaybackRateCommand` with `supportedPlaybackRates` in `MPRemoteCommandCenter`

Ниже приведён пример кода, как реализовать изменение скорости воспроизведения. В этом примере текущая скорость воспроизведения равна 1.0, а поддерживаемые варианты скорости — 0.5, 1.0, 1.5, 2.0. Когда пользователь хочет увеличить скорость воспроизведения, если аудио уже воспроизводится на максимальной поддерживаемой скорости, дальнейшее увеличение циклически переключит скорость на минимальную. В этом примере, если пользователь трижды подряд увеличит скорость, она изменится так: 1.0 -> 1.5 -> 2.0 -> 0.5.

```swift
// Change Playback Rate

let infoCenter = MPNowPlayingInfoCenter.default()
infoCenter.nowPlayingInfo = [MPNowPlayingInfoPropertyDefaultPlaybackRate: 1.0,
                             … ]

let changePlaybackRateCommand = MPRemoteCommandCenter.shared().changePlaybackRateCommand
changePlaybackRateCommand.supportedPlaybackRates = [0.5, 1.0, 1.5, 2.0]
```

**Playback Controls**

Некоторые аудиоприложения могут отвечать на несколько команд в `MPRemoteCommandCenter`. В зависимости от поддерживаемых команд экран «Now Playing» в CarPlay объединяет некоторые связанные команды в одну кнопку, например кнопку `многоточие` или `меню`, которая заменяет прежние кнопки.

![](https://gitee.com/junteng/images/raw/master/img/image (7).png)

![](https://gitee.com/junteng/images/raw/master/img/image (8).png)

**Best Practices**

* **Перед вызовом completion handlers в `MPPlayableContentDataSource` и `MPPlayableContentDelegate` убедитесь, что контент, который будет воспроизводиться, действительно готов к воспроизведению или отображению, а на это время показывайте индикатор активности.** (Иначе произойдёт переход на экран воспроизведения и сразу выход с него)
* Для приложений, которые используют только `TableView` без `Tabs`, `TableView` должен возвращать как минимум один элемент контента.
* CarPlay показывает индикатор загрузки, и если приложение не вернёт контент в течение определённого времени, произойдёт тайм-аут. Если вашему приложению требуется первоначальная настройка, например учётные данные для входа, то в первой строке страницы следует сообщить пользователю текущее состояние приложения, заполнив `MPContentItem`; MPContentItem is neither playable nor a container indicating the state of the app to the user.

**Улучшения производительности и оптимизации в iOS12**

* Improved performance in `MPPlayableContent`
* Faster startup sequence 
* Smoother animations 
* Better communication to your app

API `MPPlayableContent` был оптимизирован: улучшена производительность того, как запрашиваются источник данных и делегат, при этом изменять код не требуется. Ускорен процесс запуска, а анимации стали плавнее. Кроме того, когда контент текущего аудиоприложения в CarPlay меняется, ваше приложение получает более качественную информацию, что позволяет предугадать, какой контент пользователь захочет воспроизвести или просмотреть в CarPlay.

**Best Practices**

Рассмотрим подробнее, как оптимизировать аудиоприложение:

* Call `reloadData()` only when needed
  * Это API из `MPPlayableContent`, который умеет лучше оптимизировать работу вашего аудиоприложения. Вызывать `reloadData()` следует только при необходимости: это очень дорогая операция, снижающая отзывчивость приложения.
* Use `beginUpdates()` and `endUpdates()`
  * Наоборот, если нужно обновить лишь часть контента, помещайте обновления между этими двумя вызовами API, чтобы контент обновлялся корректно.
* Keep an internal representation of the data source to optimize performance
  * Сейчас вызовы `MPPlayableContent` — асинхронные операции, когда мы запрашиваем данные у вашего приложения. Поэтому где-то в приложении нужно хранить внутреннее представление или кэш информации, чтобы при запросе контента вы могли быстро предоставить данные и сохранить отзывчивость приложения.

**Don’t Miss a Beat!**

Account for these common scenarios

* Screen locked with passcode
* Unreliable network connectivity

Далее обсудим способы «дальнейшей оптимизации производительности аудиоприложения в CarPlay»:

Список воспроизведения загружается нестабильно, а если приложение не предоставляет контент вовремя, это приводит к тайм-ауту загрузки в CarPlay. Чем это может быть вызвано?

* Пользователи CarPlay часто ездят в местах без быстрого соединения или при заблокированном экране телефона, а для разблокировки экрана требуется пароль. Если политика защиты данных вашего приложения зависит от того, что телефон разблокирован, вы не сможете получить информацию своего приложения, и в итоге CarPlay завершит работу по тайм-ауту. Поэтому, если ваши данные доступны только при разблокированном телефоне, нужно пересмотреть политику защиты данных вашего приложения.
* Пользователи CarPlay могут ездить в районах со слабой сетью или без сети, разные CarPlay предоставляют разные сервисы передачи данных, и вам нужно протестировать работу без постоянного подключения к WIFI.

![](https://gitee.com/junteng/images/raw/master/img/image (10).png)

Как оптимизировать?

Используйте `beginLoadingChildItems()`, чтобы инициировать получение контента.

Есть API `beginLoadingChildItems`, предназначенный для запуска получения контента; он вызывается всякий раз, когда любой из ваших путей индексов отображается в автомобиле CarPlay (можно сравнить с методом `cellForRow` у `tableview`). Когда пользователь прокручивает `tableview` или выбирает разные `tabs`, `beginLoadingChildItems` вызывается для каждого отдельного пути индекса, отображаемого на экране. Так у вашего приложения есть возможность начать загрузку до того, как пользователь реально выберет контент. Поэтому к моменту выбора контента пользователем сетевой запрос либо уже выполняется, либо данные уже готовы.

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

Далее инженер Apple приводит сценарий, с которым вы можете столкнуться при разработке CarPlay-приложения.

Если вашему аудиоприложению для предоставления контента нужен вход в аккаунт, то опыт использования в CarPlay будет очень плохим: пользователь не сможет взаимодействовать с вашим приложением. Нужно обеспечить хоть какой-то вариант работы, чтобы пользователь мог хотя бы без входа в аккаунт пользоваться вашим приложением в CarPlay, — это улучшит пользовательский опыт. 

![](https://gitee.com/junteng/images/raw/master/img/image (11).png)

**Greatest Hits**

* Use `MPPlayableContent` to populate the CarPlay screen 
* Anticipate user scenarios while driving 
* Run your audio apps in CarPlay!

Подведём итог: у аудиоприложений CarPlay есть свои сильные стороны, и вы можете напрямую использовать API `MPPlayableContent`, чтобы предоставлять контент для вашего CarPlay-приложения. Нужно учитывать некоторые ситуации, например блокировку экрана iPhone или отсутствие входа пользователя в аккаунт: в таких случаях ваше приложение всё равно должно корректно работать в CarPlay.

В iOS12 мы сделали ряд отличных оптимизаций и улучшений производительности, чтобы ваше приложение хорошо работало в CarPlay. Запустите своё приложение ещё раз и посмотрите, есть ли ещё возможности для улучшения производительности, чтобы сделать его ещё лучше.

### API

#### MPPlayableContentManager

MPPlayableContentManager — класс, управляющий взаимодействием между медиаприложением и внешним интерфейсом медиаплеера. Приложение предоставляет ContentManager:

* dataSource — источник данных, позволяющий медиаплееру просматривать медиаконтент, предоставляемый приложением
* delegate — делегат, позволяющий медиаплееру пересылать приложению немедиа-команды удалённого воспроизведения

```swift
@available(iOS, introduced: 7.1, deprecated: 14.0, message: "Use CarPlay framework")
class MPPlayableContentManager : NSObject {

    weak var dataSource: MPPlayableContentDataSource?
    weak var delegate: MPPlayableContentDelegate?

    @available(iOS, introduced: 8.4, deprecated: 14.0, message: "Use CarPlay framework")
    var context: MPPlayableContentManagerContext { get }

    /// Сообщает ContentManager идентификаторы воспроизводимых в данный момент MPContentItems
    @available(iOS, introduced: 10.0, deprecated: 14.0, message: "Use CarPlay framework")
    var nowPlayingIdentifiers: [String]

    class func shared() -> Self

    /// Сообщает ContentManager, что источник данных изменился и данные нужно перезагрузить из источника данных
    func reloadData()
    
    /// Используется для начала синхронного обновления нескольких MPContentItems
    func beginUpdates()
    /// Завершает синхронное обновление
    func endUpdates()
}
```

#### MPPlayableContentManagerContext

MPPlayableContentManagerContext представляет текущее состояние конечной точки playable-контента; context можно получить из экземпляра MPPlayableContentManager.

```swift
@available(iOS, introduced: 8.4, deprecated: 14.0, message: "Use CarPlay framework")
class MPPlayableContentManagerContext : NSObject {

    /// Количество items, отображаемых content server при ограничении контента
    /// Если contentLimitsEnforced равен false, возвращает NSIntegerMax
    var enforcedContentItemsCount: Int { get }
    /// Глубина иерархии навигации, разрешённая content server; превышение этого ограничения приведёт к Crash
    var enforcedContentTreeDepth: Int { get }
    /// Ограничивает ли content server контент
    var contentLimitsEnforced: Bool { get }
    /// Доступен ли content server
    var endpointAvailable: Bool { get }
}
```

#### MPPlayableContentDataSource

MPPlayableContentDataSource — протокол, которому должен соответствовать объект application, если он хочет поддерживать внешние медиаплееры (например, CarPlay). Источник данных отвечает за то, чтобы осмысленным образом предоставлять этим системам метаданные о медиа, благодаря чему можно автоматически настраивать такие функции, как пользовательский интерфейс и очередь воспроизведения.

```swift
@available(iOS, introduced: 7.1, deprecated: 14.0, message: "Use CarPlay framework")
protocol MPPlayableContentDataSource : NSObjectProtocol {

    /// Сообщает источнику данных начать загрузку child content items для item, заданного indexPath
    /// Это нужно, чтобы приложение могло начать асинхронную пакетную загрузку контента до того, как MediaPlayer начнёт запрашивать отображение content items
    /// Если этот метод реализован, приложение всегда должно вызывать completionHandler после завершения загрузки
    optional func beginLoadingChildItems(at indexPath: IndexPath, completionHandler: @escaping (Error?) -> Void)

    /// Сообщает MediaPlayer, поддерживает ли контент, предоставляемый источником данных, прогресс воспроизведения в качестве свойства своих метаданных.
    /// Если этот метод не реализован, MediaPlayer будет считать, что прогресс не поддерживается ни для одного content item.
    optional func childItemsDisplayPlaybackProgress(at indexPath: IndexPath) -> Bool

    /// Предоставляет content item для переданного identifier
    /// Если content item, соответствующего identifier, нет, предоставляется nil
    /// Если есть error, не позволяющая получить content item, предоставляется error
    /// Если этот метод реализован, приложение всегда должно вызывать completionHandler после завершения загрузки
    @available(iOS, introduced: 10.0, deprecated: 14.0, message: "Use CarPlay framework")
    optional func contentItem(forIdentifier identifier: String, completionHandler: @escaping (MPContentItem?, Error?) -> Void)
    /// Возвращает количество дочерних узлов по указанному indexPath
    /// В виртуальной файловой системе это будет количество файлов в конкретной папке. Пустой путь индекса (0, 0) обозначает корневой узел
    func numberOfChildItems(at indexPath: IndexPath) -> Int

    /// Возвращает content item, находящийся по указанному indexPath
    /// Если content item изменился после возврата, его обновлённое содержимое будет отправлено в MediaPlayer
    func contentItem(at indexPath: IndexPath) -> MPContentItem?
}
```

#### MPPlayableContentDelegate

MPPlayableContentDelegate — протокол, позволяющий внешнему медиаплееру отправлять приложению команды воспроизведения. Например, пользователь может просматривать медиаконтент приложения (предоставляемый MPPlayableContentDataSource) и выбрать content item для воспроизведения. Если медиаплеер решает воспроизвести этот item, он попросит content delegate приложения запустить воспроизведение.

```swift
@available(iOS, introduced: 7.1, deprecated: 14.0, message: "Use CarPlay framework")
protocol MPPlayableContentDelegate : NSObjectProtocol {
    
    /// Этот метод вызывается, когда интерфейс медиаплеера хочет воспроизвести запрошенный content item
    /// Если при начале воспроизведения item возникла ошибка, приложение должно вызвать completionHandler и передать error
    /// [Примечание] Нажатие на MPContentItem с isPlayable, равным true, вызывает этот метод
    @available(iOS, introduced: 7.1, deprecated: 14.0, message: "Use CarPlay framework")
    optional func playableContentManager(_ contentManager: MPPlayableContentManager, initiatePlaybackOfContentItemAt indexPath: IndexPath, completionHandler: @escaping (Error?) -> Void)
    
    /// Этот метод вызывается, когда интерфейс медиаплеера хочет, чтобы now playing app подготовил очередь воспроизведения для последующего воспроизведения
    /// app должно загрузить контент в очередь воспроизведения, но начинать воспроизведение только после получения команды воспроизведения или когда playableContentManager запросит воспроизведение другого контента
    /// Как только app будет готово что-то воспроизводить, оно должно вызвать completionHandler
    @available(iOS, introduced: 9.0, deprecated: 9.3, message: "Use Intents framework for initiating playback queues.")
    optional func playableContentManager(_ contentManager: MPPlayableContentManager, initializePlaybackQueueWithCompletionHandler completionHandler: @escaping (Error?) -> Void)
  
    /// Этот метод вызывается, когда интерфейс медиаплеера хочет, чтобы now playing app подготовил очередь воспроизведения для последующего воспроизведения
    /// app должно загрузить контент в очередь воспроизведения на основе переданных content item и подготовиться к воспроизведению, но не начинать воспроизведение до получения команды воспроизведения или до тех пор, пока playableContentManager не запросит воспроизведение другого контента
    /// nil contentItems array означает, что app должно подготовить свою очередь из того, что сочтёт подходящим.
    /// Как только app будет готово что-то воспроизводить, оно должно вызвать completionHandler
    @available(iOS, introduced: 9.3, deprecated: 12.0, message: "Use Intents framework for initiating playback queues.")
    optional func playableContentManager(_ contentManager: MPPlayableContentManager, initializePlaybackQueueWithContentItems contentItems: [Any]?, completionHandler: @escaping (Error?) -> Void)

    /// Этот метод вызывается, когда content server уведомляет manager о том, что текущий context изменился
    @available(iOS, introduced: 8.4, deprecated: 14.0, message: "Use CarPlay framework")
    optional func playableContentManager(_ contentManager: MPPlayableContentManager, didUpdate context: MPPlayableContentManagerContext)
}
```

#### MPContentItem

MPContentItem представляет высокоуровневые метаданные конкретного медиа-элемента, используемые для его представления вне клиентского приложения. Примеры медиа-элементов, которые разработчик может захотеть представить: файл песни, url потокового аудио или радиостанция.

```swift
@available(iOS 7.1, *)
class MPContentItem : NSObject {
    /// Designated initializer. Требуется уникальный идентификатор, чтобы идентифицировать item для последующего использования.
    init(identifier: String)

    /// Уникальный идентификатор (Required)
    var identifier: String { get }
    /// Заголовок
    var title: String?
    /// Подзаголовок
    var subtitle: String?
    /// Обложка
    var artwork: MPMediaItemArtwork?
    /// Прогресс воспроизведения: 0.0 — не воспроизводилось, 1.0 — воспроизведено полностью, по умолчанию -1.0 (индикатор прогресса не показывается)
    var playbackProgress: Float
    /// Является ли контент потоковым, то есть хранится в cloud, а не локально
    @available(iOS 10.0, *)
    var isStreamingContent: Bool 
    /// Является ли контент explicit-контентом
    @available(iOS 10.0, *)
    var isExplicitContent: Bool
    /// Является ли контейнером, содержащим другие content item, например album или playlist
    var isContainer: Bool
    /// Можно ли воспроизводить
    var isPlayable: Bool
}
```

### Экран воспроизведения

Независимо от того, используете ли вы для создания CarPlay-приложения CarPlay framework или MediaPlayer framework, аудиоинформация для экрана воспроизведения предоставляется и отвечает на события удалённого управления воспроизведением через [MPNowPlayingInfoCenter](https://developer.apple.com/documentation/mediaplayer/mpnowplayinginfocenter/) и [MPRemoteCommandCenter](https://developer.apple.com/documentation/mediaplayer/mpremotecommandcenter/). Разница в том, что при использовании CarPlay framework команды удалённого управления, такие как режим повтора (changeRepeatModeCommand) и скорость воспроизведения (changePlaybackRateCommand), обрабатываются не через target-action, а через handler у CPNowPlayingRepeatButton и CPNowPlayingPlaybackRateButton; а при использовании MediaPlayer framework эти command обрабатываются через target-action.

#### Подводные камни при изменении скорости воспроизведения

Отображение и изменение скорости воспроизведения на экране воспроизведения — новая возможность iOS 11.

Чтобы поддержать её с помощью CarPlay framework, нужно сделать несколько вещей:

1. Добавить MPNowPlayingInfoPropertyPlaybackRate и MPNowPlayingInfoPropertyDefaultPlaybackRate в MPNowPlayingInfoCenter
2. Установить changePlaybackRateCommand.enabled в true
3. Создать экземпляр CPNowPlayingRepeatButton и обновить nowPlayingTemplate через updateNowPlayingButtons
4. В handler у CPNowPlayingRepeatButton обновить скорость воспроизведения аудио в приложении и обновить nowPlayingInfo (обновить значения MPNowPlayingInfoPropertyPlaybackRate и MPNowPlayingInfoPropertyDefaultPlaybackRate)

```swift
// 1.
let infoCenter = MPNowPlayingInfoCenter.default()
infoCenter.nowPlayingInfo = [MPNowPlayingInfoPropertyDefaultPlaybackRate: 1,
                             MPNowPlayingInfoPropertyPlaybackRate: 1,
                            … ]
// 2.
let remoteCommandCenter = MPRemoteCommandCenter.shared()
remoteCommandCenter.changePlaybackRateCommand.enabled = true
// 3.
let playbackRateButton = CPNowPlayingPlaybackRateButton() { 
    // 4. Обновить скорость воспроизведения аудио в приложении и обновить nowPlayingInfo
    ... 
}
nowPlayingTemplate.updateNowPlayingButtons([playbackRateButton, ...])
```

При использовании MediaPlayer framework нужно сделать другое:

1. Добавить MPNowPlayingInfoPropertyPlaybackRate и MPNowPlayingInfoPropertyDefaultPlaybackRate в MPNowPlayingInfoCenter
2. Установить changePlaybackRateCommand.enabled в true
3. Добавить target-action для changePlaybackRateCommand.enabled
4. Установить changePlaybackRateCommand.supportedPlaybackRates — массив поддерживаемых скоростей воспроизведения
5. В action обновить скорость воспроизведения аудио в приложении и обновить nowPlayingInfo (обновить значения MPNowPlayingInfoPropertyPlaybackRate и MPNowPlayingInfoPropertyDefaultPlaybackRate)

```swift
// 1.
let infoCenter = MPNowPlayingInfoCenter.default()
infoCenter.nowPlayingInfo = [MPNowPlayingInfoPropertyDefaultPlaybackRate: 1,
                             MPNowPlayingInfoPropertyPlaybackRate: 1,
                            … ]
// 2.
MPRemoteCommandCenter *commandCenter = [MPRemoteCommandCenter sharedCommandCenter];
commandCenter.changePlaybackRateCommand.enabled = YES;
// 3.
[commandCenter.changePlaybackRateCommand addTarget:self action:@selector(changePlaybackRate)];
// 4.
commandCenter.changePlaybackRateCommand.supportedPlaybackRates = @[@0.5, @1.0, @1.5, @2.0];
// 5.
- (MPRemoteCommandHandlerStatus)changePlaybackRate {
    // Обновить скорость воспроизведения аудио в приложении и обновить nowPlayingInfo
    ...
    return MPRemoteCommandHandlerStatusSuccess;
}
```

Лучшая практика изменения скорости воспроизведения в CarPlay такова: например, текущая скорость воспроизведения равна 1.0, а поддерживаемые варианты — `[0.5, 1.0, 1.5, 2.0]`. Когда пользователь нажимает кнопку скорости воспроизведения, приложение увеличивает скорость и синхронизирует её с CarPlay, обновляя nowPlayingInfo. Если аудио уже воспроизводится на максимальной поддерживаемой скорости, дальнейшее увеличение переключает скорость на минимальную, и так по кругу. В этом примере, если пользователь трижды подряд увеличит скорость, она изменится так: 1.0 -> 1.5 -> 2.0 -> 0.5.

При поддержке скорости воспроизведения через MediaPlayer framework я столкнулся с проблемой: аудио в нашем приложении поддерживает варианты скорости `[0.5, 0.6, 0.7, 0.8, 0.9, 1.0, 1.1, 1.2, 1.3, 1.4, 1.5]`. При нажатии кнопки скорости воспроизведения на экране воспроизведения CarPlay скорость успешно изменилась один раз (1.0 -> 1.1), а дальше сколько ни нажимай, эффекта нет, action не вызывается; при этом изменение скорости в приложении на iPhone нормально синхронизируется с CarPlay-приложением. Меня смутило, что в докладе «WWDC17 - Добавьте в своё приложение поддержку CarPlay» инженеры Apple не упоминали никаких дополнительных шагов для поддержки изменения скорости воспроизведения в CarPlay-приложении. Изучив документацию Apple и поискав в Google, я так и не нашёл причину проблемы и её решение.

В итоге я заменил значение supportedPlaybackRates на пример, который инженеры Apple показывали в докладе «WWDC17 - Добавьте в своё приложение поддержку CarPlay», `[0.5, 1.0, 1.5, 2.0]`, и поменял на него же скорости, поддерживаемые нашим приложением, — и функция заработала нормально. Мне показалось, что я нашёл зацепку, и я изменил поддерживаемые скорости на `[0.5, 0.6, 1.0, 1.5, 2.0]`, после чего моя догадка подтвердилась. При нажатии кнопки скорости воспроизведения на экране воспроизведения CarPlay скорость успешно менялась 1.0 -> 1.5 -> 2.0 -> 0.5, пока всё хорошо, но когда я снова нажимал кнопку, action уже не вызывался. Затем в приложении на iPhone я устанавливал скорость воспроизведения в любое из значений 1.0, 1.5, 2.0 и обновлял nowPlayingInfo, и кнопка скорости воспроизведения в CarPlay-приложении снова работала нормально — до тех пор, пока скорость не дойдёт до 0.5, после чего снова переставала работать.

Поэтому я пришёл к выводу, что при использовании MediaPlayer framework поддерживаемые CarPlay-приложением скорости воспроизведения должны быть кратны 0.5, иначе функциональность работает неправильно, но подтверждения этому в документации Apple я пока не нашёл. Чтобы сохранить совместимость с рядом значений скорости воспроизведения, поддерживаемых приложением на iPhone, `[0.5, 0.6, 0.7, 0.8, 0.9, 1.0, 1.1, 1.2, 1.3, 1.4, 1.5]`, и при этом чтобы CarPlay-приложение поддерживало изменение скорости воспроизведения `[0.5, 1.0, 1.5]`, я выбрал такое решение:

* если пользователь меняет скорость воспроизведения через CarPlay-приложение, скорость переключается между тремя значениями `[0.5, 1.0, 1.5]`
* если пользователь меняет скорость воспроизведения через приложение на iPhone, то при условии, что установленное значение входит в `[0.5, 1.0, 1.5]`, в CarPlay-приложении отображается кнопка скорости воспроизведения; иначе кнопка скорости воспроизведения скрывается





Как добавить вкладки в CarPlay for Audio App? https://www.xknote.com/ask/60d536b9ec7f1.html

## Проблемы

* Если нажать на item для воспроизведения, а воспроизведение фактически ещё не началось, то произойдёт push экрана воспроизведения и сразу за ним автоматический pop. Нужно вызывать completionHandler только после того, как аудио действительно начнёт воспроизводиться
