![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167921395-assets/web-upload/4770bd80-5f39-4b0a-9d6e-c607daa3f30b.png)

## Введение

Эта статья — конспект сессии [WWDC20｜10635 - Accelerate your app with CarPlay](https://developer.apple.com/wwdc20/10635). Если вы собираетесь создавать CarPlay app, эту сессию нельзя пропускать. В iOS 14 Apple представила для CarPlay масштабное обновление: появились три новых типа приложений с поддержкой CarPlay — зарядка электромобилей, парковка и быстрый заказ еды, а CarPlay framework впервые стал доступен не только навигационным приложениям. В статье рассказывается о новых template в CarPlay framework и об улучшениях старых, а также о том, как с их помощью разрабатывать CarPlay app всех поддерживаемых типов.

Статья по возможности воспроизводит содержание сессии и лишь слегка дополняет его. Если хотите узнать о CarPlay больше, прочитайте другие статьи автора из рубрики о CarPlay.

## Обзор

Впервые Apple представила CarPlay framework в iOS 12, причём только для разработки навигационных приложений. Возможно, вы пользовались в CarPlay хорошими сторонними навигаторами, например Waze и Google Maps — они построены на template из CarPlay framework.

В iOS 14 Apple приносит в CarPlay следующие обновления:

* Добавлены три новых типа приложений с поддержкой CarPlay: зарядка электромобилей, парковка и быстрый заказ еды
* CarPlay framework впервые стал доступен не только навигационным приложениям. Помимо новых типов, с его помощью можно обновить и ваши аудио- и коммуникационные CarPlay app
* Для разных типов CarPlay app добавлены новые общие и специализированные template
* Ряд старых template улучшен, чтобы они могли выполнять разные роли в CarPlay app разных типов

**Примечание** | Для удобства чтения статья разбита на разделы по типам приложений: можно читать только об обновлениях CarPlay для интересующего вас типа.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167090875-assets/web-upload/ec98a7eb-0e9a-4253-9f93-c4712cdbfa0c.png)

## Принципы дизайна

Сначала вспомним несколько принципов дизайна, о которых нужно помнить при создании app для CarPlay.

* **Проектируйте для водителя**. CarPlay создан для водителя, а не для пассажиров, поэтому в CarPlay app стоит предоставлять только функции, которые помогают вести машину.
* **Упрощайте взаимодействие**. CarPlay app узконаправленны: ваше приложение должно запускаться с самого частого сценария, а каждое действие должно быть настолько простым, чтобы водитель мог выполнить его за несколько секунд.
* **Переиспользуйте конфигурацию app**. По возможности используйте настройки iPhone app, чтобы свести к минимуму настройку, которую пользователю приходится делать в CarPlay. Первое знакомство пользователя с вашим приложением должно состояться на iPhone ещё до начала поездки.
* **Уберите из app любую логику, зависящую от запуска на iPhone**. Приложение может запускаться сначала и только в CarPlay. Когда пользователь нажимает на ваше приложение на главном экране CarPlay, оно запускается и подключается к car scene, а не к iPhone scene. Поэтому важно убрать любую логику, зависящую от запуска на iPhone. С UIScene это легко сделать. Более того, ваше приложение обязано использовать UIScene, чтобы работать с CarPlay framework. Значит, нужно перейти с традиционных API UIWindow и UIApplicationDelegate на UIScene.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167097487-assets/web-upload/4af1e461-357d-4b9b-ba4d-5b5fedd8ae75.png)

## Первое знакомство с CarPlay framework

Как уже сказано, чтобы использовать CarPlay framework, приложение должно использовать UIScene. Важная часть перехода на UIScene — объявление манифеста сцен (Scene Manifest) в info.plist вашего приложения.

Помимо iPhone scene, в манифесте нужно указать конфигурацию CarPlay scene, как показано в примере ниже.

