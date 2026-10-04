## iOS CarPlay｜Поддержка CarPlay в вашем аудио-приложении

Начиная с iOS 14 для разработки аудио-приложений CarPlay можно использовать CarPlay framework (для навигационных приложений он доступен ещё с iOS 12). Он предоставляет набор UI-шаблонов, которые разработчик может настраивать. Если нужна совместимость с iOS 13 и более ранними версиями, придётся использовать MediaPlayer framework, который работает и на старых системах. Поэтому, если вашему приложению нужен CarPlay framework на iOS 14 и выше и при этом нужна поддержка iOS 13 и ниже, придётся поддерживать две кодовые базы, и объём работы может вырасти почти вдвое. Автор поддержал только iOS 14 и выше, поэтому в этой главе подробно разбираются нюансы разработки с CarPlay framework. Если вам нужна поддержка старых версий, посмотрите также конспекты автора по «WWDC17 - Поддержка CarPlay в вашем приложении» и «WWDC18 - Аудио- и навигационные приложения CarPlay».

### Запрос разрешения и настройка проекта

Сначала нужно определить, подходит ли ваше приложение для CarPlay, затем запросить на сайте для разработчиков разрешение CarPlay для соответствующего типа приложения и настроить проект. Только в этом случае ваш проект сможет использовать CarPlay Simulator, иначе он не откроется (пункт будет серым и недоступным). Впрочем, автор заметил, что если CarPlay Simulator уже когда-то был включён, его можно открыть даже для приложения без поддержки CarPlay. Поэтому можно запустить пример от Apple [CarPlay Music App](https://developer.apple.com/documentation/carplay/integrating_carplay_with_your_music_app?language=objc), чтобы активировать CarPlay Simulator.

Но чтобы запускать и отлаживать ваше CarPlay-приложение в CarPlay Simulator, всё равно нужно, чтобы ваш собственный проект поддерживал CarPlay.

Поэтому, если вы планируете разрабатывать CarPlay-приложение, лучше запросить разрешение заранее: проверка в Apple занимает время. Пока ждёте, можно изучить документацию, а когда разрешение будет получено и проект настроен, можно отлаживать приложение в CarPlay Simulator.

Документация: [Запрос entitlements для CarPlay](https://developer.apple.com/documentation/carplay/requesting_the_carplay_entitlements?language=objc)

### Запуск и отладка CarPlay-приложения в CarPlay Simulator

К каждому iPhone Simulator прилагается CarPlay Simulator, который открывается через **I/O > External Displays > CarPlay**. Размер и масштаб окна стандартного CarPlay Simulator по умолчанию: `800 x 480, @2x`.

Для навигационных приложений Apple рекомендует включить дополнительные опции CarPlay Simulator, выполнив в терминале следующую команду. Это позволит перед каждым запуском CarPlay Simulator задавать размер и масштаб окна, чтобы убедиться, что содержимое карты корректно отображается во всех рекомендуемых конфигурациях. Это поддерживается только для навигационных приложений; у аудио-приложений при изменении размера окна отображение получается неудовлетворительным, поэтому включать опцию не рекомендуется.

```
defaults write com.apple.iphonesimulator CarPlayExtraOptions -bool YES
```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d693e45a0ca4c1598fa3b71e5706e45~tplv-k3u1fbpfcp-watermark.image?)

Если вы открыли CarPlay Simulator, но не видите своё приложение на главном экране, возможно, вы забыли добавить соответствующий entitlement. Нужно добавить в Entitlements.plist ключ `com.apple.developer.carplay-audio` со значением 1.

```xml
<key>com.apple.developer.carplay-audio</key>
<true/>
```

Документация: [Запуск и отладка CarPlay-приложения в CarPlay Simulator](https://developer.apple.com/documentation/carplay/using_the_carplay_simulator?language=objc)

На Mac с чипом M1 CarPlay Simulator может не работать. Если Xcode запущен в режиме Rosetta, запуск CarPlay-приложения приводит к crash. Запуск Simulator тоже через Rosetta проблему не решает. Решения этой проблемы пока нет. https://issueexplorer.com/issue/mapbox/mapbox-navigation-ios/3355。

### Объявление CarPlay scene

Объявите scene в Info.plist.

```xml
<key>UIApplicationSceneManifest</key>
<dict>
    <key>UIApplicationSupportsMultipleScenes</key>
		<false/>
		<key>UISceneConfigurations</key>
		<dict>
			<key>CPTemplateApplicationSceneSessionRoleApplication</key>
			<array>
			    <dict>
			        <key>UISceneClassName</key>
			        <string>CPTemplateApplicationScene</string>
			        <key>UISceneConfigurationName</key>
			        <string>CarPlaySceneConfiguration</string>
			        <key>UISceneDelegateClassName</key>
			        <string>$(PRODUCT_MODULE_NAME).CarPlaySceneDelegate</string>
			    </dict>
			</array>
    </dict>
</dict>
```

### Реализация CarPlaySceneDelegate

```swift
import CarPlay

class CarPlaySceneDelegate: UIResponder, CPTemplateApplicationSceneDelegate {
    var interfaceController: CPInterfaceController?

    func templateApplicationScene(_ templateApplicationScene: CPTemplateApplicationScene,
            didConnect interfaceController: CPInterfaceController) {

        self.interfaceController = interfaceController
        let item = CPListItem(text: "Rubber Soul", detailText: "The Beatles")
        let section = CPListSection(items: [item])
        let listTemplate = CPListTemplate(title: "Albums", sections: [section])
        interfaceController.setRootTemplate(listTemplate, animated: true)
    }

    func templateApplicationScene(_ templateApplicationScene: CPTemplateApplicationScene,
            didDisconnect interfaceController: CPInterfaceController) {
        self.interfaceController = nil
    }
}
```

Протокол CPTemplateApplicationSceneDelegate определяет методы, которые вызываются при подключении и отключении сцены CarPlay, а также при некоторых действиях пользователя. Корневой шаблон нужно создать и установить в тот момент, когда CarPlay запускает ваше приложение и подключает его сцену. Обычно реализуют два следующих метода:

* [- templateApplicationScene:didConnectInterfaceController:](https://developer.apple.com/documentation/carplay/cptemplateapplicationscenedelegate/3578119-templateapplicationscene?language=objc)

  Уведомляет делегат о том, что CarPlay Scene подключена. Когда приложение запускается на экране автомобиля, CarPlay framework вызывает этот метод, и в нём выполняется инициализация шаблонов. Как видно, система автоматически создаёт экземпляр [CPInterfaceController](https://developer.apple.com/documentation/carplay/cpinterfacecontroller/) (аналог UINavigationController), который служит входным контроллером нашего CarPlay-приложения; нам достаточно сохранить этот экземпляр в колбэке. В примере выше мы создаём списочный шаблон CPListTemplate (аналог UITableView), который принимает несколько CPListSection, а каждая секция содержит несколько CPListItem (аналог UITableViewCell). Наконец, мы устанавливаем CPListTemplate корневым шаблоном приложения (аналог rootViewController).

* [- templateApplicationScene:didDisconnectInterfaceController:](https://developer.apple.com/documentation/carplay/cptemplateapplicationscenedelegate/3578120-templateapplicationscene?language=objc)

  Уведомляет делегат о том, что CarPlay Scene отключена. Метод вызывается при отключении от автомобиля, в нём можно выполнить очистку.

Документация: [Отображение контента в вашем CarPlay-приложении](https://developer.apple.com/documentation/carplay/displaying_content_in_carplay?language=objc)

### Построение интерфейса CarPlay

Интерфейс CarPlay-приложения по сути состоит из Template и Item, а для аудио-приложений CarPlay обычно достаточно CPTabBarTemplate, CPListTemplate, CPListImageRowItem, CPListItem и т. п.

| Templates                                                    | Description                                                  |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| [CPListTemplate](https://developer.apple.com/documentation/carplay/cplisttemplate?language=objc)<br />- [CPListItem](https://developer.apple.com/documentation/carplay/cplistitem?language=objc)<br />- [CPListImageRowItem](https://developer.apple.com/documentation/carplay/cplistimagerowitem?language=objc)<br />- [CPMessageListItem](https://developer.apple.com/documentation/carplay/cpmessagelistitem?language=objc) | Шаблон списка (аналог UITableView)<br />- Универсальный выбираемый элемент списка (на рисунке ниже — второй)<br />- Элемент списка, отображающий набор изображений (на рисунке ниже — третий)<br />- Элемент списка, представляющий беседу или контакт (для приложений связи) (на рисунке ниже — первый) |
| [CPGridTemplate](https://developer.apple.com/documentation/carplay/cpgridtemplate?language=objc) | Шаблон для отображения и управления сеткой элементов         |
| [CPTabBarTemplate](https://developer.apple.com/documentation/carplay/cptabbartemplate?language=objc) | Шаблон TabBar (аналог UITabBarController)                    |

![](https://docs-assets.developer.apple.com/published/49959584ba/rendered2x-1619630673.png)

#### CPTabBarTemplate

Шаблон TabBar, аналог UITabBarController в UIKit. Инициализируется набором CPTemplate и может быть установлен как rootTemplate у interfaceController.

```swift
let tabBarTemplate = CPTabBarTemplate(templates: templates)
interfaceController.setRootTemplate(tabBarTemplate, animated: true)
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/90920a58c3264fb98908516fffb09449~tplv-k3u1fbpfcp-watermark.image?)

Обратите внимание: количество templates в CPTabBarTemplate ограничено. Максимум можно получить через свойство класса [maximumTabCount](https://developer.apple.com/documentation/carplay/cptabbartemplate/3589351-maximumtabcount/); его значение зависит от entitlements, добавленных в Entitlements.plist. Для аудио-приложения можно добавить не более 4 вкладок, при превышении произойдёт crash.

> В [WWDC17 - Поддержка CarPlay в вашем приложении](Content/WWDC17%20-%20让你的%20App%20支持%20CarPlay%20车载.md) Apple упоминала, что при построении CarPlay-приложения на MediaPlayer framework рекомендуется использовать не более 4 вкладок с короткими заголовками: места мало, а у некоторых автомобилей узкие экраны. Кроме того, во время воспроизведения аудио в правом верхнем углу rootTemplate нужно показывать кнопку «Сейчас играет».

```swift
/**
 The maximum number of tabs that your app may display in a @c CPTabBarTemplate,
 depending on the entitlements that your app declares.

 @warning The system will throw an exception if your app attempts to display more
 than this number of tabs in your tab bar template.
 */
open class var maximumTabCount: Int { get }
```

У каждого template можно задать для вкладки tabTitle и tabImage, а также tabSystemItem, чтобы использовать системный стиль (доступных стилей немного, и текст изменить нельзя). Если tabSystemItem не задан, а tabImage равен nil, то для этого tabBarItem будет использован UITabBarItem.SystemItem.more.

```swift
// Пользовательский стиль tab
listTemplate.tabTitle = "推荐"
listTemplate.tabImage = UIImage(named: "tabbar_recommend")
// Системный стиль tab; если одновременно заданы tabTitle и tabImage, tabSystemItem не применяется
listTemplate.tabSystemItem = .favorites
// Показать красную точку
listTemplate.showsTabBadge = true
```

Здесь придётся идти к UI-дизайнеру за иконками. Документация: [Руководство по UI-дизайну CarPlay](https://developer.apple.com/design/human-interface-guidelines/carplay/icons-and-images/custom-icons/).

У CPTabBarTemplate есть свойство delegate, реализующее протокол [CPTabBarTemplateDelegate](https://developer.apple.com/documentation/carplay/cptabbartemplatedelegate/). В CPTabBarTemplateDelegate всего один метод:

```swift
public protocol CPTabBarTemplateDelegate : NSObjectProtocol {
    func tabBarTemplate(_ tabBarTemplate: CPTabBarTemplate, didSelect selectedTemplate: CPTemplate)
}
```

В этом методе при необходимости можно обновить данные selectedTemplate.

Обратите внимание, что в отличие от `- tabBarController:didSelectViewController:` у UITabBarController:

* Когда CPTabBarTemplate отображается и по умолчанию выбирается первая вкладка, этот метод делегата вызывается один раз, поэтому в его реализации нужно учитывать, не требуется ли отфильтровать первый вызов
* Нажатие на уже выбранную вкладку тоже вызывает метод делегата, поэтому этот случай также стоит учесть и при необходимости отфильтровать

#### CPListTemplate

Шаблон списка, аналог UITableview. Набором CPListTemplate можно инициализировать CPTabBarTemplate. CPListTemplate состоит из элементов, реализующих протокол [CPListTemplateItem](https://developer.apple.com/documentation/carplay/cplisttemplateitem/); элемент аналогичен UITableviewCell. В аудио-приложениях обычно используют CPListItem и CPListImageRowItem.

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c284c0486f8947dd82d8334b1b6c10b0~tplv-k3u1fbpfcp-watermark.image?)

У CPListTemplate есть свойство delegate, реализующее протокол [CPListTemplateDelegate](https://developer.apple.com/documentation/carplay/cplisttemplatedelegate/). В CPListTemplateDelegate всего один метод, он срабатывает при нажатии пользователем на элемент, и в его реализации можно выполнить push других Template.

```swift
@available(iOS, introduced: 12.0, deprecated: 14.0)
protocol CPListTemplateDelegate : NSObjectProtocol {
    func listTemplate(_ listTemplate: CPListTemplate, didSelect item: CPListItem, completionHandler: @escaping () -> Void)
}
```

Обратите внимание на параметр completionHandler. Когда вызывается метод `- listTemplate:didSelectListItem:completionHandler:`, то до вызова completionHandler справа на didSelectListItem показывается индикатор загрузки. Лучшая практика: вызывать его, когда контент для воспроизведения готов или когда переход на страницу завершён (в completion метода `- pushTemplate:animated:completion:`). Также нужно гарантировать, что completionHandler будет вызван в любом случае, например при раннем выходе, иначе индикатор активности останется навсегда.

> `CPListTemplateDelegate` в iOS 14 помечен как устаревший (deprecated); для обработки действий рекомендуется использовать свойство `handler` протокола [CPSelectableListItem](https://developer.apple.com/documentation/carplay/cpselectablelistitem?language=objc) — необязательный блок действия. Протокол `CPSelectableListItem` реализуют CPListItem, CPListImageRowItem и другие.
>
> ```swift
> /**
>  @c CPListSelectable describes list items that accept a list item handler, called when
>  the user selects this list item.
>  */
> @available(iOS 14.0, *)
> public protocol CPSelectableListItem : CPListTemplateItem {
>     /**
>      An optional action block, fired when the user selects this item in a list template.
>      You must call the completion block after processing the user's selection.
>      */
>     var handler: ((CPSelectableListItem, @escaping () -> Void) -> Void)? { get set }
> }
> ```

#### CPListImageRowItem

Можно использовать для отображения блока, состоящего из набора альбомов. Инициализируется текстом и набором изображений: текст может показывать название блока, а изображения — обложки первых n альбомов блока.

```swift
let imagesCount = CPMaximumNumberOfGridImages // зависит от доступной ширины экрана автомобиля
let images = Array(repeating: image, count: imagesCount)
let listImageRowItem = CPListImageRowItem(text: text, images: images)
```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0c4ef7bb3d374843b3daf25ae649fb4f~tplv-k3u1fbpfcp-watermark.image?)

Область нажатия CPListImageRowItem делится на область каждого изображения и всю остальную область, кроме изображений.

Действие для каждого изображения обрабатывается через свойство listImageRowHandler экземпляра CPListImageRowItem. По нажатию можно выполнить push на страницу со списком аудио этого альбома.

```swift
var listImageRowHandler: ((CPListImageRowItem, Int, @escaping () -> Void) -> Void)? // The image row item that the user selected.
```

Действие для области вне изображений обрабатывается через свойство handler экземпляра CPListImageRowItem. По нажатию можно выполнить push на страницу со списком альбомов этого блока.

```swift
var handler: ((CPSelectableListItem, @escaping () -> Void) -> Void)?
```

#### CPListItem

Можно использовать для отображения альбома или аудио, а также других элементов, например «Загрузка...», «Истории воспроизведения пока нет», «Воспроизвести всё» и т. п.

Элемент аудио обычно состоит из обложки, названия и описания аудио; для инициализации CPListItem можно использовать следующий конструктор.

```swift
let listItem = CPListItem(text: audioName, 
                          detailText: audioDesc, 
                          image: audioCover)
```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/125d842edab74a5d97cddbe2edbd2357~tplv-k3u1fbpfcp-watermark.image?)

Элементу альбома, чтобы отличать его от элемента аудио, нужна ещё стрелка навигации справа, показывающая, что нажатие откроет страницу со списком аудио. Для инициализации CPListItem можно использовать следующий конструктор.

```swift
let listItem = CPListItem(text: albumName, 
                          detailText: albumDesc, 
                          image: albumCover, 
                          accessoryImage: nil, 
                          accessoryType: .disclosureIndicator)
```

Для иконки справа есть два системных стиля (стрелка, облако), также поддерживается пользовательская иконка.

```swift
enum CPListItemAccessoryType : Int {
    case none = 0
    case disclosureIndicator = 1
    case cloud = 2
}
```

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1d384991a82f4deba5f3fe53d9fc2f56~tplv-k3u1fbpfcp-watermark.image?)

Обратите внимание: если detailText у CPListItem равен nil, высота view уменьшается, а text выравнивается по центру (см. рисунок ниже), что портит внешний вид и единообразие. Поэтому, если у аудио нет подзаголовка, можно заполнять detailText длительностью аудио или другими данными.

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0aaab77e2bc146649b32fe54b06b4c2b~tplv-k3u1fbpfcp-watermark.image?)

#### CPNowPlayingTemplate

Экран воспроизведения — самая важная страница аудио-приложения CarPlay. Для него используется [CPNowPlayingTemplate](https://developer.apple.com/documentation/carplay/cpnowplayingtemplate/), это синглтон.

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95c7eb8bdac4467396a7c9dce25a11a8~tplv-k3u1fbpfcp-watermark.image?)

CPNowPlayingTemplate можно настроить под свои нужды, например добавить кнопки управления. Можно использовать системные кнопки, предоставляемые CarPlay framework, или создать собственные. Обратите внимание: CPNowPlayingTemplate нужно настроить уже в момент `- templateApplicationScene:didConnectInterfaceController:`, а не в момент push на CPNowPlayingTemplate, потому что переход на CPNowPlayingTemplate не обязательно происходит через явный push: он может быть вызван и через приложение «Сейчас играет», и через кнопку «Сейчас играет» в правом верхнем углу rootTemplate.

```swift
let nowPlayingTemplate = CPNowPlayingTemplate.shared
let repeatButton = CPNowPlayingRepeatButton() { ... }
let playbackRateButton = CPNowPlayingPlaybackRateButton() { ... }
nowPlayingTemplate.updateNowPlayingButtons([repeatButton, playbackRateButton])
```

Независимо от того, построено ли ваше CarPlay-приложение на CarPlay framework или на MediaPlayer framework, информация об аудио на экране воспроизведения предоставляется через [MPNowPlayingInfoCenter](https://developer.apple.com/documentation/mediaplayer/mpnowplayinginfocenter/), а реакция на удалённые команды управления воспроизведением — через [MPRemoteCommandCenter](https://developer.apple.com/documentation/mediaplayer/mpremotecommandcenter/). Разница лишь в том, что в CarPlay framework часть событий удалённого управления, например режим повтора и скорость воспроизведения, обрабатывается через handler у [CPNowPlayingButton](https://developer.apple.com/documentation/carplay/cpnowplayingbutton/). Если ваше приложение аудио, то эти функции у него, скорее всего, уже поддерживаются, ведь информация о воспроизведении и управление на экране блокировки iPhone и в Пункте управления тоже работают через них. Поэтому нам нужно только оптимизировать или расширить функциональность для CarPlay. Конкретно нужно сделать следующее:

* Задавать и обновлять [nowPlayingInfo](https://developer.apple.com/documentation/mediaplayer/mpnowplayinginfocenter/1615903-nowplayinginfo) у MPNowPlayingInfoCenter, где содержатся метаданные текущего аудио: название, автор, длительность и т. п. Когда это делать:

  * При смене аудио (предыдущее, следующее и т. д.)
  * При паузе, возобновлении, остановке воспроизведения
  * При seek (пропуск заставки, перемотка по прогрессу и т. д.)
  * При изменении скорости воспроизведения (состояние отображения кнопки скорости в CPNowPlayingTemplate). Если текущее аудио не воспроизводится, скорость воспроизведения нужно установить в 0
  * ...

```swift
import MediaPlayer
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

Обратите внимание: прогресс воспроизведения, то есть уже проигранное время текущего аудио, система вычисляет автоматически на основе ранее предоставленных **проигранного времени** и **скорости воспроизведения**. Поэтому часто обновлять nowPlayingInfo не нужно и не рекомендуется: это дорогая операция.

* Помимо nowPlayingInfo, часть состояния нужно синхронизировать с CarPlay-приложением другими способами, например:

  * Состояние воспроизведения аудио: пауза/воспроизведение (состояние отображения кнопки воспроизведения в CPNowPlayingTemplate)

  ```objectivec
  typedef NS_ENUM(NSUInteger, MPNowPlayingPlaybackState) {
      MPNowPlayingPlaybackStateUnknown = 0,
      MPNowPlayingPlaybackStatePlaying,
      MPNowPlayingPlaybackStatePaused,
      MPNowPlayingPlaybackStateStopped,
      MPNowPlayingPlaybackStateInterrupted
  };
  
  if (@available(iOS 13.0, *)) {
      MPNowPlayingInfoCenter.defaultCenter.playbackState = MPNowPlayingPlaybackStatePlaying; 
  }
  ```
  
  * Состояние режима повтора: повтор списка/повтор одного трека (состояние отображения кнопки режима повтора в CPNowPlayingTemplate)
  
  ```objectivec
  typedef NS_ENUM(NSInteger, MPRepeatType) {
      MPRepeatTypeOff,    /// Nothing is repeated during playback.
      MPRepeatTypeOne,    /// Repeat a single item indefinitely.
      MPRepeatTypeAll,    /// Repeat the current container or playlist indefinitely.
  };
  
  MPRemoteCommandCenter.sharedCommandCenter.changeRepeatModeCommand.currentRepeatType = MPRepeatTypeOne;
  ```

Обложка аудио, название, описание, длительность, текущий прогресс, режим повтора, скорость воспроизведения и другая информация на рисунке ниже синхронизируются с CPNowPlayingTemplate именно описанными выше способами. Ваше аудио-приложение, вероятно, уже реализовало эту функциональность для синхронизации информации о воспроизведении с экраном блокировки iPhone и Пунктом управления, поэтому сейчас достаточно проверить, что информация о воспроизведении, отображаемая в CPNowPlayingTemplate вашего CarPlay-приложения, корректна.

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e90b21bcb08844f28653fec9258721c3~tplv-k3u1fbpfcp-watermark.image?)

* Реагировать на события MPRemoteCommandCenter, то есть на удалённые команды управления воспроизведением: воспроизведение, пауза, смена трека и т. д.

  * playCommand
  * pauseCommand
  * previousTrackCommand
  * nextTrackCommand
  * togglePlayPauseCommand
  * changeRepeatModeCommand
  * changePlaybackRateCommand
  * ...
* При использовании CarPlay framework удалённые команды changeRepeatModeCommand и changePlaybackRateCommand больше не обрабатываются через target-action, а обрабатываются через handler у [CPNowPlayingRepeatButton](https://developer.apple.com/documentation/carplay/cpnowplayingrepeatbutton/) и [CPNowPlayingPlaybackRateButton](https://developer.apple.com/documentation/carplay/cpnowplayingplaybackratebutton), но command.enabled всё равно нужно включить. Например:
  * Когда command.enabled равен true и пользователь нажимает кнопку режима повтора, срабатывает handler у CPNowPlayingRepeatButton; в нём можно обновить режим повтора в приложении и синхронизировать его состояние с CarPlay описанным выше способом. Если ваше приложение также поддерживает режим случайного воспроизведения, можно добавить CPNowPlayingShuffleButton и включить changeShuffleModeCommand, чтобы вместе с CPNowPlayingRepeatButton переключать 3 режима.
  * Поскольку известно только то, что пользователь нажал кнопку, но не его конкретное намерение, лучшая практика для handler у CPNowPlayingPlaybackRateButton такова: задать диапазон скоростей воспроизведения, при каждом нажатии увеличивать скорость в приложении и синхронизировать её с CarPlay через обновление nowPlayingInfo. Если текущее аудио уже воспроизводится на максимальной поддерживаемой скорости, то при следующем нажатии скорость сбрасывается на минимальную, и так по кругу.

### Лучшие практики

#### Хранение данных в userInfo

У CPListItem и CPListImageRowItem есть свойство [userInfo](https://developer.apple.com/documentation/carplay/cplistitem/2977574-userinfo?language=objc), предназначенное для хранения данных. Отношение `CPListItem -> userInfo` аналогично `UITableViewCell -> model`.

```swift
// Use this property to attach a value that provides additional context to the list item. For example, you can attach a model object and reference it in the list item’s handler when processing the selection.
var userInfo: Any?
```

#### Управление интерактивностью item через isEnabled (iOS 15)

У CPListItem и CPListImageRowItem есть свойство [isEnabled](https://developer.apple.com/documentation/carplay/cplistitem/3751895-enabled?language=objc), задающее интерактивность item (по умолчанию true). Item с isEnabled = false отображается серым и не реагирует на нажатия, то есть не вызывает его `handler` или метод `- listTemplate:didSelectListItem:completionHandler:` у CPListTemplateDelegate. Лучшая практика: устанавливать isEnabled = false для item, которые сами по себе не интерактивны, например «Истории воспроизведения пока нет» или «Загрузка...», так UI выглядит лучше; однако этот API поддерживается только начиная с iOS 15.

```swift
// A Boolean value that indicates if the item is enabled.
@available(iOS 15.0, *)
var isEnabled: Bool
```

#### Обработка нажатий на item через handler

CPListItem и CPListImageRowItem реализуют протокол `CPSelectableListItem`, у которого есть свойство [handler](https://developer.apple.com/documentation/carplay/cplistitem/3667716-handler?language=objc) для обработки нажатий на item. Если для item задан handler, то нажатие вызывает handler, а не методы CPListTemplateDelegate. CPListTemplateDelegate в iOS 14 помечен как устаревший (deprecated), поэтому для обработки действий рекомендуется использовать handler.

```swift
/**
 @c CPListSelectable describes list items that accept a list item handler, called when
 the user selects this list item.
 */
@available(iOS 14.0, *)
public protocol CPSelectableListItem : CPListTemplateItem {
    /**
     An optional action block, fired when the user selects this item in a list template.
     You must call the completion block after processing the user's selection.
     */
    var handler: ((CPSelectableListItem, @escaping () -> Void) -> Void)? { get set }
}
```

У handler по сравнению с обработкой действий через CPListTemplateDelegate есть преимущество: handler привязан к конкретному item, а CPListTemplateDelegate относится ко всем item в CPListTemplate. При использовании CPListTemplateDelegate нам пришлось бы делать guard для item вроде «Истории воспроизведения пока нет» или «Загрузка...», а с handler действия таких item можно обрабатывать отдельно или не обрабатывать вовсе.

```swift
let item = CPListItem(text: "正在加载中", detailText: nil)
if #available(iOS 15.0, *) {
    item.isEnabled = false
} else {
    item.handler = { (item, completionHandler) in
        completionHandler()
    }
}
```

#### Индикатор воспроизведения через свойство isPlaying

Если CPListItem используется для отображения аудио, то с помощью свойства [isPlaying](https://developer.apple.com/documentation/carplay/cplistitem/3551780-isplaying) можно показать индикатор воспроизведения.

```swift
var isPlaying: Bool
```

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/33d26d619a764d15a0449063b6c531b4~tplv-k3u1fbpfcp-watermark.image?)

По умолчанию индикатор находится слева и скрывает image. Через свойство [playingIndicatorLocation](https://developer.apple.com/documentation/carplay/cplistitem/3566414-playingindicatorlocation?language=objc) можно переместить индикатор вправо.

```swift
enum CPListItemPlayingIndicatorLocation : Int {
    case leading = 0
    case trailing = 1
}

var playingIndicatorLocation: CPListItemPlayingIndicatorLocation
```

#### Поддержка открытия текущего плейлиста

![](https://gitee.com/junteng/images/raw/master/img/20211222110753.jpg)

Удобный экран воспроизведения аудио-приложения должен позволять открыть текущий плейлист для удобного переключения треков, и CarPlay-приложение тоже должно это поддерживать. Особенно если альбом воспроизводился через iPhone-приложение, а в CarPlay-приложении данных об этом альбоме нет (данные в CarPlay и в iPhone-приложении могут различаться): тогда, чтобы послушать другие треки альбома, пользователь может переключаться только кнопками «Предыдущий/Следующий» или через «плейлист iPhone-приложения». Поэтому очень хорошо, если на экране воспроизведения CarPlay-приложения можно открыть текущий плейлист.

CPNowPlayingTemplate поддерживает кнопку открытия текущего плейлиста в правом верхнем углу; по нажатию выполняется push CPListTemplate с текущим плейлистом.

```swift
let nowPlayingTemplate = CPNowPlayingTemplate.shared
nowPlayingTemplate.isUpNextButtonEnabled = true
nowPlayingTemplate.upNextTitle = "播放列表"
nowPlayingTemplate.add(observer)

ObserverClass: CPNowPlayingTemplateObserver {
    func nowPlayingTemplateUpNextButtonTapped(_ nowPlayingTemplate: CPNowPlayingTemplate) {
        interfaceController.pushTemplate(aListTemplate, animated: true)
    }
}
```

### Переходы между страницами

Помните [CPInterfaceController](https://developer.apple.com/documentation/carplay/cpinterfacecontroller/) в точке входа CarPlay-приложения [- templateApplicationScene:didConnectInterfaceController:](https://developer.apple.com/documentation/carplay/cptemplateapplicationscenedelegate/3578119-templateapplicationscene?language=objc)? Он служит входным контроллером нашего CarPlay-приложения, и ему мы присваиваем template в качестве rootTemplate. Переходы между страницами тоже выполняются через него: он похож на UINavigationController и поддерживает push, pop, present, dismiss и т. д. (present и dismiss — только для CPActionSheetTemplate, CPVoiceControlTemplate, CPAlertTemplate). Для аудио-приложения обычно хватает push, а на дочерних страницах кнопка «Назад» в левом верхнем углу есть по умолчанию.

### Архитектура кода

Можно ориентироваться на следующую схему:

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2f1c43bf92714e8e8dde65bd14dc3089~tplv-k3u1fbpfcp-watermark.image?)

### Изображения

#### Иконки и изображения

Посмотрите [CarPlay - Руководство по дизайну](https://developer.apple.com/design/human-interface-guidelines/carplay/overview/introduction/) и отправьте его своему PM и UI-дизайнеру.

#### Тёмный/светлый режим

Подход к адаптации такой же, как в iPhone-приложении: если вашему приложению нужны оба стиля, поддержите их.

#### Асинхронные изображения

* CarPlay не поддерживает GIF-изображения, их использование приведёт к crash. Автор пробовал извлекать из GIF первый кадр, но он всё равно не поддерживался; возможно, автор делал что-то не так.
* В asyncImage тоже нужно учитывать scale, иначе изображение будет размытым.
* Для CPListItem и CPListImageRowItem можно добавить расширения с методами asyncImage для удобства использования.

```swift
@available(iOS 14, *)
protocol CPAsyncImage {
    func loadImage(with url: URL?, complete: @escaping (UIImage?) -> (Void))
    static var placeholderImage: UIImage { get }
}

@available(iOS 14, *)
extension CPAsyncImage {
    
    func loadImage(with url: URL?, complete: @escaping (UIImage?) -> (Void)) {
        
        guard let url = url else {
            complete(nil)
            return
        }
        
        // Здесь также можно выполнить фильтрацию по urlType
        
        SDWebImageManager.shared.loadImage(with: url, options: .retryFailed, progress: nil) { image, data, error, type, finished, imageURL in
            
            guard var image = image, image.images == nil else {
                complete(nil)
                return
            }
                   
            if let cgImage = image.cgImage {
                let screen = TTScene.carPlay?.value(forKey: "screen") as? UIScreen
                let scale = screen?.scale ?? 1
                image = UIImage(cgImage: cgImage, scale: scale, orientation: .up)
            }
            complete(image)
        }
    }
    
    static var placeholderImage: UIImage {
        UIImage(named: "CP_default") ?? UIImage()
    }
}

@available(iOS 14, *)
extension CPListItem: CPAsyncImage {
    
    func asyncImage(with url: URL?, placeholderImage: UIImage? = CPListItem.placeholderImage, complete: ((UIImage?) -> (Void))? = nil) {
        
        self.setImage(placeholderImage)
        loadImage(with: url) { [weak self] image in
            guard let image = image else { return }
            self?.setImage(image)
            complete?(image)
        }
    }
}

@available(iOS 14, *)
extension CPListImageRowItem: CPAsyncImage {
    
    func asyncImage(with urls: [URL?], placeholderImage: UIImage = CPListImageRowItem.placeholderImage, complete: (((index: Int, image: UIImage?)) -> (Void))? = nil) {
        
        var images = Array(repeating: placeholderImage, count: urls.count)
        self.update(images)
        
        for (index, url) in urls.enumerated() {
            loadImage(with: url) { [weak self] image in
                guard let image = image else { return }
                images[index] = image
                self?.update(images)
                complete?((index, image))
            }
        }
    }
}
```

### Перезагрузка данных

Если запустить CarPlay-приложение при плохой сети, возможен тайм-аут запроса, и в CarPlay-приложении не будет данных. Есть несколько способов это обработать:

1. Не перезагружать. Если это главная страница, пользователю придётся перезапустить CarPlay-приложение, чтобы данные загрузились снова; если это дочерняя страница, пользователю придётся выйти из неё и зайти снова. CarPlay-приложение NetEase Cloud Music (Wangyi Yun) обрабатывает это именно так, но на его главной странице есть несколько фиксированных item, поэтому на опыт это не влияет. Если на вашей главной странице нет локальных item, так делать не рекомендуется;
2. Если rootTemplate — CPTabBarTemplate, можно выполнять перезагрузку selectedTemplate в момент `- tabBarTemplate:didSelectTemplate:`;
3. При втором способе перезагрузка для пользователя незаметна, потому что самому добавить индикатор активности нельзя или неудобно. Мой вариант: на время запроса данных добавлять loading item (например, CPListItem с текстом «Загрузка...»). Если запрос завершился ошибкой, обновить item до failure item (например, CPListItem с текстом «Не удалось загрузить, нажмите для повтора»). Когда пользователь нажимает на failure item, он снова превращается в loading item и данные запрашиваются заново. Так можно поступать на каждой странице, которой нужны данные с сервера.

Нужно ли обновление? Если вы хотите разрешить обновлять данные в процессе использования CarPlay-приложения, можно применить второй способ. Однако пользователь обычно пользуется CarPlay за один раз недолго, обновление не нужно: свежие данные можно загрузить при следующем запуске. Поэтому я сделал перезагрузку только для случая, когда не удалась первая загрузка данных.

### Внимание к работе при слабой сети и без сети

В материалах WWDC и в документации Apple не раз упоминает, что нужно заботиться о пользовательском опыте при слабой сети или её отсутствии, ведь во время поездки автомобиль может проезжать участки или районы с плохим покрытием. Например, упомянутая выше проблема «тайм-аут запроса, перезагрузка данных», а также корректность синхронизации данных в CPNowPlayingTemplate и событий управления воспроизведением и т. д.

### Siri

Даже если ваше приложение не поддерживает SiriKit, переключение треков, пауза и возобновление воспроизведения через Siri всё равно будут работать, потому что эти события удалённого управления поддерживают Siri изначально.

### Тестирование

Тестируйте в реальных условиях (в автомобиле). В [Apple｜Запуск и отладка CarPlay-приложения в CarPlay Simulator](https://developer.apple.com/documentation/carplay/using_the_carplay_simulator?language=objc) перечислены функции, которые нельзя протестировать в CarPlay Simulator.

Для аудио-приложений у Simulator есть ограничения в состоянии воспроизведения: он не отражает реальный пользовательский опыт.

Чтобы полноценно отлаживать приложение с помощью LLDB, начиная с Xcode 9 поддерживается беспроводная отладка, так что приложение можно отлаживать, пока iPhone подключён к автомобилю. Подробнее можно посмотреть в [WWDC19 - Отладка с Xcode 9](https://developer.apple.com/wwdc17/404).

## Важные моменты и лучшие практики

* Проблема синглтонов. CarPlay использует классы-синглтоны: если CarPlay-приложение закрыто, а iPhone-приложение нет, процесс продолжает жить, синглтон не освобождён, и это может приводить к проблемам. Можно инициализировать синглтон в `- templateApplicationScene:didConnectInterfaceController:`, а освобождать в `- templateApplicationScene:didDisconnectInterfaceController:`.
* Язык CarPlay следует языку iPhone, в Simulator так же.
* CarPlay framework нужно сделать слабо подключаемым (weak link, optional) (Target > Build phases > Link Binary With Libraries), иначе запуск приложения на iOS ниже 12 приведёт к crash.
* При отключении CarPlay рекомендуется ставить музыку на паузу.
* При отключении CarPlay можно проверить через Memory Graph, нет ли утечек памяти.
* Страница Template должна отображать хотя бы один элемент, особенно если вы не используете CPTabBarTemplate в качестве rootTemplate; иначе страница будет пустой, что ухудшает опыт. Например, на странице недавно воспроизведённого, когда истории воспроизведения нет, добавьте CPListItem с текстом «Сейчас нет истории воспроизведения».
* Учитывайте такие ситуации, как блокировка iPhone или то, что пользователь не вошёл в аккаунт: ваше приложение должно по-прежнему отлично работать в CarPlay.
