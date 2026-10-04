Эта статья — конспект сессии [WWDC22｜10016 - Get more mileage out of your app with CarPlay](https://developer.apple.com/videos/play/wwdc2022/10016/).

## Обзор 

Спустя 2 года CarPlay наконец-то обновился. Основное содержание сессии:

* Знакомство с двумя новыми типами приложений с поддержкой CarPlay в iOS16: Fueling App и Driving Task App
* Как Navigation App рисует карту в реальном времени на цифровой приборной панели поддерживаемых автомобилей
* 🌟 CarPlay Simulator: новый инструмент для разработки и тестирования CarPlay App. Он позволяет подключить iPhone Device и разрабатывать и тестировать CarPlay App, не вставая из-за рабочего стола, имитируя реальное окружение, — без необходимости постоянно бегать в машину или покупать aftermarket-головное устройство для тестирования
* Краткая демонстрация тестирования CarPlay App с помощью CarPlay Simulator

![20220628141134.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/132d77361244433190a20070c9201ca0~tplv-k3u1fbpfcp-watermark.image?)

**Примечание**｜В этой статье, за исключением `CarPlay App`, слово `App` само по себе означает `iPhone App`.

## Два новых типа приложений с поддержкой CarPlay: Fueling App и Driving Task App

**Отступление**｜CarPlay создан для водителей, поэтому при разработке CarPlay App основным пользователем нужно считать водителя. Включайте в CarPlay только функции, связанные с вождением, и отбрасывайте всё, что не следует делать за рулём, а также сложные и редко используемые функции. Такие вещи, как одноразовая настройка, вход в аккаунт или чтение условий, лучше выполнять до или после поездки, поэтому в вашем CarPlay App им не место. Обратите внимание: чтобы ваше приложение отображалось в CarPlay, нужно получить разрешение (entitlement). Запросить его в зависимости от типа вашего приложения можно на сайте Apple CarPlay Developer.

До этого существовало 6 типов приложений с поддержкой CarPlay:

![20220628141415.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2bf5619542804a899d3e8e8513b6248f~tplv-k3u1fbpfcp-watermark.image?)

В этом году Apple добавила 2 новых типа приложений с поддержкой CarPlay: Fueling App и Driving Task App.

![20220628141428.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/21a0ed99eef34c058f869ac71f231648~tplv-k3u1fbpfcp-watermark.image?)

### Отступление｜Templates

Templates (шаблоны) — это способ отображения UI в CarPlay App. Ваш CarPlay App поставляет данные, а Templates отрисовывают UI на экране автомобиля от вашего имени. Подключить Templates к CarPlay App легко, и у этого есть несколько преимуществ:

* UI на основе Templates не поддаётся такой глубокой кастомизации, как UIKit UI, так что можно не переживать о вычурном дизайне интерфейса. Сложность Templates UI невысока: UI в CarPlay App выводится через простой Templates API
* Не нужно беспокоиться о размере шрифтов: Templates сами адаптируются к разным экранам автомобилей
* Благодаря Templates UI вашего CarPlay App выдержан в том же стиле, что и UI других CarPlay App, поэтому пользователям проще освоиться в вашем приложении
* Templates гарантируют, что UI вашего CarPlay App корректно отображается и работает в любом автомобиле с поддержкой CarPlay, независимо от типа и размера дисплея

Короче говоря, Templates берут на себя большую часть работы.

При создании приложения можно выбирать из множества Templates. Например, Grid Template, отображающий массив кнопок (на рисунке ниже — 4-й в 1-м ряду), List Template, отображающий таблицу (на рисунке ниже — 1-й во 2-м ряду), и другие.

![2022062815492444.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5758885944044755a47e59c0123bfa2b~tplv-k3u1fbpfcp-watermark.image?)

И как разработчик, и как пользователь iOS вы должны быть знакомы с этими Templates. Главное — с ними знакомы ваши пользователи CarPlay, ведь они встречаются во всей системе CarPlay в автомобиле.

Одни Templates общие для всех типов приложений, а другие доступны только приложениям определённых типов. Подробности смотрите в диаграмме ниже.

![20220628142803.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5f2df0e10ea340439c181feaa41fb21b~tplv-k3u1fbpfcp-watermark.image?)

### Fueling App

**EV Charging App**｜EV Charging App — тип приложений с поддержкой CarPlay, добавленный Apple в iOS14. Такие приложения помогают найти расположение зарядных станций для электромобилей, а также подключить автомобиль к нужной станции и запустить зарядку.

Подобная функциональность подходит не только электромобилям, но и бензиновым. Поэтому в этом году Apple добавила поддержку CarPlay и для Fueling App, предназначенных для помощи водителю в заправке автомобиля. Обратите внимание: помимо поиска АЗС, Fueling CarPlay App должен уметь, например, запускать топливораздаточную колонку.

### Driving Task App

Driving Task App — новый тип CarPlay App, который охватывает самые разные приложения для выполнения простых задач. Но по-прежнему важно помнить: основная цель таких CarPlay App — выполнение задач, которые действительно нужны водителю во время вождения и реально помогают в езде, а не просто задач, которые вам (разработчику) нужно сделать, пока вы за рулём.

Примеры приложений этого типа:

* Приложения, помогающие управлять аксессуарами автомобиля
* Приложения, предоставляющие информацию о вождении или состоянии дорог
* Приложения, помогающие выполнять задачи в начале или конце поездки

Давайте рассмотрим несколько более конкретных примеров.

#### Road Status App

Road Status App может уведомлять пользователя о важной информации о дорогах. Этот CarPlay App построен на CPPointOfInterestTemplate. Учтите, что пользователь такого CarPlay App ведёт автомобиль, поэтому он должен показывать очень короткий список важной дорожной информации рядом с текущим местоположением пользователя. Это не подходит для приложений, помогающих пользователю полностью спланировать маршрут перед поездкой.

![20220628143633.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/957fca709c8e4ffdb2c750ae7dd8a7ce~tplv-k3u1fbpfcp-watermark.image?)

В этом CarPlay App вот что видит пользователь при выборе местоположения; подсказки лучше делать краткими и ёмкими.

![20220628143523.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a9654090dea64e8293323793d2f9b497~tplv-k3u1fbpfcp-watermark.image?)

####  Trailer Controller App

Trailer Controller App можно использовать для управления аксессуарами автомобиля. Этот CarPlay App использует CPInformationTemplate, чтобы показать базовую информацию о подключённом аксессуаре, и предоставляет две кнопки для действий пользователя. Вот насколько просты UI и функциональность этого CarPlay App. Конечно, у самого iPhone App есть много других функций, но те из них, что не связаны с вождением, в CarPlay App не выносятся. Пользователь сможет воспользоваться ими в iPhone App, когда выйдет из машины или остановится.

![20220628143811.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a60ce7bbb308463ba6d167ba10286caa~tplv-k3u1fbpfcp-watermark.image?)

#### Mileage Logger, Express Lane

Наконец, рассмотрим несколько примеров с использованием CPGridTemplate.

Mileage Logger — очень простой CarPlay App всего с двумя кнопками. Он позволяет водителю записывать пробег как личный или как служебный. Это приложение отлично подходит под новый тип Driving Task App, потому что позволяет пользователю очень удобно выполнять простую задачу прямо во время вождения.

![20220628143915.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c369fc2cc2294620acfb0aad483a90ea~tplv-k3u1fbpfcp-watermark.image?)

UI Express Lane App похож на Mileage Logger App. Это CarPlay App для транспондера платной полосы с быстрым проездом, который с помощью CPGridTemplate позволяет пользователю выбрать, сколько пассажиров находится в автомобиле. Функциональность этого CarPlay App тоже очень проста и удобна, так что это ещё один идеальный Driving Task App.

![20220628144018.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/63ea594ddf684870b70ee866cc2a3cbc~tplv-k3u1fbpfcp-watermark.image?)

#### Итоги

Подытожим: при проектировании Driving Task App обратите внимание на следующее:

* Обязательно подумайте о CarPlay App с одним экраном, предоставляющем минимум функций, необходимых во время вождения
* Предоставляйте только те функции, которые можно выполнить за несколько секунд
* Избегайте сложных или редко используемых функций, например первоначальной настройки или детальной конфигурации
* Не предоставляйте функции, которые не нужны во время вождения, даже если они связаны с автомобилем

## Как Navigation App рисует карту в реальном времени на цифровой приборной панели поддерживаемых автомобилей

![20220628135234.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/16f6e62ad22a4da9b7ee019932ef5796~tplv-k3u1fbpfcp-watermark.image?)

**CarPlay Dashboard**｜В iOS 13 Apple добавила API, позволяющий Navigation App рисовать карту в CarPlay Dashboard. Вы редактируете Info.plist, чтобы объявить поддержку Dashboard, добавляете необходимую Scene session role и реализуете нужный delegate. Когда Navigation CarPlay App появляется в Dashboard или исчезает из него, система уведомляет ваш delegate и передаёт вам UIWindow, в которой вы можете рисовать содержимое карты.

![20220628153406.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29b656a687774d40b85af5369a853319~tplv-k3u1fbpfcp-watermark.image?)

Если вы уже добавили поддержку Dashboard, то добавить поддержку Instrument Cluster будет проще простого, поскольку реализуется она аналогично Dashboard.

Сначала в Application scene manifest файла Info.plist объявите поддержку Instrument Cluster (`CPSupportsInstrumentClusterNavigationScene = true`) и добавьте необходимую Scene session role.

```xml
<key>UIApplicationSceneManifest</key>
<dict>
    <!-- Indicate support for CarPlay dashboard -->
    <key>CPSupportsDashboardNavigationScene</key>
    <true/>
    <!-- Indicate support for instrument cluster displays -->
    <key>CPSupportsInstrumentClusterNavigationScene</key>
    <true/>
    <!-- Indicate support for multiple scenes -->
    <key>UIApplicationSupportsMultipleScenes</key>
    <true/>
    <key>UISceneConfigurations</key>
    <dict>
        <!-- For device scenes -->
        <key>UIWindowSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>UIWindowScene</string>
                <key>UISceneConfigurationName</key>
                <string>Phone</string>
                <key>UISceneDelegateClassName</key>
                <string>MyAppWindowSceneDelegate</string>
            </dict>
        </array>
        <!-- For the main CarPlay scene -->
        <key>CPTemplateApplicationSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>CPTemplateApplicationScene</string>
                <key>UISceneConfigurationName</key>
                <string>CarPlay</string>
                <key>UISceneDelegateClassName</key>
                <string>MyAppCarPlaySceneDelegate</string>
            </dict>
        </array>
        <!-- For the CarPlay Dashboard scene -->
        <key>CPTemplateApplicationDashboardSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>CPTemplateApplicationDashboardScene</string>
                <key>UISceneConfigurationName</key>
                <string>CarPlay-Dashboard</string>
                <key>UISceneDelegateClassName</key>
                <string>MyAppCarPlayDashboardSceneDelegate</string>
            </dict>
        </array>
        <!-- For the CarPlay instrument cluster scene -->
        <key>CPTemplateApplicationInstrumentClusterSceneSessionRoleApplication</key>
        <array>
            <dict>
                <key>UISceneClassName</key>
                <string>CPTemplateApplicationInstrumentClusterScene</string>
                <key>UISceneConfigurationName</key>
                <string>CarPlay-Instrument-Cluster</string>
                <key>UISceneDelegateClassName</key>
                <string>MyAppCarPlayInstrumentClusterSceneDelegate</string>
            </dict>
        </array>
    </dict>
</dict>
```

Затем реализуйте scene delegate для Instrument Cluster вашего приложения. API предоставит вам UIWindow для рисования содержимого карты и уведомит вас о запуске и закрытии Instrument Cluster. Вы можете рисовать карту в Instrument Cluster в реальном времени.

```swift
extension TemplateApplicationSceneDelegate: CPTemplateApplicationInstrumentClusterSceneDelegate {
    
    func templateApplicationInstrumentClusterScene(
        _ templateApplicationInstrumentClusterScene: CPTemplateApplicationInstrumentClusterScene,
        didConnect instrumentClusterController: CPInstrumentClusterController) {
        // Connected to Instrument Cluster
        TemplateManager.shared.clusterController(instrumentClusterController, didConnectWith: templateApplicationInstrumentClusterScene.contentStyle)
    }
    
…

    func instrumentClusterControllerDidConnect(_ instrumentClusterWindow: UIWindow) {
        // Window in which to draw instrument cluster contents 
       self.instrumentClusterWindow = instrumentClusterWindow
    }
}
```

Хотя реализация поддержки Instrument Cluster и Dashboard очень похожа, у Instrument Cluster есть свои особенности.

* Во-первых, Instrument Cluster позволяет пользователю увеличивать и уменьшать карту, поэтому эту функцию нужно реализовать в вашем CarPlay App с помощью CPInstrumentClusterControllerDelegate.
* Кроме того, если ваш CarPlay App поддерживает отображение компаса или ограничения скорости, система в нужный момент уведомит ваш delegate, чтобы вы их отрисовали.
* Наконец, учтите, что ваше представление для Instrument Cluster может быть частично закрыто другими элементами приборной панели автомобиля. Разумеется, в iOS для этого уже есть первоклассный механизм — safe area. Вы можете переопределить `viewSafeAreaInsetsDidChange` в view controller, чтобы отслеживать изменения safe area, и использовать `safeAreaLayoutGuide` в Cluster View, чтобы ключевое содержимое оставалось в видимой области и не перекрывалось.

## CarPlay Simulator

![carplay_simulator.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c42456f739ec426eab9c0d2581e23b55~tplv-k3u1fbpfcp-watermark.image?)

Это главная новинка в обновлении CarPlay этого года, и она доступна CarPlay App любого типа.

Раньше протестировать CarPlay App можно было двумя способами:

1. В Xcode iPhone Simulator есть встроенное окно CarPlay, с помощью которого удобно быстро протестировать ваш CarPlay App
2. Подключить iPhone Device к автомобилю с поддержкой CarPlay или к aftermarket-головному устройству — раньше это был единственный способ протестировать CarPlay App на iPhone Device

В этом году Apple представила CarPlay Simulator — Mac App, который имитирует полную среду CarPlay в автомобиле. Его можно скачать на сайте Apple для разработчиков: [Дополнительные инструменты для Xcode - Additional Tools for Xcode 14 beta](https://developer.apple.com/download/all/).

![20220628144938.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4096db6cfcd64c46ba1a4fcaab0fafdf~tplv-k3u1fbpfcp-watermark.image?)

Откройте его — CarPlay Simulator находится в папке Hardware.

![20210628093610.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/31bd5dc71ef14aaf87e977ff67992de1~tplv-k3u1fbpfcp-watermark.image?)

![20210628093642.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/abd512da8b2d4a309ead345b7901bc96~tplv-k3u1fbpfcp-watermark.image?)

Запустите это приложение и подключите iPhone Device к Mac кабелем — CarPlay запустится.

**Примечание**｜Чтобы пользоваться CarPlay Simulator, обновлять macOS и Xcode не нужно.

![20220628145204.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb219deb0c9641089c9bf3a0a3646147~tplv-k3u1fbpfcp-watermark.image?)

У CarPlay Simulator есть несколько преимуществ:

* Когда вы используете CarPlay Simulator, ваш iPhone device уже подключён к Mac, поэтому одновременно можно пользоваться другими инструментами разработки на Mac, например отлаживать в Xcode или тестировать производительность в Instruments.
* Окно CarPlay, встроенное в iPhone Simulator, не позволяет протестировать некоторые сценарии. Например, нужно проверить, правильно ли голосовые подсказки вашего Navigation App смешиваются с нативными аудиоисточниками автомобиля (например, FM radio). Раньше для этого приходилось часто бегать в машину или покупать aftermarket-головное устройство. Теперь можно использовать CarPlay Simulator: CarPlay фактически работает на вашем iPhone Device — так же, как в автомобиле. Это значит, что тестировать теперь очень удобно прямо на рабочем месте.
* Кроме того, CarPlay Simulator позволяет тестировать автомобили с разными конфигурациями, например дисплеи разных размеров.

Теперь расскажем, как пользоваться CarPlay Simulator. Окно его интерфейса состоит из 3 частей:

* В центре окна — представление CarPlay, имитирующее дисплей автомобиля
* В нижней части окна — кнопки, имитирующие различные аппаратные клавиши и ручки автомобиля
* В верхней части окна — несколько кнопок
  * Configure: кнопка настройки, открывающая дополнительное окно с расширенными функциями (подробнее ниже)
  * Limit UI Off/On: имитация ограничения некоторого содержимого в CarPlay, например сокращения списков в аудио CarPlay App
  * Light/Dark UI: имитация переключения светлого и тёмного оформления UI в CarPlay автомобиля
  * Light/Dark Map: имитация переключения светлого и тёмного оформления карты в CarPlay автомобиля
  * Connected/Disconnected: имитация отключения и повторного подключения iPhone Device к CarPlay автомобиля без втыкания и вытаскивания кабеля. Поскольку при использовании этой кнопки iPhone Device остаётся подключённым к Mac, с её помощью можно отлаживать в Xcode отключение и повторное подключение CarPlay Scene.

![WX20220628-230108.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/adc44f7d154f44fcba1cebcefa5d96a4~tplv-k3u1fbpfcp-watermark.image?)

### Configure

Нажмите кнопку Configure в главном окне, чтобы открыть дополнительное окно с расширенными функциями.

![20220628154833.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e71bfb0a4f51453b875e12f802d4c4ee~tplv-k3u1fbpfcp-watermark.image?)

На вкладке General можно задать размер дисплея CarPlay. Если UI вашего CarPlay App состоит только из Templates, они сами обеспечат корректное отображение и работу в любом автомобиле с поддержкой CarPlay, но всё равно можно попробовать разные размеры, чтобы посмотреть, как UI выглядит в разных автомобилях. Однако если ваше приложение — Navigation App, то крайне важно проверить разные размеры и соотношения сторон, чтобы убедиться, что код отрисовки карты работает правильно.

![20220628154842.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a08626b419c45da92bde0e40efb8594~tplv-k3u1fbpfcp-watermark.image?)

Рекомендуемые размеры дисплея для тестирования Navigation App:

* 800 x 480 (default)
* 1280 x 720
* 900 x 1200

На вкладке Cluster Display можно отметить Instrument Cluster Display enabled и нажать кнопку Restart Session — откроется новое окно, имитирующее дисплей приборной панели автомобиля.

![20220628154918.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/379fba09b50a4784afa1bdd8574b3bc6~tplv-k3u1fbpfcp-watermark.image?)

![20220628154924.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/533151b1cc3b4ed191b45f6dc9f4b66a~tplv-k3u1fbpfcp-watermark.image?)

Эта функция относится к Navigation App: дисплей приборной панели служит для показа водителю карты или карточки поворота в поле зрения на приборной панели автомобиля.

Кнопка Restart Session нужна, чтобы изменения, внесённые в Configure, вступили в силу немедленно, без перезапуска CarPlay Simulator.

Это всё о CarPlay Simulator; остальные возможности вы можете изучить самостоятельно.

## Тестирование CarPlay App с помощью CarPlay Simulator

Запустите CarPlay Simulator.

![20220628112419.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2b3e24f13601442f966871ba85fc6cda~tplv-k3u1fbpfcp-watermark.image?)

Подключите iPhone Device к Mac — CarPlay запустится.

![20220628112601.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/712038a060bf435b81f1b3b7182e9e5d~tplv-k3u1fbpfcp-watermark.image?)

Если ваш CarPlay App предоставляет два набора icon для светлого и тёмного оформления, то для тестирования можно переключать оформление кнопкой Light/Dark UI. Но перед этим нужно зайти в настройки CarPlay и выставить режим оформления `Автоматически`, а не `Всегда тёмный режим`.

![20220628112724.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/528c38871cf6483a917d335d9a04de0d~tplv-k3u1fbpfcp-watermark.image?)

![20220628112639.jpg](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8716a200bc8545d68c9579b541c9431e~tplv-k3u1fbpfcp-watermark.image?)

Размер дисплея CarPlay можно задать на вкладке Configure - General, а затем нажать кнопку Restart Session — изменения применятся немедленно, без перезапуска Carplay Simulator, что очень удобно. С помощью этой функции можно проверить, хорошо ли ваш CarPlay App, и особенно Navigation App, адаптирован к дисплеям автомобилей разных размеров.

![20220628115604.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/14457fe2ab144ca8ba68b1d0c938659d~tplv-k3u1fbpfcp-watermark.image?)

![20220628115614.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/db8674e8796e4c3ab9c02d016c3f363f~tplv-k3u1fbpfcp-watermark.image?)

На вкладке Cluster Display можно включить Instrument Cluster Display, чтобы протестировать поддержку Instrument Cluster автомобиля в вашем Navigation App.

![20220628115640.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0aa6cb08e50d40928563678ba91056e3~tplv-k3u1fbpfcp-watermark.image?)

Если вы всесторонне протестировали свой CarPlay App в CarPlay Simulator и всё работает хорошо, можно быть уверенным, что он будет хорошо работать и в настоящем автомобиле. Тем не менее всё же рекомендуется попробовать его в машине.

![20220628135107.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8823c6c6539c45bf9b23a1ecc15885cd~tplv-k3u1fbpfcp-watermark.image?)

Templates гарантируют, что UI вашего CarPlay App корректно отображается и работает в любом автомобиле с поддержкой CarPlay.

![20220628135123.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d99c61f763a48e192a9a3ecff3abd12~tplv-k3u1fbpfcp-watermark.image?)

Некоторым пользователям может нравиться управлять CarPlay App с помощью ручек автомобиля. Если вы протестировали приложение с помощью имитации ручек в CarPlay Simulator, то и в автомобиле оно поведёт себя хорошо. Templates сами обрабатывают события ручек.

![20220628135132.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a4ce841a0f6148c6b27a1ad5f7952575~tplv-k3u1fbpfcp-watermark.image?)

![20220628135152.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4d4c34aaab334c089f950be6f332a4a3~tplv-k3u1fbpfcp-watermark.image?)


Также можно проверить, как ваш Navigation App поддерживает цифровую приборную панель автомобиля. Для водителя очень здорово видеть карту в реальном времени прямо в поле зрения, и водителям, пользующимся вашим Navigation App, это обязательно понравится.

![20220628135234.jpg](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e8246be568e64617b308e7fc524b5bed~tplv-k3u1fbpfcp-watermark.image?)

![20220628135241.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4a68ab8d6e5d4585b6d1820557a57c8f~tplv-k3u1fbpfcp-watermark.image?)

## Источники 

* [WWDC22｜10016 - Get more mileage out of your app with CarPlay](https://developer.apple.com/videos/play/wwdc2022/10016/) 
* [Apple｜CarPlay App Programming Guide](https://developer.apple.com/carplay/documentation/CarPlay-App-Programming-Guide.pdf)
* [iOS CarPlay｜делимся процессом и деталями разработки аудио-приложения для CarPlay](https://juejin.cn/post/7035671279218720805)