```xml
// CarPlay Scene Manifest

<key>UIApplicationSceneManifest</key>
<dict>
		<key>UISceneConfigurations</key>
  	<dict>
  			<key>CPTemplateApplicationSceneSessionRoleApplication</key>
  			<array>
   					<dict>
   				 		  <key>UISceneClassName</key>
    						<string>CPTemplateApplicationScene</string>
    						<key>UISceneConfigurationName</key>
    						<string>MyApp—Car</string>
    						<key>UISceneDelegateClassName</key>
    						<string>MyApp.CarPlaySceneDelegate</string>
   					</dict>
  			</array>
 		</dict>
</dict>
```

**Примечание** | Если вы создаёте навигационный CarPlay app, можно также указать конфигурацию scene для CarPlay dashboard.

В конфигурации CarPlay scene нужно указать имя класса, который в вашем приложении выступает как scene delegate для CarPlay scene.

Затем нужно реализовать указанный в конфигурации класс-делегат и выполнить следующие шаги:

* Реализуйте метод `didConnect` протокола CPTemplateApplicationSceneDelegate. CarPlay framework вызывает его, когда ваше приложение запускается на экране CarPlay.
  * Во-первых, вероятно, стоит сохранить объект CPInterfaceController из параметров метода, потому что он понадобится позже.
  * Во-вторых, здесь экземпляр CPListTemplate устанавливается как rootTemplate приложения.
* Реализуйте метод `didDisConnect` и в нём освободите сохранённый ранее объект CPInterfaceController.

Пример кода:

```swift
// CarPlay App Lifecycle

import CarPlay

class CarPlaySceneDelegate: UIResponder, CPTemplateApplicationSceneDelegate {
    var interfaceController: CPInterfaceController?
   
    func templateApplicationScene(_ templateApplicationScene: CPTemplateApplicationScene,
            didConnect interfaceController: CPInterfaceController) {

        self.interfaceController = interfaceController
        let listTemplate: CPListTemplate = ...
        interfaceController.setRootTemplate(listTemplate, animated: true)
    }

  	func templateApplicationScene(_ templateApplicationScene: CPTemplateApplicationScene,
            didDisconnect interfaceController: CPInterfaceController) {
    		self.interfaceController = nil
		}
}
```

CPListTemplate — это template в виде списка. CPListItem — базовый элемент, из которых состоит CPListTemplate. Каждый CPListItem соответствует одной строке в CPListTemplate; их отношение похоже на отношение UITableViewCell к UITableView.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167088589-assets/web-upload/f885fa2d-3d66-49c6-bb15-d5c9e8426f6f.png)

Пример создания CPListTemplate:

```swift
// CPListTemplate

import CarPlay

let item = CPListItem(text: "Rubber Soul", detailText: "The Beatles") 
let section = CPListSection(items: [item]) 
let listTemplate = CPListTemplate(title: "Albums", sections: [section]) 
self.interfaceController.pushTemplate(listTemplate, animated: true)
```

Когда пользователь нажимает на CPListItem, вызывается блок listItemHandler этого CPListItem. Он принимает два параметра: нажатый listItem и completion block. При нажатии на listItem ваше приложение может выполнять разные задачи, например:

* в аудио CarPlay app — воспроизвести аудио, соответствующее нажатому listItem
* перейти к новому template

Когда listItem нажат, на экране CarPlay появляется индикатор загрузки, сообщающий пользователю, что приложение сейчас загружается. После вызова completion block индикатор исчезает. Поэтому, какую бы задачу вы ни выполняли, completion block нужно обязательно вызвать.

```swift
// CPListTemplate

import CarPlay

let item = CPListItem(text: "Rubber Soul", detailText: "The Beatles") 
item.listItemHandler = { item, completion, [weak self] in
    // Start playback, then...
    self?.interfaceController.pushTemplate(CPNowPlayingTemplate.shared, animated: true)
    completion()
}
```

## Template

С CarPlay framework ваш CarPlay app строится на template (шаблонах). Template — это способ показать UI в CarPlay app. Ваше CarPlay app отвечает за данные, а Template от вашего имени отрисовывает UI на дисплее автомобиля.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658053819343-assets/web-upload/93eeafa8-d444-4af3-8af7-b874decb0159.png)

