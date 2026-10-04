## [Перевод] Добавляем поддержку CarPlay в Swift Radio

Оригинал: [Fethi El Hassasna｜Add CarPlay support to Swift Radio](https://blog.fethica.com/add-carplay-support-to-swiftradio/#)

В этом руководстве мы посмотрим, как добавить поддержку **CarPlay** в open-source радиоприложение [SwiftRadio](https://github.com/analogcode/Swift-Radio-Pro) и протестировать её в симуляторе.

### Настройка проекта

Для начала склонируем проект или просто скачаем его с [GitHub](https://github.com/analogcode/Swift-Radio-Pro):

```
git clone https://github.com/analogcode/Swift-Radio-Pro
```

После запуска проекта в симуляторе проверим, есть ли меню **CarPlay** по следующему пути: **Hardware (I/O) > External Displays > CarPlay**:

![](https://gitee.com/junteng/images/raw/master/img/20220107135013.png)

Если меню CarPlay не найдено, откройте терминал и выполните следующую команду:

```
defaults write com.apple.iphonesimulator CarPlay -bool YES
```

> [Примечание] Сейчас этот шаг, по-видимому, уже не нужен.

Теперь нужно добавить в проект файл entitlements (.entitlement), добавить в него ключ `com.apple.developer.playable-content` со значением `Boolean/YES`, чтобы включить поддержку **CarPlay**.

Чтобы файл создался автоматически, достаточно в **target > capabilities** проекта включить **PushNotification**, а затем снова выключить.

![](https://gitee.com/junteng/images/raw/master/img/20220107140228.png)

> [Примечание]: сейчас достаточно в **target > Signing & Capabilities** добавить любой Capability — файл .entitlements создастся автоматически — а затем удалить добавленный Capability.
>
> ![](https://gitee.com/junteng/images/raw/master/img/20220107140626.png)
>
> ![](https://gitee.com/junteng/images/raw/master/img/20220107141132.png)

Файл `SwiftRadio.entitlements` должен выглядеть так:

![](https://gitee.com/junteng/images/raw/master/img/20220107142032.png)

При повторном запуске приложения мы должны увидеть наше **CarPlay**-приложение:

![](https://gitee.com/junteng/images/raw/master/img/20220107142138.png)

Теперь вернёмся к проекту и начнём добавлять код.

Сначала нужно импортировать framework **MediaPlayer** в классе **AppDelegate** и добавить новое свойство `playableContentManager`.

```swift
// AppDelegate.swift

import UIKit
import MediaPlayer

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
    weak var stationsViewController: StationsViewController?
    var playableContentManager: MPPlayableContentManager?
    
    // ...
```

Чтобы отделить логику CarPlay, создадим для `AppDelegate` категорию (extension) в файле `AppDelegate+CarPlay.swift`.

Сначала добавим метод `setupCarPlay`, который инициализирует свойство `playableContentManager` и устанавливает его `delagate` и `dataSource` равными `self (AppDelagate)`:

```swift
// AppDelegate+CarPlay.swift

import Foundation
import MediaPlayer

extension AppDelegate {
    
    func setupCarPlay() {
        playableContentManager = MPPlayableContentManager.shared()
        
        playableContentManager?.delegate = self
        playableContentManager?.dataSource = self
    }
}
```

Далее с помощью extension реализуем протоколы `delagate` и `dataSource` и добавим обязательные методы:

```swift
// AppDelegate+CarPlay.swift

import Foundation
import MediaPlayer

extension AppDelegate {
    
    func setupCarPlay() {
        playableContentManager = MPPlayableContentManager.shared()
        
        playableContentManager?.delegate = self
        playableContentManager?.dataSource = self
    }
}

extension AppDelegate: MPPlayableContentDelegate {
    
    func playableContentManager(_ contentManager: MPPlayableContentManager, initiatePlaybackOfContentItemAt indexPath: IndexPath, completionHandler: @escaping (Error?) -> Void) {
        completionHandler(nil)
    }
    
    func beginLoadingChildItems(at indexPath: IndexPath, completionHandler: @escaping (Error?) -> Void) {
    }
}

extension AppDelegate: MPPlayableContentDataSource {
    
    func numberOfChildItems(at indexPath: IndexPath) -> Int {
        return 0
    }
    
    func contentItem(at indexPath: IndexPath) -> MPContentItem? {
        return nil
    }
}
```

Чтобы получить данные для `datasource` (в нашем примере это список радиостанций), добавим класс `CarPlayPlaylist`. В этом классе будет свойство для хранения массива станций и метод, загружающий данные из нашего класса `DataManager`.

```swift
// CarPlayPlaylist.swift

import Foundation

class CarPlayPlaylist {
    
    var stations = [RadioStation]()
    
    func load(_ completion: @escaping (Error?) -> Void) {
        
        DataManager.getStationDataWithSuccess() { (data) in
            
            guard let data = data else {
                completion(nil)
                return
            }
            
            do {
                let jsonDictionary = try JSONDecoder().decode([String: [RadioStation]].self, from: data)
                if let stationsArray = jsonDictionary["station"] {
                    self.stations = stationsArray
                }
            } catch (let error) {
                completion(error)
                return
            }
            
            completion(nil)
        }
    }    
}
```

Теперь добавим в `AppDelegate` свойство `carPlayPlaylist`:

```swift
// AppDelegate.swift

import UIKit
import MediaPlayer

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?
    weak var stationsViewController: StationsViewController?
    var playableContentManager: MPPlayableContentManager?
    let carplayPlaylist = CarPlayPlaylist()
  
// ...
```

В `datasource` нам нужно создать tab для отображения списка станций. Для этого необходимо добавить в `info.plist` ключ `UIBrowsableContentSupportsSectionedBrowsing` со значением `Boolean/YES`.

![](https://gitee.com/junteng/images/raw/master/img/20220107150450.png)

Теперь добавим в `datasource` нужные данные: для `numberOfChildItems` количество tabs равно 1, а количество items равно `count` у `CarPlayPlayList stations`:

```swift
// AppDelegate+CarPlay.swift

func numberOfChildItems(at indexPath: IndexPath) -> Int {
    if indexPath.indices.count == 0 {
        return 1
    }
    return carplayPlaylist.stations.count
}
```

В методе `contentItem(at indexPath: IndexPath) -> MPContentItem?` для каждой секции (tab и list) мы создаём item типа `MPContentItem`. Для tab добавим:

```swift
if indexPath.count == 1 {
    // Tab section
    let item = MPContentItem(identifier: "Stations")
    item.title = "Stations"
    item.isContainer = true
    item.isPlayable = false
    if let tabImage = UIImage(named: "carPlayTab") {
        item.artwork = MPMediaItemArtwork(boundsSize: tabImage.size, requestHandler: { _ -> UIImage in
            return tabImage
        })
    }
    return item
}
```

> Иконки carPlayTab можно скачать [здесь](https://blog.fethica.com/assets/zips/carPlayTabIcon.zip) и добавить в images .xcassets проекта.

Для списка станций:

```swift
if indexPath.count == 1 {
    // Tab section
    // ...
} else if indexPath.count == 2, indexPath.item < carplayPlaylist.stations.count {
            
	  // Stations section
    let station = carplayPlaylist.stations[indexPath.item]
    let item = MPContentItem(identifier: "\(station.name)")
    item.title = station.name
    item.subtitle = station.desc
    item.isPlayable = true
    item.isStreamingContent = true
    
    // Get the station image from http or local
    if station.imageURL.contains("http") {
        ImageLoader.sharedLoader.imageForUrl(urlString: station.imageURL) { image, _ in
            DispatchQueue.main.async {
                guard let image = image else { return }
                item.artwork = MPMediaItemArtwork(boundsSize: image.size, requestHandler: { _ -> UIImage in
                    return image
                })
            }
        }
    } else {
        if let image = UIImage(named: station.imageURL) ?? UIImage(named: "stationImage") {
            item.artwork = MPMediaItemArtwork(boundsSize: image.size, requestHandler: { _ -> UIImage in
                return image
            })
        }
    }
    return item
} else {
    return nil
}
```

В delegate-методе `beginLoadingChildItems` мы вызываем метод загрузки у carPlayPlaylist.

```swift
func beginLoadingChildItems(at indexPath: IndexPath, completionHandler: @escaping (Error?) -> Void) {
    carplayPlaylist.load { error in
        completionHandler(error)
    }
}
```

Наконец, не забудьте вызвать метод `setupCarPlay` в AppDelegate.

```swift
// AppDelegate.swift

func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        
    // ...
        
    setupCarPlay()
        
    return true
}
```

Запустим приложение ещё раз — на дисплее CarPlay мы должны увидеть этот список:

![](https://gitee.com/junteng/images/raw/master/img/20220107160820.png)

### Обработка воспроизведения

Сначала создадим новый файл и расширим класс StationsViewController методом для воспроизведения станции, назвав его `selectFromCarPlay:`:

```swift
// StationsViewController+CarPlay.swift

import UIKit

extension StationsViewController {
    func selectFromCarPlay(_ station: RadioStation) {
        radioPlayer.station = station
        handleRemoteStationChange()
    }
}
```

В delegate-методе `playableContentManager` мы по indexPath получаем выбранную station, вызываем метод `selectFromCarPlay:` у stationViewController и передаём в него station:

```swift
// AppDelegate+CarPlay.swift

func playableContentManager(_ contentManager: MPPlayableContentManager, initiatePlaybackOfContentItemAt indexPath: IndexPath, completionHandler: @escaping (Error?) -> Void) {
        
    DispatchQueue.main.async {
        if indexPath.count == 2 {
            let station = self.carplayPlaylist.stations[indexPath[1]]
            self.stationsViewController?.selectFromCarPlay(station)
        }
        
        completionHandler(nil)
    }
}
```

Если снова запустить приложение, мы сможем воспроизводить выбранную станцию прямо из CarPlay.

> Если при выборе станции вы получаете ошибку **out of range**, просто перезапустите приложение-симулятор **CarPlay** — плейлист перезагрузится, и проблема исчезнет.

### Обходное решение для отображения экрана «Now Playing» в симуляторе

Вот обходное решение, позволяющее увидеть экран `NowPlaying` в симуляторе (на реальном устройстве оно не требуется), [источник](https://stackoverflow.com/questions/52818170/handling-playback-events-in-carplay-with-mpnowplayinginfocenter).

Обновим delegate-метод следующим образом:

```swift
func playableContentManager(_ contentManager: MPPlayableContentManager, initiatePlaybackOfContentItemAt indexPath: IndexPath, completionHandler: @escaping (Error?) -> Void) {
        
    DispatchQueue.main.async {
            
        if indexPath.count == 2 {
            let station = self.carplayPlaylist.stations[indexPath[1]]
            self.stationsViewController?.selectFromCarPlay(station)
        }
        completionHandler(nil)
        
        // Workaround to make the Now Playing working on the simulator:
        #if targetEnvironment(simulator)
            UIApplication.shared.endReceivingRemoteControlEvents()
        	  UIApplication.shared.beginReceivingRemoteControlEvents()
        #endif
    }
}
```

Если снова запустить приложение, мы увидим экран **NowPlaying** в **CarPlay**:

![](https://gitee.com/junteng/images/raw/master/img/20220107163558.png)

> Вы заметите, что кнопка воспроизведения не синхронизирована с состоянием воспроизведения: при первом запуске она показывается в состоянии паузы, потому что этим кодом мы отключили начальные remote-события.

### Тестирование на реальном устройстве

Чтобы запустить приложение на реальном устройстве или опубликовать его в App Store, нужно запросить у Apple entitlement для **CarPlay** с помощью [этой формы](https://developer.apple.com/contact/carplay/).

Спасибо [@urayoanm](https://github.com/urayoanm) за то, что поделился своим опытом в этом GitHub [issue](https://github.com/analogcode/Swift-Radio-Pro/issues/104#issuecomment-433407949).

Как только ваш запрос будет одобрен, вы получите письмо о том, что entitlement для **CarPlay** добавлен в ваш account, и сможете сгенерировать для своего приложения explicit provisioning profile с entitlement для **CarPlay**.

### Заключение

Весь этот код отправлен в **CarPlay** [branch](https://github.com/analogcode/Swift-Radio-Pro/tree/carplay) репозитория **SwiftRadio**.

Больше информации о **CarPlay**:

- [CarPlay Audio and Navigation Apps - WWDC 2018](https://developer.apple.com/videos/play/wwdc2018/213/)
- [Enabling Your App for CarPlay - WWDC 2017](https://developer.apple.com/videos/play/wwdc2017/719/)
- [CarPlay - Human Interface Guidelines: Audio Apps](https://developer.apple.com/design/human-interface-guidelines/carplay/overview/audio-apps/)