Рекомендуем прочитать [iOS CarPlay｜WWDC22 - Расширяем возможности вашего App с помощью CarPlay - Template](https://juejin.cn/post/7114239495360233479#heading-2), чтобы узнать о Template подробнее.

## Все типы App

### CPListTemplate (улучшен)

#### CPListItem (улучшен)

В iOS 14 listItem поддерживают динамическое обновление, в том числе CPListItem, CPListImageRowItem и другие. Многие свойства CPListItem, которые раньше были readonly, теперь readwrite. Это удобно; вот примеры для аудио CarPlay app:

* Обложку альбома нужно получать из сети: можно сначала задать listItem placeholderImage, а когда сетевое изображение загрузится, установить его в listItem заново. CarPlay автоматически перезагрузит только те listItem в CPListTemplate, которые требуют обновления.

```swift
// CPListTemplate

import CarPlay

let item = CPListItem(text: "Rubber Soul", detailText: "The Beatles") 
let section = CPListSection(items: [item]) 
let listTemplate = CPListTemplate(title: "Albums", sections: [section]) 
self.interfaceController.pushTemplate(listTemplate, animated: true)

// Later...
item.image = ...
```

* Apple Podcasts (приложение для подкастов) использует динамическое обновление, чтобы индикатор прогресса менялся по мере воспроизведения аудио.

#### CPListImageRowItem (новый)

CPListImageRowItem — новый в iOS 14 listItem в виде сетки; он поддерживает отображение заголовка и набора изображений.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167088163-assets/web-upload/055dcf3d-a4bb-48c5-bae8-c8028339481f.png)

### CPTabbarTemplate (новый)

CPTabbarTemplate — новый template в iOS 14. Это контейнерный шаблон, который показывает несколько других template в интерфейсе в стиле tab bar, чтобы пользователь видел всё сразу. CPTabbarTemplate хорошо подходит на роль rootTemplate для CarPlay interfaceController.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658067080784-assets/web-upload/1432eb45-fe09-4dd0-b117-093be72aca78.png)

Рассмотрим на примере кода, как создать CPTabbarTemplate.

* Сначала мы создаём два template: CPListTemplate и CPGridTemplate. В iOS 14 каждый template получил новые свойства, позволяющие настроить его отображение в tab bar: tabTitle, tabImage, tabSystemItem, showsTabBadge и другие.
* Затем создаём CPTabbarTemplate из массива template; каждый template в массиве становится одним tab в tab bar.
* Позже CPTabbarTemplate можно обновлять динамически: добавлять новые tab, удалять и переставлять их и так далее. Как показано в примере, также можно показывать и скрывать badge у одного или нескольких tab.

```swift
// CPTabBarTemplate

import CarPlay

let item = CPListItem(text: "Rubber Soul", detailText: "The Beatles") 
let section = CPListSection(items: [item]) 
let favorites = CPListTemplate(title: "Albums", sections: [section])
favorites.tabSystemItem = .favorites
favorites.showsTabBadge = true

let albums: CPGridTemplate = ...
albums.tabTitle = "Albums"
albums.tabImage = ...

let tabBarTemplate = CPTabBarTemplate(templates: [favorites, albums])
self.interfaceController.setRootTemplate(tabBarTemplate, animated: false)

// Later...
favorites.showsTabBadge = false
tabBarTemplate.updateTemplates([favorites, albums])
```

## Аудио App

### Совместное использование MPPlayableContent API и template в одном app

До iOS 14 для разработки аудио CarPlay app использовался MPPlayableContent API из MediaPlayer framework. MPPlayableContent API основан на метаданных: вы описываете данные приложения, например альбомы и песни, а система сама собирает из них древовидный UI. Важно: если вы хотите, чтобы ваше аудио app поддерживало CarPlay в iOS 13 и более ранних версиях, MPPlayableContent API и template могут сосуществовать в одном приложении.

* В iOS 13 и более ранних версиях система запускает ваше CarPlay app через MPPlayableContent API;
* В iOS 14 и новее, если вы предоставили template, система использует их для запуска вашего CarPlay app.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658068581534-assets/web-upload/f09e22bc-3f2e-4502-a803-0c7c6036b2fe.png)

### CPListTemplate (улучшен)

#### CPListItem (улучшен)

Для аудио CarPlay app в CPListItem добавлены новые свойства, например индикатор прогресса воспроизведения [playbackProgress](https://developer.apple.com/documentation/carplay/cplistitem/3551779-playbackprogress) и прыгающая анимация Now Playing [isPlaying](https://link.juejin.cn/?target=https%3A%2F%2Fdeveloper.apple.com%2Fdocumentation%2Fcarplay%2Fcplistitem%2F3551780-isplaying).

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658069090845-assets/web-upload/7485e27e-f747-4d9e-a4e0-7f9ac5b067f2.png)

#### CPListImageRowItem (новый)

CPListImageRowItem — новый в iOS 14 listItem в виде сетки. Apple Books (приложение для книг) использует этот template, чтобы показывать аудиокниги, которые пользователь слушал недавно, как на рисунке ниже.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658069090835-assets/web-upload/ff087095-2084-400c-aa20-ba0245961b1c.png)

Рассмотрим на примере кода, как создать CPListImageRowItem и добавить его в CPListTemplate; это так же просто, как с CPListItem.

* Сначала мы создаём CPListImageRowItem из заголовка и набора изображений.
* У CPListImageRowItem тоже есть блок listItemHandler для обработки нажатий. Он вызывается, когда пользователь нажимает на заголовок в CPListImageRowItem. В Apple Books при этом открывается новый CPListTemplate с другими недавно прослушанными аудиокнигами. Нужно убедиться, что completion block вызван.
* Каждое изображение в CPListImageRowItem можно нажимать независимо; для обработки нажатия на изображение служит блок listItemRowHandler. Здесь тоже нужно убедиться, что completion block вызван. В Apple Books нажатие на изображение открывает соответствующую аудиокнигу и запускает воспроизведение.

```swift
// List Items for Audio Apps

import CarPlay

let gridImages: [UIImage] = ...
let imageRowItem = CPListImageRowItem(text: "Recent Audiobooks", images: gridImages) 

imageRowItem.listItemHandler = { item, completion in
    print("Selected image row header!")
    completion()
}

imageRowItem.listImageRowHandler = { item, index, completion in
    print("Selected artwork at index \(index)!")
    completion()
}

let section = CPListSection(items: [imageRowItem]) 
let listTemplate = CPListTemplate(title: "Listen Now", sections: [section]) 
self.interfaceController.pushTemplate(listTemplate, animated: true)
```

### CPNowPlayingTemplate (новый)

CPNowPlayingTemplate — новый в iOS 14 template для аудио CarPlay app; он является основой аудио CarPlay app.

Пользователи аудио CarPlay app хорошо знакомы с экраном Now Playing. С CPNowPlayingTemplate вы получаете больше контроля над его внешним видом и функциями.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658077486096-assets/web-upload/4840b1ad-a0ec-476c-aa10-7dde9186073d.png)

Ваше приложение может включить кнопки «Очередь воспроизведения» (Up Next) и «Исполнитель» (Artist).

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658077486073-assets/web-upload/45cc9c97-fc3a-44b0-b066-26632fc0be93.png)

Также можно настроить кнопки действий воспроизведения внизу. Можно использовать системные значки (система предоставляет значки для многих распространённых действий воспроизведения) или собственные.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658077486060-assets/web-upload/be68c992-51d8-45a5-a1d7-696bf956a470.png)

Запомните несколько особенностей CPNowPlayingTemplate:

* CPNowPlayingTemplate использует паттерн «одиночка» (singleton): нужно использовать и настраивать этот единственный экземпляр.
* Если вы включили необязательные кнопки «Очередь воспроизведения» и «Исполнитель», добавьте хотя бы одного наблюдателя для действий этих кнопок.
* Настраивать CPNowPlayingTemplate нужно сразу при запуске CarPlay app (то есть при подключении CarPlay scene). Система может показать CPNowPlayingTemplate от вашего имени, и даже запустить ваше приложение только ради этого.
  * Например, когда пользователь нажимает кнопку Now Playing на главном экране CarPlay, система запускает ваше CarPlay app и сразу показывает CPNowPlayingTemplate.
  * Другой пример: когда ваше приложение становится Now Playing app, система также добавляет кнопку Now Playing bar в navigation bar или tab bar вашего приложения. Если пользователь нажмёт её, система покажет CPNowPlayingTemplate. Если в этот момент Now Playing app станет другое приложение, система автоматически уберёт эту кнопку.
* В template stack поверх CPNowPlayingTemplate можно сделать push только одного CPListTemplate. Если в вашем приложении включена кнопка «Очередь воспроизведения», хорошим решением будет сделать push нового CPListTemplate из CPNowPlayingTemplate, чтобы показать пользователю очередь воспроизведения.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167097469-assets/web-upload/d1020ddc-bfc8-440b-b4c8-598d289ad329.png)

В разделе «Первое знакомство с CarPlay framework» мы видели код жизненного цикла CarPlay app. Метод `didConnect` — лучшее место для настройки singleton CPNowPlayingTemplate.

В примере ниже мы настраиваем для CPNowPlayingTemplate кнопку скорости воспроизведения, при нажатии на которую вызывается completion block. После начальной настройки ваше приложение готово к тому, чтобы система показывала CPNowPlayingTemplate от вашего имени, и в дальнейшем вы можете обновлять CPNowPlayingTemplate в любой момент.

```swift
// Now Playing Template

import CarPlay

class CarPlaySceneDelegate: UIResponder, CPTemplateApplicationSceneDelegate {

    func templateApplicationScene(_ templateApplicationScene: CPTemplateApplicationScene,
            didConnect interfaceController: CPInterfaceController) {
      
        let nowPlayingTemplate = CPNowPlayingTemplate.shared

        let rateButton = CPNowPlayingPlaybackRateButton() { button in
                                                           
            // Change the playback rate!
                                                           
        }
        nowPlayingTemplate.updateNowPlayingButtons([rateButton])
    }
}
```

## Коммуникационные App

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096792-assets/web-upload/fb74438a-45e0-4b98-bc62-d93b48c8ef96.png)

В iOS 14 с помощью Carplay framework можно обновить ваш коммуникационный Carplay app, чтобы улучшить работу с коммуникациями на дисплее автомобиля.

До iOS 14 коммуникационные app в CarPlay существовали как приложения для сообщений и голосовых вызовов. Обычно их функциональность строилась на SiriKit и CallKit, и коммуникационные app по-прежнему должны использовать SiriKit и CallKit для голосовых и телефонных функций. Начиная с iOS 14 они могут также использовать CarPlay framework для отображения контактов, списка сообщений и статуса сообщений.

### CPListTemplate (улучшен)

#### CPMessageListItem (новый)

Одна из самых частых функций коммуникационных app — показ списка сообщений. В iOS 14 Apple представила новый подкласс CPListItem под названием CPMessageListItem для построения списков сообщений.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096859-assets/web-upload/eb096201-b357-44e5-a976-1a48f3979f9e.png)

При нажатии на CPMessageListItem completion block не вызывается: вместо этого автоматически активируется Siri в соответствии с параметрами, указанными в CPMessageListItem. Пользователь по-прежнему редактирует, читает и отвечает на сообщения через Siri.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096775-assets/web-upload/a33391a0-2be1-49d8-9e95-b745d7b6be16.png)

Слева в CPMessageListItem можно показывать индикатор непрочитанного, булавку (закреплено), звёздочку (избранное) или другие значки. Включайте их в зависимости от функций вашего приложения.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096546-assets/web-upload/b352b6df-da62-4150-9e16-f1b4e679f008.png)

Справа в CPMessageListItem можно показывать значок отключённого звука, текст или необязательный значок. Как и другие item, CPMessageListItem поддерживает динамическое обновление: достаточно изменить его свойства, чтобы легко поменять элементы сообщения.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096619-assets/web-upload/e73fb43f-fe76-45ea-bd15-17451fc13043.png)

### CPContactTemplate (новый)

Другая важная функция коммуникационных app — показ информации о контактах. CPContactTemplate — новый в iOS 14 template, созданный именно для этого.

С помощью CPContactTemplate можно показать аватар контакта, описательный текст, набор кнопок действий и кнопки navigation bar. Template поддерживает до трёх строк текста и до четырёх кнопок; текст и значки можно настраивать.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096546-assets/web-upload/05a61010-fce1-486f-8319-bba4f271e981.png)

## Зарядка электромобилей, парковка и быстрый заказ еды

В iOS 14 Apple добавила три новых типа приложений с поддержкой CarPlay:

* Зарядка электромобилей
* Парковка
* Быстрый заказ еды

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167097173-assets/web-upload/590ff989-2e56-4e92-a518-de2b315e3ed9.png)

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096282-assets/web-upload/d47e48fb-a13b-4a15-8420-f7f095bfcc85.png)

У этих типов приложений много общего: все они тесно связаны с опытом вождения и помогают водителю находить пункты назначения на карте. Кроме того, такие приложения могут узнавать доступность зарядных станций, бронировать зарядку или парковку и оформлять заказ. В iOS 14 появился набор новых template, с помощью которых ваше приложение может предоставлять эти функции в CarPlay.

Далее на примере CarPlay app для быстрого заказа еды разберём, как с помощью новых template разрабатывать приложения этих типов. У этого примера вверху экрана 4 tab: расположение ближайших точек, история покупок, избранное и информация о заказах. Этот CarPlay app упрощён: в нём нет подробного меню, управления аккаунтом и прочих настроек — только самые востребованные функции с простым управлением.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167094130-assets/web-upload/d59234ce-db96-41ee-a3dd-07f3b97b8d98.png)

### CPPointOfInterestTemplate (новый)

Ищет ли пользователь ближайший ресторан с drive-through, зарядную станцию для электромобиля или парковку — первая задача приложения — определить местоположение.

CPPointOfInterestTemplate — новый в iOS 14 template, сочетающий интерактивную карту из MapKit framework и информационную панель; он показывает ближайшие места и позволяет пользователю выбрать одно из них. Ваше приложение предоставляет список мест для показа на карте и необязательные значки. Template также поддерживает панорамирование и масштабирование и уведомляет вас об изменении области карты.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167097159-assets/web-upload/f55da0ca-16b3-45e8-82c3-58456a0605cd.png)

CPPointOfInterest (POI) — базовый элемент информационной панели CPPointOfInterestTemplate. Экземпляр CPPointOfInterestTemplate создаётся из массива `[CPPointOfInterest]`, и в этом массиве может быть не более 12 POI. Каждый POI содержит заголовок, необязательный подзаголовок и информационный текст, который показывается на панели с подробностями.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096950-assets/web-upload/59d02b6f-827d-482e-af24-68ed78c74060.png)

**Примечание** | Помните, что в CarPlay водителю нужно показывать только самую релевантную информацию. В iPhone app можно показывать все возможные места, а в CarPlay стоит ограничиться: показывайте только наиболее релевантные или ближайшие места. Если несколько мест на карте расположены очень близко, CarPlay автоматически сгруппирует их.

CPPointOfInterestTemplate, как и другие template, поддерживает динамическое обновление свойств.

В нашем примере, чтобы показать и отличить выбранное место, мы обновляем свойство `pinImage` выбранного места, используя другой цвет (здесь жёлтый, а у остальных невыбранных — красный).

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096916-assets/web-upload/e853cf5c-0d4a-4cb6-bcc2-0b0af433aca1.png)

Когда в списке выбрано место, можно показать панель с подробной информацией о нём и не более чем двумя кнопками. Кнопки могут служить для разных задач. Например:

* переключиться на навигационное CarPlay app и проложить маршрут к выбранному месту;
* сделать push другого template, чтобы показать подробную информацию о выбранном месте.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096937-assets/web-upload/c51fdc04-0b19-4301-9961-714c2db73722.png)

#### Обновление списка мест

Пользователь может панорамировать и масштабировать карту в CPPointOfInterestTemplate; оба действия меняют видимую область карты, и ваше приложение должно обновить template новым списком мест, соответствующим новой видимой области.

Посмотрим на пример кода:

* Сначала задайте template делегат, соответствующий протоколу CPPointOfInterestTemplateDelegate; он будет получать уведомление при каждом изменении области карты.
* Затем реализуйте метод `pointOfInterestTemplate(:didChangeMapRegion)` для отслеживания изменений области карты. В реализации мы по новой области карты `region` получаем новый список мест `locations` и передаём его в template. После этого CarPlay обновит список мест и карту по новым `locations`.

```swift
// CPPointOfInterestTemplateDelegate

func pointOfInterestTemplate(_ template: CPPointOfInterestTemplate, 
                             didChangeMapRegion region: MKCoordinateRegion) {

    self.locationManager.locations(for: region) { locations in
        template.setPointsOfInterest(locations, selectedIndex: 0)
    }
}
```

В следующем примере показана реализация метода получения списка мест.

* Сначала создаём массив `[CPPointOfInterest]`.
* Затем определяем ближайшие места, обращаясь к вашей собственной базе мест или используя MapKit.
* Далее обходим полученный список данных о местах, создаём из них экземпляры CPPointOfInterest и добавляем в массив. Экземпляры CPPointOfInterest можно кешировать для повторного использования.
* Наконец, вызываем handler и передаём в него массив `[CPPointOfInterest]`.

```swift
// CPPointOfInterest creation

func locations(for region: MKCoordinateRegion, 
               handler: ([CPPointOfInterest]) -> Void) {
    var tempateLocations: [CPPointOfInterest] = []
        
    for clientModel in self.executeQuery(for: region) {
        let templateModel : CPPointOfInterest = self.locations[clientModel.mapItem] ??
                CPPointOfInterest(location: clientModel.mapItem,
                                  title: clientModel.title,
                                  subtitle: clientModel.subtitle,
                                  informativeText: clientModel.informativeText,
                                  image: clientModel.mapImage)
            
            
        tempateLocations.append(templateModel)
    }
    handler(templateLocations)
}
```

Обновлять список мест так же просто: реагируя на изменения области карты и в реальном времени показывая актуальный список POI, вы помогаете пользователю находить ближайшие пункты назначения.

#### Поддержка выбора места

Когда пользователь нажимает на место в списке, показывается панель с подробной информацией о нём. Задав необязательное свойство кнопки у CPPointOfInterest, можно добавить на эту панель кнопку Select, с помощью которой пользователь выберет это место.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096950-assets/web-upload/cfea2db9-87a2-441e-a49a-7b718c03e822.png)

Пример кода:

* Сначала создаём экземпляр CPPointOfInterestButton и передаём ему completion block, который вызывается при нажатии на кнопку. Здесь мы устанавливаем у экземпляра CPPointOfInterest выбранного места image в «выбранный» вариант, а selectedIndex у CPPointOfInterestTemplate — в index выбранного места. CarPlay динамически обновит дисплей автомобиля.
* Затем присваиваем кнопку свойству primaryButton экземпляра CPPointOfInterest, после чего она появится на панели с подробностями.

```swift
// Point of Interest Template location selection

let primaryButton = CPPointOfInterestButton(title: "Select") { button, [weak self] in
            let selectedIndex = ...
            
            if selectedIndex != NSNotFound {
                // Remove any existing selected state on previous location
                self?.selectedLocation.image = defaultMapImage
                // Change annotation for selected POI
                self?.selectedLocation = templateModel
                templateModel.image = selectedMapImage
                // Update the template with new values
                self?.pointOfInterestTemplate.selectedIndex = selectedIndex
            }
        }

let templateModel: CPPointOfInterest = ...

templateModel.primaryButton = primaryButton
```

### CPInformationTemplate (новый)

Мы только что реализовали показ и обновление списка мест для видимой области карты, а также выбор места. Теперь, когда пользователь выбрал место или выполнил задачу, может понадобиться показать сводку или иным образом завершить взаимодействие.

CPInformationTemplate — тоже новый template в iOS 14. CPInformationTemplate собирает много содержимого на одной странице: он состоит из списка меток в одну или две колонки и набора кнопок внизу, служит для показа текста и получения ответа пользователя.

Когда нужно полноэкранное взаимодействие, этот template подходит для многих случаев:

* В нашем примере быстрого заказа еды он может служить сводкой заказа, как на рисунке ниже, или страницей подтверждения заказа.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167096385-assets/web-upload/26c2d96b-6337-4315-9440-8bc3019fbc5c.png)

* В приложении для зарядки электромобилей он может показывать важную информацию о зарядной станции.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167095996-assets/web-upload/fce8f432-c5bd-4b45-8151-ca343d71863a.png)

## Что учесть при создании CarPlay App

* Для вашего приложения нужно запросить CarPlay entitlement. CarPlay app должно относиться к одной категории, и вам нужно выбрать категорию, которую поддерживает ваше приложение. Выбранный entitlement определяет, какие CarPlay template доступны вашему приложению.
* Если вы создаёте аудио CarPlay app и хотите поддерживать iOS 13 и более ранние версии, MPPlayableContent API и template будут сосуществовать в вашем приложении.

![](https://cdn.nlark.com/yuque/0/2022/png/12376889/1658167089726-assets/web-upload/a2241a4a-3bbf-464e-ac82-35e77d01f2aa.png)

## Итоги

В статье кратко рассмотрены обновления CarPlay framework в iOS 14. Если категория вашего приложения поддерживается CarPlay, начните с import CarPlay и сделайте так, чтобы ваше приложение заблистало в CarPlay!

Больше о CarPlay можно узнать на сайте [CarPlay for developers](https://developer.apple.com/carplay/), где обновлено [«CarPlay App Programming Guide»](https://developer.apple.com/carplay/documentation/CarPlay-App-Programming-Guide.pdf). Там же можно запросить CarPlay entitlement для вашего приложения и посмотреть, какие именно CarPlay template доступны для каждого entitlement.

Если вам нужно развернуть аудио CarPlay app на iOS 13 или более ранних версиях, рекомендуем ещё раз прочитать [документацию по MPPlayableContent API](https://developer.apple.com/documentation/mediaplayer/mpplayablecontentmanager/); если вы создаёте коммуникационный CarPlay app, оно обязано использовать SiriKit, а большое количество документации и ресурсов есть на сайте для разработчиков CarPlay; если вы создаёте навигационный CarPlay app на основе template, посмотрите [WWDC18 - CarPlay Audio and Navigation Apps](https://developer.apple.com/wwdc18/213).

## Ссылки

* [WWDC20｜10635 - Accelerate your app with CarPlay](https://developer.apple.com/wwdc20/10635)
* [WWDC18｜213 - CarPlay Audio and Navigation Apps](https://developer.apple.com/wwdc18/213)
* [Apple Developer｜CarPlay for developers](https://developer.apple.com/carplay/)
* [Apple Developer｜CarPlay App Programming Guide](https://developer.apple.com/carplay/documentation/CarPlay-App-Programming-Guide.pdf)
* [Apple Developer｜MPPlayableContent API](https://developer.apple.com/documentation/mediaplayer/mpplayablecontentmanager/)

## Рекомендуем почитать

* [师大小海腾 на GitHub｜iOS-CarPlay](https://github.com/teney97/iOS-CarPlay)
* [iOS CarPlay｜Делюсь процессом и деталями разработки аудио CarPlay App](https://juejin.cn/post/7035671279218720805)
* [iOS CarPlay｜WWDC22 - Расширяем возможности вашего App с помощью CarPlay](https://juejin.cn/post/7114239495360233479)
* [iOS CarPlay｜WWDC20 - Ускоряем ваш App с помощью CarPlay]()



